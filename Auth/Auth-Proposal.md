# Implementation Blueprint

So far we have looked at:

- [Challenges + Access Controll Strategies](https://github.com/BBITWestin/BBITWestin.github.io/blob/main/Auth/Auth-Challenges.md)
- [Azure AD B2C in the Field](https://github.com/BBITWestin/BBITWestin.github.io/blob/main/Auth/AzAD_Setup.md)
- [Our Use Case + Usefull Features in Azure B2C](https://github.com/BBITWestin/BBITWestin.github.io/blob/main/Auth/Authorization_Schema.md)

## Goals

- Offset as much authentication and authorization tasks and logic to an **identity as a service provider**
  - Authentication/Authorization should not rely on tables in our database
- Scalable authorization schema
  - Onbording new customers/clients should be automated with little to no action required on our end

## Proposal Summary

Using Azure AD B2C here is a summary of our proposal focusing on the implementation of Role-Based Access Control (RBAC) with hierarchical extensions for global and local roles, along with inheritance and overrides.

... how the hell do we do this... the stuff below is great but doesn't help us come up with some sort of schema to handle this network of users and permissions... still lost here.

This works great for specifying if a user should be able to access Financial info or some other category of actions. But what about assigning a single user access to financial info for just a single load.

## Implementation Blueprint

### User Flows

Start with the default user flows for common identity tasks such as sign-up, sign-in, profile editing, and password reset.

### Azure AD B2C Groups

Utilize groups in Azure AD to represent roles. Membership in a group can then dictate a user's role within your application.

### Microsoft Graph API

Build an interface for role management, allowing authorized users to assign or revoke roles (_B2C Groups_) from other users. by making calls to MS Graph API.

### Audit Logs

When implementing a system that allows users to modify roles or permissions for other users, audit logs and monitoring should be in place to track changes to groups and permissions.
