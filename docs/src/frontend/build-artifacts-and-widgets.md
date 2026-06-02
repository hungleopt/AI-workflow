# Build Artifacts And Widgets

The frontend output consumed by the backend comes from two cooperating build systems.

## Pattern Lab and gulp output

`firstmile.ui` is responsible for the main static asset and pattern build. The integration pipeline later checks for generated changes in:

- `firstmile.web/wwwroot/assets`
- `firstmile.patterns`

That indicates these generated directories are the contract surface consumed by the backend repository.

## React widget output

`firstmile.widgets/vite.config.ts` confirms several implementation details:

- widget bundles are emitted into `../firstmile.ui/widgets`
- hashed asset names are generated under `assets/app/`
- non-node-module code is bundled by feature, while node-module code is collapsed into a `vendor` manual chunk
- source maps are enabled
- production mode controls minification

The config also forces a Pattern Lab reload by touching a fake Handlebars file in `firstmile.ui/source/_patterns/atoms/fake` after bundle completion.

## Integration implication

The widget build is not isolated from the Pattern Lab project. It writes into the UI project and triggers Pattern Lab refresh behavior, so frontend architecture should be understood as one combined pipeline with two toolchains.
