# Application Bootstrap

The backend application starts in `firstmile.web/Infrastructure/Program.cs` and delegates almost all application assembly to `Startup`.

## Entry point

`Program.Main` calls `CreateHostBuilder(args).Build().Run()`.

`CreateHostBuilder` does three important things:

1. Starts from `Host.CreateDefaultBuilder(args)` and applies `ConfigureCmsDefaults()` for Optimizely CMS hosting.
2. In development only, builds a configuration stack manually and initializes Serilog from configuration.
3. Configures the ASP.NET Core web host to use `Startup` and static web assets.

## Why `Startup` matters here

This repository keeps most architectural behavior in `Startup.cs` and its partial companion `Startup.Services.cs`. That means runtime behavior is spread across:

- service registration
- environment-specific middleware
- routing configuration
- security/authentication setup
- static file and cache behavior
- outbound API client registration through extension methods

## Request pipeline summary

The implemented request pipeline in `Startup.Configure` is, at a high level:

1. Development exception page locally, or exception handler plus HTTPS redirect and HSTS outside development.
2. `X-Robots-Tag` handling to keep non-production traffic out of indexing and to protect selected production paths.
3. Trailing slash removal for HTTPS requests, excluding selected Optimizely paths.
4. static file, image processing, and CDN support.
5. routing, authentication, authorization, session, and cookie policy.
6. controller routes, Razor pages, and Optimizely content routing including the custom blog paging route `p{page:int}/`.
7. not found handling via Geta NotFoundHandler.

## Architectural implication

There is no separate architecture layer or orchestrator project. The practical composition root for the whole application is the combination of:

- `Program.cs`
- `Startup.cs`
- `Startup.Services.cs`
- infrastructure extension methods such as API client registration and authentication helpers

Anyone documenting or changing cross-cutting behavior should inspect those files first.
