# Repository Map

The solution file contains seven .NET projects:

| Project                    | Type                 | Role                                                                       |
| -------------------------- | -------------------- | -------------------------------------------------------------------------- |
| `firstmile.web`            | ASP.NET Core web app | Runtime host, feature folders, CMS integration, middleware, HTTP endpoints |
| `FirstMile.Models`         | Class library        | Page types, block types, shared POCOs, constants, interfaces               |
| `FirstMile.Integration`    | Class library        | Integration-specific helpers such as CDN support                           |
| `FirstMile.Services`       | Class library        | Business logic and orchestration services                                  |
| `FirstMile.WebUtils`       | Class library        | Shared utilities used by the application stack                             |
| `FirstMile.Salesforce`     | Class library        | Salesforce integration layer                                               |
| `FirstMile.Services.Tests` | Test project         | Unit tests for service-layer behavior                                      |

## Backend folders worth knowing first

### `firstmile.web/Infrastructure`

This is the main composition root. It contains `Program.cs`, `Startup.cs`, partial startup extensions, authentication wiring, environment helpers, and service collection extensions.

### `firstmile.web/Api`

This folder contains backend HTTP endpoints for cart, orders, location, documents, invoices, promo codes, recurring orders, and related flows.

### `firstmile.web/Features`

This folder organizes pages, page models, controllers, shared components, and presentation logic by feature area instead of a flat MVC structure.

### `FirstMile.Models`

This project is split into content and support areas such as `Pages`, `Blocks`, `EditorDescriptors`, `Constants`, `Interfaces`, `Poco`, and `Users`.

## Frontend folders worth knowing first

### `firstmile.ui`

Pattern Lab plus gulp based frontend source. It contains the main static asset build flow, Pattern Lab commands, and the `inte` packaging command used by the frontend integration pipeline.

### `firstmile.widgets`

Vite and React based widget build. It produces compiled frontend widgets that are included in the frontend packaging flow.

### `firstmile.patterns`

Pattern outputs and HTML documentation assets. The frontend integration pipeline checks for changes here when deciding whether to create the release package branch.

## Existing documentation sources

Before the mdBook scaffold, the repository had standalone notes in the root `docs` folder:

- `branching.md`
- `db-and-logs.md`
- `git-hooks.md`

Those files remain as migration sources, while the mdBook chapters in `docs/src` become the new primary navigation surface.
