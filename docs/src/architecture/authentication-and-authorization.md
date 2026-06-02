# Authentication And Authorization

Authentication in FirstMile is split across multiple mechanisms and is assembled in infrastructure code rather than in a dedicated auth module.

## Implemented authentication mechanisms

### Azure AD OpenID Connect

`EngineConfigurator.ConfigureAzureAuthentication` configures:

- a cookie authentication scheme
- an OpenID Connect challenge scheme
- PKCE-enabled authorization code flow
- role mapping from Azure claims
- name mapping from `preferred_username`

Important implemented events:

- `OnSignedIn`: synchronizes CMS users and roles, then updates the contact last-login date in Salesforce-backed data if a contact is found
- `OnTokenValidated`: resolves the account or login page and redirects based on whether the user exists in contact data
- `OnRemoteFailure`: redirects back to the friendly login page instead of surfacing a raw auth failure

### SAML

`ConfigureSamlAuthentication` binds the `Saml2` configuration section, loads IdP metadata dynamically, and extracts the single sign-on destination and signing certificates. If the metadata cannot provide an IdP SSO descriptor, startup throws.

### Optimizely and custom account login

`AccountController` provides direct account login endpoints in addition to Azure AD and SAML-related flows. That controller can:

- sign users in through `UISignInManager`
- create Optimizely users on demand when a Salesforce-backed account exists but no local user exists yet
- issue create-account or reset-password style tokens
- trigger verification emails through `IEmailService`

## Authorization model

### Controller-level protection

`SalesforceAuthorizeAttribute` is the main custom authorization filter found in this pass. It can require account and location context and validates that:

- the account id exists when required
- the current user has access to the account
- the requested location exists for the current user
- the location belongs to the selected account when both are provided

Failure results are returned as JSON with `401` status codes, while unexpected exceptions produce a `500` JSON response.

### Context-driven access checks

Access control is not isolated to attributes. Controllers also call services such as:

- `IUserService.HasAccessToAccountAsync`
- `IUserService.HasAccessToLocationAsync`
- `IUserService.GetCurrentUserLocationByIdAsync`

This means access control is partly declarative and partly embedded in each feature flow.

## Operational implication

When debugging account-portal access issues, you need to check all of the following:

- Azure AD or SAML configuration
- Optimizely user synchronization
- Salesforce contact/account/location relationships
- host-name-specific redirect behavior
- controller-level access checks
