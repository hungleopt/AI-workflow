# Service Registration

The service map is assembled in `firstmile.web/Infrastructure/Startup.Services.cs` through a single `RegisterServices` method. Registrations are almost entirely scoped per request.

## Registration groups

The file effectively declares three groups.

### Salesforce-facing services

Examples include:

- `IAccountService`
- `IContactService`
- `IAuthService`
- `ICaseService`
- `ILocationService`
- `IProductService`
- `IRecurringsService`
- `IDocumentService`
- `IAccountStatementService`
- `IOrderHistoryService`
- `IReportingService`

These represent the lower-level integration and domain data access surface around Salesforce-backed operations.

### Web and application services

Examples include:

- `ICartService`
- `IOrderService`
- `IPriceService`
- `IRecurringOrderService`
- `IKeyCloakService`
- `IStripeService`
- `IPromoService`
- `ISettingsService`
- `IUrlService`
- `IFindService`
- `IOptimizelyUserService`

These services sit closer to request handling and feature orchestration.

### Request-scoped context

`IRequestContext` is registered as scoped and is used to carry request-level state through the service layer.

## Related infrastructure registrations

`Startup.ConfigureServices` adds other important infrastructure outside `RegisterServices`, including:

- distributed memory cache and session
- `IHttpContextAccessor`
- NotFoundHandler storage
- Salesforce and KeyCloak HTTP clients
- Stripe settings binding
- TinyMCE configuration
- security packages and recaptcha settings

## Architectural implication

Service boundaries are conventional rather than enforced by a separate dependency diagram. The most reliable way to understand the active service topology is to read `Startup.Services.cs` together with the project references in `firstmile.web/FirstMile.Web.csproj`.
