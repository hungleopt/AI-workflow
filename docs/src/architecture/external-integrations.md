# External Integrations

FirstMile depends on several external systems and infrastructure services. Most of the runtime wiring is configured in startup extension methods and appsettings sections.

## Salesforce

Salesforce is the most pervasive integration boundary in the codebase.

Verified integration characteristics:

- HTTP client named `SalesForce`
- retry policy for transient errors plus `401 Unauthorized`
- automatic bearer-token reassignment through `ReassignSalesForceAccessTokenHandler`
- token refresh attempt through `IAuthService.PostAuthenticateAsync()` when a retry sees `401`
- service interfaces and implementations split between `FirstMile.Salesforce` and `FirstMile.Services`

Salesforce-backed behavior appears in authentication, account access, order history, recurring orders, documents, invoices, and location selection.

## Keycloak

The repository also wires a `KeyCloak` HTTP client with a similar retry and token-reassignment pattern. `ConfigureKeyCloakApis` binds config for:

- Keycloak API settings
- authorization settings
- `IKeyCloakService`

The coexistence of Azure AD, SAML, Optimizely users, and Keycloak makes identity behavior a multi-system concern rather than a single-provider flow.

## Stripe

Stripe is configured through `StripeAppSettingsConfiguration` and is used by `CartService` and `PaymentIntentController`.

Verified behavior in this pass:

- Stripe secret key is assigned during startup
- payment intents are created from cart totals or invoice amounts
- payment context is stored back into the session cart
- the payment intent response returned to the frontend includes the publishable key

## Optimizely platform components

The web project uses a significant Optimizely stack:

- CMS
- Forms
- Find
- cloud platform support
- CMS ASP.NET Identity integration
- TinyMCE editor customization

## Imaging and CDN support

Image processing is configured through ImageSharp. Depending on environment, the cache is backed by either:

- Azure Blob storage
- a local configured cache folder plus physical file system provider

`FirstMile.Integration.CdnSupport` participates in this path, which makes media behavior another environment-sensitive integration boundary.
