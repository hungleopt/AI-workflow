# Frontend Integration Pipeline

The frontend packaging workflow is defined in `firstmile.ui/azure-pipelines-integration.yml`.

## Triggering

The pipeline automatically triggers on `develop` when changes affect:

- `firstmile.ui/*`
- `firstmile.widgets/*`

It also contains branch-specific variable mappings for `develop`, `inte`, and `prep`.

## Branch mapping

### `develop`

- target backend branch: `develop`
- release package branch: `releases/develop-fe-package`
- static site deployment: enabled

### `inte`

- target backend branch: `inte`
- release package branch: `releases/inte-fe-package`
- static site deployment: disabled

### `prep`

- target backend branch: `prep`
- release package branch: `releases/prep-fe-package`
- static site deployment: disabled

## Verified pipeline steps

1. checkout with persisted credentials and full history
2. install Epi CLI
3. install Node.js 20
4. build the widgets package
5. run the frontend integration build in `firstmile.ui`
6. inspect generated changes under `firstmile.web/wwwroot/assets` and `firstmile.patterns`
7. if changes exist, commit generated artifacts and force-push the release package branch
8. create a pull request back to the target backend branch
9. for `develop` only, build gulp assets, build Pattern Lab, and deploy the static site

## Important implementation detail

This pipeline executes `git checkout -- .` after the widget build and again after the UI integration build. That indicates the pipeline expects generated and intermediate working-tree noise during the build, then intentionally relies on a later diff check for the final packaged output.

## Risk to document later

The pipeline force-pushes release branches. Any manual work on those branches would be fragile and should be treated as disposable unless a later process change formalizes them.
