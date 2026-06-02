# Content Model And Features

The repository uses a feature-folder layout in the web project and a separate content-model project for Optimizely page and block types.

## Content model location

`FirstMile.Models` is the central content model library. Relevant folders include:

- `Pages`
- `Blocks`
- `EditorDescriptors`
- `PropertyTypes`
- `Users`
- `Constants`
- `Interfaces`
- `Poco`

This split keeps Optimizely content definitions out of the web project while leaving rendering and request behavior inside `firstmile.web`.

## Feature-folder layout

`firstmile.web/Features` groups UI and presentation logic by business area. Current examples include:

- account pages and restricted access flows
- blog and content pages
- cart and checkout
- payment invoice and thank you pages
- login, forgotten password, and SAML related endpoints
- custom admin pages
- robots and sitemap features

`Startup.ConfigureServices` configures Razor Pages to use `/Features` as the root directory, and routing also maps Optimizely content with a custom template for blog pagination.

## API surface next to feature pages

The repository keeps a separate `firstmile.web/Api` folder for backend JSON endpoints. This means feature behavior is split across:

- page or Razor feature code in `Features`
- controller endpoints in `Api`
- orchestration services in `FirstMile.Services`
- external data access in `FirstMile.Salesforce`

## Documentation implication

When documenting one user-visible feature, do not stop at the feature folder. Most important flows cross all four areas above.
