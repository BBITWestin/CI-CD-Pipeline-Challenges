### 1/29/2024 - 3/5/2024

No logs

1. Started on team darwin, switched to team transpo day 2.
2. Reach weeks for project portal!
3. Pushed a bug on my first transpo task
4. CI/CD MASTER!
5. Learned bicep, powershell, azure command line, microsoft graph command line

### 3/6/2024 - Wednesday

CI/CD!!!! Today i created a powershell script to provision a service principle (app registration) with federated credentials scoped down to have a contributor role for a resource group to be used by github actions!!! Woo!

### 3/7/2024 - Thursday

More CI/CD... Today emily and i updated our app registration provision script to handle different portals and environments. Then we created another script to create a subscription level runner which we are looking to automatically create and delete before/after provision workflows... check it out here: https://github.com/bbi-dev-ops/project-portal-management/tree/main/github_runner_scripts_dont_touch

### 3/8/2024 - Friday

Code review this morning... all ps1 scripts working and github actions running fine. Spent the second half of the day working with Viraj or ML quote bot project and we've just discovered Azure's Machine Learning workspace... Reached the valley of dispair in the Dunning-Kruger curve.

### 3/11/2024 - Monday

Starting to work on powershell scripts for setting up B2C resources.

### 3/12/2024 - Tuesday

More b2c app reg ps1 stuff

### 3/13/2024 - Wednesday

More b2c app reg ps1 stuff
TODO: clean up the ps1 files and add documentation. Also setup auth in the swa and api... and fix the suffix problem when repovisioning the runners... then figure out how to edit the variables stored in github from the command line...

### 3/14/2024 - Thursday

Azure Auto ML jobs started and proof of concept completed. Have questions for matt by wednesday

### 3/15/2024 - Friday

Code review presentation on npm run in wrong directory, and azure b2c manager script.

### 3/18/2024 - Monday

Updating docs for azure b2c manager script... setting up MSAL in the frontend and attempted to protect some endpoints but ran into cors issues and useQurry init... Trying to come up with a PortalCaller.tsx that we can use everywhere.

### 3/19/2024 (Tuesday) - 3/29/2024 (Friday)

- Solved CORS issues for management portal including Veet build and deploy bugs.
- Updates to BBILogistics.com website
- Setup frontend and backend for different environments with same code. Vite env files and dotnet asp environment variables set on provision.
- Codeowners file for github pr reviews.
- Github branch protection rules.
- ENTIRE PROJECT STEPS TO RECREATE README!!! (several days)
- Machine Learning Project: Google drive api + service account python scripts + Outlook MS Graph python script for fetching data.

### 4/1/2024 (Tuesday)

- Continued working on the `steps the recreate` doc for all of project portal. Everything from Making the new repo to specifying which ports to use, to onion arch, to implementing azure b2c with msal... everything.

### 4/2/2024 (Wednesday)

- Started research on preview environments for pr's.w
- Worked with seph on steps to recreate docs for project portal dotnet api.

### 4/3/2024 (Thursday)

- Preview Enviroment research / planning.
