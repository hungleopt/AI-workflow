# Configuration And Secrets Reference

This chapter maps the main configuration sections currently visible in the repository to their functional areas.

## Core app sections

| Section | Purpose | Primary usage |
| --- | --- | --- |
| `AzureAd` | Azure AD login configuration | OpenID Connect setup in `EngineConfigurator` |
| `Saml2` | SAML provider metadata and issuer settings | SAML setup in `EngineConfigurator` |
| `ConnectionStrings` | database and storage connection strings | startup and image/cache configuration |
| `SalesForceApiConfiguration` | Salesforce API credentials and endpoint | API client registration and auth flow |
| `KeyCloakApiConfiguration` | Keycloak API credentials and root URL | Keycloak client registration |
| `MailSettings` | SMTP settings | email sending services |
| `StripeSettings` | Stripe API keys | checkout/payment flow |
| `Authorization` | user-cache timing and related settings | Keycloak and user access behavior |
| `reCAPTCHA` | anti-bot settings | forms and submission validation |
| `FirstMileConfig` | local application-specific options | resized image cache on local environments |

## Content-managed configuration

Not all runtime settings live in appsettings files. `HomePage` currently acts as a content-managed configuration node for:

- login and account links
- checkout pages
- footer and navigation content
- mini-cart labels
- popup flags
- saved-basket and promo toggles
- Salesforce error email recipients

This split matters operationally: some behavior changes are CMS content changes, not configuration-file changes.

## Secrets guidance for this handbook

- document the existence and purpose of a secret, not the secret value
- point readers to the owning section or pipeline variable name
- treat committed secret-like values in shared files as implementation facts that should be remediated, not as examples to repeat elsewhere

## Pipeline secret variables already visible in automation

Verified secret variable names include:

- `SECRET_DXP_PROJECT_ID`
- `SECRET_DXP_CLIENT_KEY`
- `SECRET_DXP_CLIENT_SECRET`
- `SECRET_TEAMS_PUSH_CHANNEL`
- `SECRET_TEAMS_PUSH_SALT`
- `AZURE_STATIC_WEB_APPS_API_TOKEN`

## Ownership model to make explicit later

This repository shows at least four configuration ownership domains that should be assigned clearly in a future pass:

- identity and access
- Salesforce and external APIs
- payments and messaging
- DXP deployment and operations

## Source anchors

- `firstmile.web/appsettings.json`
- `firstmile.web/appsettings.Development.json`
- `firstmile.web/Infrastructure/EngineConfigurator.cs`
- `firstmile.web/Infrastructure/ServiceCollections/ApiServiceCollectionExtensions.cs`
- `FirstMile.Models/Pages/HomePage.cs`
- `azure-pipelines.yml`
- `firstmile.ui/azure-pipelines-integration.yml`
