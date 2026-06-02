# Branching And Releases

This chapter replaces the earlier standalone branching note with mdBook-friendly content.

## Branch roles in the current repository

- `develop`: base branch for ongoing frontend and shared development flow
- `inte`: backend integration deployment branch
- `prep`: backend preproduction deployment branch
- `prod`: backend production deployment branch

## Backend release flow

When code is merged into `inte`, `prep`, or `prod`, `azure-pipelines.yml` can trigger a deployment pipeline for the matching Optimizely DXP environment.

The backend pipeline watches changes in:

- `FirstMile.Integration/*`
- `FirstMile.Models/*`
- `FirstMile.Salesforce/*`
- `FirstMile.Services/*`
- `firstmile.web/*`
- `FirstMile.WebUtils/*`
- `FirstMile.sln`

Environment mapping implemented in the pipeline:

- `inte` -> Integration
- `prep` -> Preproduction
- `prod` -> Production

## Frontend release flow

When frontend changes land on `develop`, the frontend integration pipeline can:

1. build frontend assets
2. generate a release package branch such as `releases/develop-fe-package`
3. open a PR back to the target backend branch
4. deploy the static frontend site for demonstration use

The same pipeline supports manual runs from `inte` and `prep`, which retarget the release package branch and PR destination to those branches.

## Practical release implication

The repository has two overlapping release mechanics:

- direct backend packaging and DXP deployment from environment branches
- frontend asset generation that feeds backend branches through auto-generated PRs

That makes branch selection part of the implementation process, not only a source control concern.
