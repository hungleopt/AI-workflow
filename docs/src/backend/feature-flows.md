# Feature Flows

This chapter captures the main functional areas visible from the repository layout. It is intentionally behavior-oriented rather than endpoint-oriented.

## Account and identity

The web project contains feature areas for login, forgotten password, account pages, restricted access, and SAML/AAD related controllers. Authentication and identity behavior are assembled in infrastructure code, while page features live in `firstmile.web/Features`.

Representative verified behavior from `AccountController`:

- login first checks whether the email exists in Salesforce-backed account data
- successful login clears cart and location state and updates the contact last-login date
- some host names trigger a special response that returns the credentials back as `dataNextStep`, indicating a cross-domain or delegated login handoff flow
- failed Optimizely login can trigger on-demand user creation and a create-account email workflow

## Commerce

Cart, checkout, payment, promo code, recurring order, and thank-you flows are spread across:

- `firstmile.web/Features/CartPage`
- `firstmile.web/Features/CheckoutStepOnePage`
- `firstmile.web/Features/CheckoutStepTwoPage`
- `firstmile.web/Features/ThankYouPage`
- controllers under `firstmile.web/Api`
- services such as `ICartService`, `IOrderService`, `IPriceService`, `IRecurringOrderService`, and `IStripeService`

Representative verified behavior:

- `CartService` persists cart state in ASP.NET Core session under `DefaultCart`
- logged-in carts are rebound to account and location context and enriched with Salesforce account/contact data
- `PaymentIntentController` creates or reuses cart payment context and returns Stripe `clientSecret` plus publishable key
- `CartController` currently contains substantial mutation logic and explicitly documents that some of it should move into the service layer later

## Service and reporting areas

Service pages, report pages, and reporting-related API controllers indicate a customer service and support workflow that includes reporting home, report issue, documents, invoices, and cases.

`OrderHistoryController` shows a common account-portal pattern:

- require location context
- verify current-user access to that location
- store selected location in the location service
- fetch paged Salesforce-backed data and reshape it into frontend-oriented JSON

`RecurringOrderController` shows another pattern where the API returns form and modal configuration objects rather than only raw recurring-order entities.

## Content and marketing

Blog, case study, news, press, home, range, and content pages indicate that the same application serves both CMS-driven marketing content and authenticated account workflows.

## Current limitation of this chapter

This pass identifies the verified feature areas but does not yet document each feature's exact request flow, backing services, or external system dependencies. That work should be done feature by feature in later passes.

## Source anchors

- `firstmile.web/Api/AccountController.cs`
- `firstmile.web/Api/CartController.cs`
- `firstmile.web/Api/OrderHistoryController.cs`
- `firstmile.web/Api/RecurringOrderController.cs`
- `FirstMile.Services/Commerce/CartService.cs`
  These files are the main verified sources for the current feature-flow notes.

## Location-home portal page (updated FMI-916, 2026-06-01)

The location-home page renders three sections inside `.fm-o-location-home__inner`:

1. **Scheduled Services** — static table + map + CTAs ("Order more sacks", "Manage scheduled services").
2. **Other waste and recycling services** — rc-collapse accordion for ad-hoc services (shredding table + order CTA), open by default on desktop, closed on mobile.
3. **One-off removal CTA** — standalone button below the accordion; dispatches `orderOneOffRemovalServices` to open the existing order-flow popup.

Section heading strings come from `labelsAndTexts` on `LocationHomeModel`. All six heading fields are optional with fallbacks so pages not yet updated in the CMS continue to work (see `docs/DECISIONS.md` DECISION-001).

The `more-than-3-items` styleModifier is applied to `NavigationMenu` when `navMenu.length > 3`.

`getLocationHomeRemovalTableData` was removed — one-off removals no longer use a table.

Source: `firstmile.widgets/src/blocks/location-home/LocationHome.tsx`
Skill file: `.ai/skills/location-home.md`
