# Tutorial: Set up sign in for an ASP.NET application using Azure Active Directory B2C

Here are the steps I took to set up my Azure Active Directory B2C demo app.

- The first thing I tried was [quickstart-web-app-dotnet](https://learn.microsoft.com/en-us/azure/active-directory-b2c/quickstart-web-app-dotnet), but was unsucessfull in getting the app to run. The problem I was running into was with the `Owin` dependancy which the project depends on. The project expects version 1.0.1 but [Owin](https://www.nuget.org/packages/Owin/1.0.0?_src=template) only has version 1.0.0 available and changing the dependancy to this version also didn't work. Since this sample app uses a .NET web app I'm going to skip over the quick start and follow the other tutorials which explain how to add Azure AD to an existing .NET web api.

1. [Tutorial: Create an Azure Active Directory B2C tenant](https://learn.microsoft.com/en-us/azure/active-directory-b2c/tutorial-create-tenant)
   - If using the Ephemral subscription I have already registered Microsoft.AzureActiveDirectory in resource providers.
