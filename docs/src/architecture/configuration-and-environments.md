# Configuration And Environments

Configuration and environment behavior are implemented in code, not only in environment-specific appsettings files.

## Configuration precedence

In development, `Program.cs` explicitly builds configuration from the following sources in order:

1. `appsettings.json`
2. `appsettings.{Environment}.json`
3. `appsettings.{MachineName}.json`
4. user secrets
5. environment variables

This matters because local runtime behavior can differ significantly between developers if machine-specific settings or user secrets override shared defaults.

`Host.CreateDefaultBuilder(args)` also contributes the standard ASP.NET Core configuration sources. The explicit development configuration inside `Program.cs` is primarily there to initialize Serilog before the web host starts.

## Environment detection

`firstmile.web/Infrastructure/Development.cs` centralizes environment flags:

- `IsDevelopment`
- `IsIntegration`
- `IsPreproduction`
- `IsProduction`
- `IsAzure`, which is true for integration, preproduction, or production

These flags are used by startup extensions and image/cache behavior, so environment changes are not purely declarative.

## Domain constants

`FirstMile.Models/Constants/DomainConstants.cs` hard-codes several important domains:

- local domain
- integration domain
- production domain
- local subdomain
- integration Optimizely subdomain
- production portal subdomain
- primary public domain

These constants influence URL rewriting and no-index behavior in `Startup.Configure`.

## Implemented environment-specific behavior

### Development

- sets `App_Data` under the content root
- disables the Optimizely scheduler
- enables Razor runtime compilation
- can support local-only routing experiments in commented rewrite logic

### Non-development

- enables HSTS
- enables Optimizely cloud platform support
- redirects to HTTPS permanently
- rewrites selected subdomain traffic toward `/login`
- relies on production-like host names defined in `DomainConstants`

### Production

- conditionally adds `X-Robots-Tag: noindex` for non-primary domains, tokenized requests, and selected protected paths

### Non-production

- always adds `X-Robots-Tag: noindex`

## Documentation risk

Because routing and indexing rules depend on both host names and path-prefix filtering, any environment or domain change should be documented and verified in the same pull request.

## Secrets handling note

The shared appsettings files define configuration sections for Azure AD, SAML, Salesforce, Keycloak, mail, Stripe, and reCAPTCHA. Those sections should be documented by purpose and ownership, but secrets themselves should not be repeated into handbook examples. Operationally, these values belong in secret stores or environment-specific secure configuration.

## Source anchors

- `firstmile.web/Infrastructure/Program.cs`
- `firstmile.web/Infrastructure/Development.cs`
- `FirstMile.Models/Constants/DomainConstants.cs`
- `firstmile.web/appsettings.json`
- `firstmile.web/appsettings.Development.json`
These files are the main verified sources for this chapter.

