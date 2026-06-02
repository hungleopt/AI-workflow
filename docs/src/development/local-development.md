# Local Development

The current repository already includes a minimal setup note in `firstmile.web/README.md`. This chapter folds that guidance into the mdBook structure and adds the broader repository context.

## Prerequisites

- .NET SDK 8.x is the effective baseline because all shared project targets come from `Common.targets` and set `TargetFramework` to `net8.0`
- SQL Server 2016 Express LocalDB or later
- Node.js 20 for the frontend integration pipeline and local frontend work
- Optimizely CLI if you need to create a new local CMS database

## Backend startup

From the repository root:

```bash
dotnet restore
dotnet build
dotnet run --project firstmile.web/FirstMile.Web.csproj
```

## Database setup options

The existing guidance suggests two approaches:

1. restore a recent database from integration
2. create an empty CMS database with Optimizely CLI

Example command from the existing README:

```bash
dotnet-episerver create-cms-database -S . -E -dn firstmile.cms -du firstmileadmin -dp firstmileadmin "firstmile.web.csproj"
```

## Expected local admin defaults

The current README recommends keeping local admin credentials aligned with integration:

- user: `admin`
- email: `admin@example.com`
- password: `Passw0rd#123`

## Frontend development

### Pattern Lab project

From `firstmile.ui`:

```bash
npm install
npm start
```

Useful alternatives:

- `npm run gulp:build`
- `npm run pl:build`
- `npm run inte`

### React widgets

From `firstmile.widgets`:

```bash
npm install
npm run watch
```

Use `npm run build` for a production bundle.

## Local configuration notes

- Development mode enables Razor runtime compilation.
- Development mode disables the Optimizely scheduler.
- Machine-specific appsettings and user secrets can override shared config and are often the source of environment drift.
