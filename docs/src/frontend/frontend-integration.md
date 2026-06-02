# Frontend Integration

The repository contains two distinct frontend build systems that feed the backend application.

## `firstmile.ui`

`firstmile.ui` is the Pattern Lab and gulp based frontend source. The `package.json` scripts show the main workflow:

- `gulp:build`: compile frontend assets
- `pl:build`: build Pattern Lab output
- `pl:serve`: run the Pattern Lab UI locally
- `start`, `dev`, and `watch`: build then run gulp watch together with Pattern Lab serving
- `inte`: build integration output through `gulp buildInte`, `patternlab build` with the integration config, and `gulp copyInte`

This project is the main source for the static frontend package that gets committed back into the repository.

## `firstmile.widgets`

`firstmile.widgets` is a Vite and React based widget build. Its scripts show two main modes:

- `build`: compile a production bundle
- `dev` or `watch`: compile in `eshn` mode and watch for changes

The dependency list shows Redux Toolkit, Stripe React bindings, Google Maps bindings, and Zod, which indicates widget-based interactive features beyond the static Pattern Lab build.

## Integration pipeline behavior

The frontend integration pipeline in `firstmile.ui/azure-pipelines-integration.yml` does the following:

1. checks out the repository with credentials persisted
2. installs Epi CLI and Node 20
3. builds widgets in `firstmile.widgets`
4. runs the integration build in `firstmile.ui`
5. checks whether generated assets or patterns changed
6. commits generated output and force-pushes it to a release package branch
7. opens a pull request back toward the target backend branch
8. for `develop` only, builds gulp output, builds Pattern Lab, and deploys the static site

## Practical implication

Frontend changes are not only shipped as deployed assets. They are also materialized as generated files committed into release package branches so backend-target branches can consume them.
