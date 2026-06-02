# Overview

FirstMile is an Optimizely CMS based ASP.NET Core application with a layered backend, a Pattern Lab based frontend build, and a separate React widget build. The repository contains both the runtime web application and the supporting libraries that model content, implement business behavior, integrate with Salesforce, and support deployment to Optimizely DXP.

## Top-level structure

- `firstmile.web`: main runtime application, middleware pipeline, feature folders, controllers, Razor views, and DXP host.
- `FirstMile.Models`: CMS content types, domain models, constants, descriptors, and shared interfaces.
- `FirstMile.Services`: application-level business services such as cart, order, price, search, keycloak, email, and stripe orchestration.
- `FirstMile.Salesforce`: Salesforce-facing service implementations and handlers.
- `FirstMile.WebUtils`: shared utility library used by service and web layers.
- `FirstMile.Integration`: integration-specific helpers, currently centered on CDN support.
- `FirstMile.Services.Tests`: unit tests for the service layer.
- `firstmile.ui`: Pattern Lab and gulp driven frontend source.
- `firstmile.widgets`: Vite and React based widget bundle.
- `firstmile.patterns`: generated or curated frontend pattern assets.

## Runtime model

At runtime, requests enter `firstmile.web`, pass through ASP.NET Core middleware configured in `Startup`, then flow into either Optimizely content routing or controller endpoints under `firstmile.web/Api`. Controllers and page handlers delegate to services from `FirstMile.Services`, which in turn rely on `FirstMile.Salesforce`, `FirstMile.Models`, and shared utilities.

## Deployment model

- Backend deployments are branch-driven from `inte`, `prep`, and `prod` through `azure-pipelines.yml`.
- Frontend integration packaging is driven from `develop` and can be manually run for `inte` and `prep` via `firstmile.ui/azure-pipelines-integration.yml`.
- The frontend pipeline generates compiled assets, commits them into a release package branch, and optionally deploys a static site for the `develop` flow.

## Key characteristics

- Common .NET target framework is `net8.0` via `Common.targets`.
- The web project is an Optimizely CMS 12 application using ASP.NET Core hosting.
- The solution couples CMS content modeling with commerce, account, reporting, and Salesforce-backed account data.
- Some important behavior is encoded indirectly through startup extensions, environment checks, and domain constants rather than one central design document.
