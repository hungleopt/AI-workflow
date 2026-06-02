# Branching

The base branch for both frontend and backend team is `develop`.

[[_TOC_]]

## Backend workflow

When a developer merges backend code into the `inte` branch, the repository automatically triggers the [**Deploy to DXP**](https://episerveremea-expertservices.visualstudio.com/First%20Mile/_build?definitionId=43) pipeline.

::: mermaid
sequenceDiagram
  autonumber
  actor dev as Developer
  participant repo as inte branch
  participant bebuild as BE Pipeline
  participant dxp as DXP INTE environment

  dev ->> repo: merge code

  repo ->> bebuild: auto trigger
  activate bebuild
  bebuild ->> bebuild: create deployment package
  bebuild ->> dxp: auto deploy
  deactivate bebuild
:::

A commit/PR is classified as a **backend change** if it includes at least one modified file in:

- FirstMile.Integration/*
- FirstMile.Models/*
- FirstMile.Salesforce/*
- FirstMile.Services/*
- firstmile.web/*
- FirstMile.WebUtils/*
- FirstMile.sln

Once complete, the code is deployed to the DXP's [INTE](https://inte.thefirstmile.co.uk/) environment.

> In addition to the `inte` branch, the backend pipeline can also be ***triggered automatically*** when changes are made to the `prep` or `prod` branches.  
> The workflow is the same as for `inte`, except the code will be deployed to the [PREP](https://inte.thefirstmile.co.uk/) or [LIVE](https://www.thefirstmile.co.uk/) environments.

## Frontend workflow

When a developer merges frontend code into the `develop` branch, the repository automatically triggers the [**FE Integration Package**](https://episerveremea-expertservices.visualstudio.com/First%20Mile/_build?definitionId=42) pipeline.

::: mermaid
sequenceDiagram
  autonumber
  actor dev as Developer
  participant repo as develop branch
  participant febuild as FE Integration pipeline
  participant festatic as FE static site

  dev ->> repo: merge code

  repo ->> febuild: auto trigger
  activate febuild
  febuild ->> febuild: create FE package
  febuild -->> repo: auto PR
  febuild ->> festatic: auto deploy
  deactivate febuild

  opt Auto PR
    dev ->> repo: approve auto PR
    repo ->> repo: merge auto PR
  end
:::

A commit/PR is considered a **frontend change** if it includes at least one modified file in:

- firstmile.ui/*
- firstmile.widgets/*

Once completed, an auto PR to the `develop` branch will be created. This PR includes the compiled frontend package (CSS, JS, etc.) required by the backend team. The frontend developer is expected to review, approve, and merge this auto PR so it will be ready for integration.

The pipeline also deploys the output to the [Frontend static site](https://tfm.eshn.dev/) which is used for demonstration purposes only.

> In addition to the `develop` branch, the frontend pipeline can also be ***triggered manually*** from the following branches: `inte`, `prep`.  
> The workflow is identical to the one for `develop`, except the auto PR will target the same branch that triggered it (`inte` or `prep`).

@<4BC985DD-3E00-6AB0-A624-EFFB51FCE675> @<E9C74D4C-8C0D-6C9A-8642-8DD62196CD56> @<D916C7EC-DCCF-64CA-A52E-98350C602B0B> @<43886725-1747-6F02-81B6-9B720B70AF06> 