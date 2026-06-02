# Backend Deployment Pipeline

The backend deployment workflow is defined in `azure-pipelines.yml`.

## Triggering

The pipeline is configured to trigger on:

- `inte`
- `prep`
- `prod`

It is path-filtered to backend-related folders and the solution file.

The pipeline also uses a `CUSTOM_SHOULD_DEPLOY` condition to avoid deploying on selected automated commits and to allow manual runs.

## Build stage

The build stage performs the following verified steps:

1. checkout with submodules
2. print and validate pipeline variables
3. install .NET SDK `8.0.407`
4. install NuGet 6.x
5. run `dotnet restore`
6. run `dotnet test`
7. run `dotnet publish`
8. archive the published web output as a deployment package
9. generate a transitive package list
10. publish build artifacts

## Environment behavior

The pipeline maps each branch to an environment identifier, friendly name, deploy mode, slot usage, and validation URL.

### Integration

- direct deploy enabled
- no DB export

### Preproduction

- direct deploy disabled
- slot deployment enabled
- no DB export

### Production

- direct deploy disabled
- slot deployment enabled
- DB export enabled

## Required secrets

The pipeline comments document these required secret variables:

- `SECRET_DXP_PROJECT_ID`
- `SECRET_DXP_CLIENT_KEY`
- `SECRET_DXP_CLIENT_SECRET`

## Operational note

This pipeline is not only a build. It also encodes release policy, environment validation targets, and whether production backups are expected as part of the deployment flow.
