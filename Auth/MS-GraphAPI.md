For all of the graph API stuff I needed the additional two roles: `Application Administer` and `Cloud Application Administer`.

All of these calls can also be found in the `OcelotAPI` Postman Group then `MS GRAPH API` Collection.

[MS Graph Explorer](https://developer.microsoft.com/en-us/graph/graph-explorer)

### POST

- Token Request

  ```HTTP
  POST https://login.microsoftonline.com/transportationwaas.onmicrosoft.com/oauth2/v2.0/token

      {
         "client_id": <client_id>
         "scope": "https://graph.microsoft.com/.default"
         "client_secret": <client_secret>
         "grant_type": "client_credentials"
      }
  ```

- revokeSignInSessions
  ```HTTP
    POST /users/{id | userPrincipalName}/revokeSignInSessions
  ```

### PATCH

- Edit User Attributes

  ```HTTP
  PATCH
  https://graph.microsoft.com/v1.0/transportationwaas.onmicrosoft.com/users/<userid>

      {
         "extension_<app-id-without-hyphens>_isAdmin": false,
         "extension_<app-id-without-hyphens>_isFinancial": false
      }
  ```

### GET

- Users
- Users/{ID}
- Groups/{ID}
- Groups

### DELETE

- Member From Group

## Links

[Register a Microsoft Graph application](https://learn.microsoft.com/en-us/azure/active-directory-b2c/microsoft-graph-get-started?tabs=app-reg-ga)

[Start Making Calls](https://learn.microsoft.com/en-us/graph/auth-v2-service?tabs=http#4-get-an-access-token)

[How to update azure ad b2c custom user attribute using graph api](https://stackoverflow.com/questions/63815710/how-to-update-azure-ad-b2c-custom-user-attribute-using-graph-api)

[PERMISSIONS: Manage user attributes and claims](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/custom-enterprise-app-permissions#manage-user-attributes-and-claims)

[create-user-custom-attribute (didn't work)](https://stackoverflow.com/questions/65919881/create-user-custom-attribute-for-whole-b2c-instance-via-graph-api-or-graph-sdk)
