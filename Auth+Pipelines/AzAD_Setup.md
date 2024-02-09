# Tutorial: Set up sign in for an ASP.NET application using Azure Active Directory B2C

Here are the steps I took to set up my Azure Active Directory B2C demo app.

- The first thing I tried was [quickstart-web-app-dotnet](https://learn.microsoft.com/en-us/azure/active-directory-b2c/quickstart-web-app-dotnet), but was unsucessfull in getting the app to run. The problem I was running into was with the `Owin` dependancy which the project depends on. The project expects version 1.0.1 but [Owin](https://www.nuget.org/packages/Owin/1.0.0?_src=template) only has version 1.0.0 available and changing the dependancy to this version also didn't work. Since this sample app uses a .NET web app I'm going to skip over the quick start and follow the other tutorials which explain how to add Azure AD to an existing .NET web api.

1. [Tutorial: Create an Azure Active Directory B2C tenant](https://learn.microsoft.com/en-us/azure/active-directory-b2c/tutorial-create-tenant)

   - If using the Ephemral subscription I have already registered Microsoft.AzureActiveDirectory in resource providers.
   - Note that you will also require `Tennant Creator` role, which you will NOT get unless ur sam... So ask sam...
     - Once the B2C tennant is created ask for `application developer` role under the newly created tennant.
     - If you were able to create the Tennant yourself than you will automatically be granted `global admin`.

2. CAUTION: Azure considers a "web application" to be a traditional web application that performs most of the application logic on the server. So for our purposes this will be our .NET Core Web API!!! Our Frontend app on the other hand is considered a SPA (single page applications) for app registration purposes!!! We will need to regeister **BOTH** our React App as well as our .NET Core web api.

   - Note that when the tutorial refers to `Azure AD B2C - App registrations` this means going to `App registrations` in the newly created Tennant _directory_. When you click on Azure AD B2C inside of this directory you will be met with `User Authorization: Access is denied` if you are limited to `application developer` role which is **OKAY**.
   - All steps in 2A and 2B were completed with only `application developer` role.
   - Make sure to copy the app secret values directly after generating them in these steps! "This secret value is never displayed again after you leave this page".

   2A. [Tutorial: Register a web application in Azure Active Directory B2C](https://learn.microsoft.com/en-us/azure/active-directory-b2c/tutorial-register-applications)

   2B. [Register a single-page application in Azure Active Directory B2C](https://learn.microsoft.com/en-us/azure/active-directory-b2c/tutorial-register-spa)

3. Step 3
   - fdgffghfdhfgyhr5hfg
