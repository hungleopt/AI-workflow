# Content Types And Editing Model

The content model is centered in `FirstMile.Models` and uses Optimizely page and block types as the main editorial contract.

## Verified structure

The repository currently contains:

- 38 page model files under `FirstMile.Models/Pages`
- 96 block model files under `FirstMile.Models/Blocks`

Those counts show that editorial content modeling is a major part of the solution, not an auxiliary layer.

## Base page model

`SitePageData` is the main shared page base. It adds:

- `MainContentArea` constrained to selected content types such as `BaseBlockData`, PDF media, form containers, and CTA buttons
- metadata fields for SEO and social sharing
- layout toggles such as header/footer visibility and simplified header behavior
- redirect flags that participate in runtime navigation logic
- notification-banner and entry-point-popup settings

This means most page types inherit both presentation composition and runtime navigation behavior from the same base class.

## Base block model

`BaseBlockData` extends `BlockData` and implements `IHavePreview`. The shared property currently verified is:

- `ShowInAnchorMenu`

Even though the shared property set is small, using a common base gives the solution one editorial marker for preview-capable, site-standard blocks.

## Editorial hub page

`HomePage` acts as a high-value settings aggregate rather than only a marketing home page. Verified responsibilities include:

- account and login destination links
- checkout and thank-you page links
- navigation and footer content
- default CTA and sticky-banner references
- VAT and mini-cart labels
- popup, promo, and saved-basket feature switches
- Salesforce error notification recipients
- head and bottom script slots

Operationally, this makes `HomePage` a site-settings container as much as a content page.

## Page family patterns

Verified page families in `FirstMile.Models/Pages` include:

- account portal pages via `AccountBasePage`
- service-related pages via `ServiceBasePage`
- case-study pages via `CaseStudyBasePage`
- standalone content pages that inherit directly from `SitePageData`

Not every page model follows the same base path. For example, `PromoCodePage` derives directly from `PageData`, which is worth treating as an exception when documenting content conventions.

## Editing-model implication

Editorial behavior is defined by a combination of:

- Optimizely attributes such as `ContentType`, `Display`, `AllowedTypes`, `CultureSpecific`, and `UIHint`
- shared tab-name constants
- block and page inheritance
- page-level feature flags that affect runtime redirects and layout

That means the content model is not only about schema. It is also a runtime configuration surface.

## Source anchors

- `FirstMile.Models/Pages/SitePageData.cs`
- `FirstMile.Models/Blocks/BaseBlockData.cs`
- `FirstMile.Models/Blocks/IHavePreview.cs`
- `FirstMile.Models/Pages/HomePage.cs`
- `FirstMile.Models/Pages`
- `FirstMile.Models/Blocks`
