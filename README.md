# CI/CD Pipeline Challenges

Trevor Westin - 1/31/2024

## Challenges

- How can we synchronize deployment of updates to Frontend, Backend, and Database?

### Scenario 1

Both frontend and backend/db have separate repositories. After testing the latest versions of frontend and backend on staging environments, we want to deploy to production without any downtime.

When deploying `Frontend_v0` -> `Frontend_v1` it may happen that `Backend_v0` is not updated to `Backend_v1` or vice-verca. During this time `Frontend_v1` would have an incompatiible backend. It may also happen that `Frontend_v1` is successfully deployed but `Backend_v1` deployment fails and will result in a longer downtime.

Same problem can be said for backend and database as you'll see in Scenario 2.

### Scenario 2

It's time to make changes to the database. We locally `add-migration` and `update-database` with the connection string to it's respective database while github actions is deploying our updated backend. Not only are these locally run commands going to be out of sync with github actions, we also introduce the posibility for human error... maybe we run the commands on the wrong branch? or use the wrong connection string? ...or maybe Jacob forgets!!!!

### Goal

- Seamless transition between deployments with zero downtime, from a user perspective.
- Maintain compatibility between the frontend, backend and database during the deployment process.
- Implement a reliable rollback strategy.

### Scenario 2 Solution 1: MigrateAsync()

Use the following in Program.cs

```C#
await using (var scope = app.Services.CreateAsyncScope())
{
    var db = scope.ServiceProvider.GetRequiredService<TodoDb>();
    // await db.Database.EnsureCreatedAsync();

    await db.Database.MigrateAsync(); // This applies any pending migrations
}
```

When we make changes to our models in the backend we can simply add the migration and merge to DEV/PROD to trigger our github workflow for redeploying the backend per usual. Program.cs will run `MigrateAsync()` on start, applying any pending migrations.


For my demo I forked the [todo-csharp-sql](https://github.com/azure-samples/todo-csharp-sql/tree/main/) Azure sample app and deployed to [Ephemral](https://portal.azure.com/#@bbilogistics.com/resource/subscriptions/229ea71b-e62b-4512-a96a-50624b60eac2/overview) Subscription.

#### Steps to recreate Demo
1. Access to azure subscription where you are have atleast `Contributor` and `User access administrator` roles.
2. In Azure Active Directory you must also be able to register applications for the purpose of configuring github pipeline with azure.
3. Now that we have the permissions required, simply fork [this repo](https://github.com/azure-samples/todo-csharp-sql/tree/main/).
4. Skip step 5 if you are okay with dropping the database each time you want to edit the schema during testing.
5. Before following the steps listed in the read me it's important to remove `await db.Database.EnsureCreatedAsync();` in program.cs before deploying (`azd up` or pushing to main). This way you can use ef core migrations. See the [Remarks section here](https://learn.microsoft.com/en-us/dotnet/api/microsoft.entityframeworkcore.infrastructure.databasefacade.ensurecreatedasync?view=efcore-8.0) for more details on why the sample app choose to use `EnsureCreatedAsync()`.
    - You might need to comment out `await db.Database.EnsureCreatedAsync();` and then deploy before adding migrations and updating for the first time.
    - You will also need to remove the `Provision Infrastructure` *step* in ./github/workflows/***.yml before continueing.
6. Now follow the `azd` steps to provision, deploy, and config the pipeline.
7. When you're done testing for the day simply run `azd down`. *Azure sql databases are expensive to run*.

I still have some more testing to get done to figure out exactly how everything is playing together in this sample app. But [my repo](https://github.com/BBITWestin/auto-migrate-demo) does in fact update the database on deployments automatically.

### Scenario 2 Solution 2: Github Actions

When we push/merge to DEV or PROD we can use github workflows to pull a connections string from secrets and run the nessisary commands to update the database. Here is a simplified example focusing on database migration within a GitHub Actions workflow based on [this article](https://mabster.net/posts/ef-migration-github-actions/):

```yml
name: Database Migration and Backend Deployment

on:
  push:
    branches:
      - main

jobs:
  database_migration:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v2
      - name: Setup .NET Core
        uses: actions/setup-dotnet@v1
        with:
          dotnet-version: '3.1'
      - name: Install EF Tool
          run: |
              dotnet new tool-manifest
              dotnet tool install dotnet-ef
      - name: Generate EF migration script
          run: dotnet ef migrations bundle --startup-project MyApp --output ${{env.DOTNET_ROOT}}/myapp/efbundle.exe --configuration Bundle
      - name: Apply EF migration script
          run: ./efbundle.exe --connection "${{ secrets.MY_CONNECTION_STRING }}"
          
  backend_deployment:
    needs: database_migration
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Backend Code
        uses: actions/checkout@v2
      - name: Build and Deploy Backend
        run: |
          # Your build and deploy scripts here
```

More tested needed to create a working demo.

### Scenario 1 Solution 1: Backwards Compatibility

In a backwards compatibility approach, the key is to ensure that the new version of one component (e.g., `Backend_v1`) is compatible with both the old and new versions of another component (e.g., `Frontend_v0` and `Frontend_v1`).

The deployment sequence might look something like this:

1. Deploy `Backend_v0` -> `Backend_v1` which supports both `Frontend_v0` and `Frontend_v1`.
2. Wait for Backend_v1 to sucessfully deploy.
3. Deploy `Frontend_v0` -> `Frontend_v1`. Since `Backend_v1` is already compatible with the new frontend, this should result in a smooth transition with minimal risk of downtime or incompatibility issues.

This approach can be used for database updates as well.

#### Pros

- Works

#### Cons

- Increased Complexity: Requires careful design and planning to ensure compatibility.
- Additional Testing / Staging Environments: Requires aditional testing of `Backend_v1` with `Frontend_v0`. If DB changes are involved then this further complicates testing, requireing testing for the following environments:
  - `Backend_v1` with

### Scenario 1 Solution 2: Blue Green Deployment

Blue-green deployment is a strategy which utilizes two production environments (a `**`blue environment`**` and a `**`green environment`**`).

The two production environments are kept as identical as possible, and when new code is deployed, it is pushed to the environment that is currently inactive (`blue environment`). Once the new changes have been tested in production, a router can then switch to point to the `blue environment` where the new changes are live.

#### Pros

- Zero downtime.
- One of the main benefits of blue-green deployments is disaster recovery. Because there are two identical environments for production, if new changes are rolled out to `blue` and any issues are discovered, a router can just switch back to `green` which has the old version of the code with zero downtime.

- Blue-green deployment can be used for canary testing (see **Related Topics** below) by simply having the router direct a percentage of your traffic to new version of the code to see how it performs with live traffic, before rolling out the change to 100% of your users.

#### Cons

- Rollback Complexity: While it simplifies the initial switch, rolling back in case of an error can be complex if any unintended changes were made to the database.
- Overhead: It requires double the resources since two full environments (`blue` and `green`) must be maintained. Blue-green deployments also adds operational overhead in terms of configuration and management of two separate environments.

### Scenario 1 Solution 3: Feature Toggles

Use feature toggles to decouple deployment from release. This allows you to deploy new code that is not yet enabled. Once the new code is successfully deployed and the database migration is complete, you can enable the new functionality. This provides a way to quickly rollback changes if issues arise after deployment.

More research needed to find practical steps for implementation.

### Scenario 1 Other 'Solutions'

#### Downtime Planning

In some cases, especially for significant database changes, it may be more practical to schedule a brief downtime to update the database schema. This should be planned during off-peak hours.

---

### Related topics

#### Canary Release Strategy

- Canary Testing is a way to reduce risk and validate new software by releasing software to a small percentage of users. With canary testing, you can deliver new features to certain groups of users at a time.

- Since the new feature is only distributed to a small number of users, its impact is relatively small and changes can be reversed quickly should the new code prove to be buggy.

- It is a technique to reduce the risk of introducing a new software version in production by slowly rolling out the change to a small subset of users before rolling it out to the entire infrastructure and making it available to everybody.

- While canary releases are a good way to detect problems and regressions, A/B testing is a way to test a hypothesis using variant implementations.

#### A/B Release Strategy

- A/B testing is a way to compare two versions of something to determine which performs better.
- In an A/B test, some percentage of your users automatically receives _version A_ and other receives _version B_.
- It is a controlled experiment process. To run the experiment user groups are split into 2 groups. Group **A** often called the 'control group', continues to receive the existing product version, while Group **B** often called the 'treatment group', receives a different experience, based on a specific hypothesis about the metrics to be measured.
- At the end the results from 2 groups which is a variety of metrics is compared to determine which version performed better.

## Links
- [EF Core Migrations from Github Actions](https://mabster.net/posts/ef-migration-github-actions/)
- [Azure Sample App Repo](https://github.com/azure-samples/todo-csharp-sql/tree/main/) and [*my forked version of this repo*](https://github.com/BBITWestin/auto-migrate-demo)
- [Strategies for Database Development and Deployment (C#)](https://learn.microsoft.com/en-us/aspnet/web-forms/overview/older-versions-getting-started/deploying-web-site-projects/strategies-for-database-development-and-deployment-cs)
- [Blue-green Deployments, A/B Testing, and Canary Releases](https://blog.christianposta.com/deploy/blue-green-deployments-a-b-testing-and-canary-releases/)


## Coming Up:
- Once deployments are live, how to get user's frontend to udpate without asking them to refresh their web page.
- Pipelines for Bugs and Features?

## Later On:
- Blue Green strategy working demo for entire CI/CD pipeline.
    - 1 Dev, 2 Prod (green/blue).
- Rollback demo
