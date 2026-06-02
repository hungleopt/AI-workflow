# FMI-946: (June) GZ Integration: Location-level and Product-level VAT exemption

> Last Reviewed: 2026-05-28 07:39 UTC  
> Status: Is Estimated  
> Type: Story

## 1. Questions, Assumptions & Decisions

### Open Questions (Needs Answer)

- [ ] Order creation posting net unit prices to Salesforce — Caiti confirmed current process should remain the same but noted: "we can confirm after discussion with the Finance Team." Await Finance Team sign-off before assuming no change needed here.

### Assumptions

- The Salesforce fields `Account.IsVATExemptLocation__c`, `Product2.IsVATExempt__c`, and additional-charge response field `isVatExempt` are the authoritative sources for exemption decisions. The old `VAT_Exempt__c` field is being removed. **(Confirmed by PO)**
- Order creation continues posting net unit prices, and Salesforce remains responsible for downstream order tax persistence. The required application changes are primarily in cart, checkout, payment-intent, and summary calculations. **(Provisionally confirmed — pending Finance Team review)**

### Decisions

- **Location-level VAT exemption overrides all per-line flags**: If the selected location is VAT exempt, the entire order is VAT exempt (no VAT on any product or surcharge), regardless of the per-line `isVatExempt` API response values. **(Confirmed by Caiti Black, 2026-05-27)**
- **Fuel surcharge / additional charges are treated like any other product**: When the location is NOT VAT exempt, the `isVatExempt` flag from the `ProcessAdditionCharges` API response applies per-line. Fuel surcharge VAT exemption depends on the product setup, not on special handling. **(Confirmed by Caiti Black, 2026-05-27)**
- **`ProcessAdditionCharges` can return 0 to many additional charges**: Each row is an additional charge product with its own VAT exemption status. Implementation must handle multiple surcharge lines, not assume a single row. **(Corrected by Caiti Black, 2026-05-27)**
- **Guest checkout**: Location-level VAT exemption does NOT apply (account/location not created yet per FMI-901). Product-level VAT exemption (`Product2.IsVATExempt__c`) DOES still apply during guest checkout since it lives on the product itself. **(Confirmed by Caiti Black, 2026-05-27)**
- Classify this ticket as `STANDARD`. It changes billing logic across multiple modules, and the safety override applies because tax and payment totals are affected.
- Split execution into two BE tasks: (1) plumb VAT-exemption data from Salesforce and checkout surcharge responses, (2) update all cart, checkout, payment, and summary tax calculations that currently assume one global VAT rate.
- Keep existing VAT amount fields in backend payloads, but add `isVatExempt` on the cart and each cart item so the frontend can hide the VAT row when the selected location is exempt.

## 2. Proposed Implementation

### Approach

Introduce explicit `isVatExempt` flags at the location, product, surcharge, cart, and line-item levels, then replace the current single-rate cart math with taxable-subtotal math. Keep VAT amount fields in the payload for compatibility, but let the frontend hide the VAT row based on the new flags.

**Key rules (confirmed by PO 2026-05-27):**
- Location VAT exempt → entire order VAT exempt (overrides all per-line flags).
- Location NOT VAT exempt → per-line `isVatExempt` flags determine each product's VAT status.
- Additional charges (fuel surcharge, etc.) are treated like any other product for VAT purposes.
- `ProcessAdditionCharges` can return 0–N lines; each is an independent product with its own VAT status.
- Guest checkout: no location-level exemption (account not yet created), but product-level exemption still applies.

The current implementation is globally VAT-based, not line-aware:

- `PriceService` starts from one VAT percentage and stores it once on the cart at `FirstMile.Services/Commerce/PriceService.cs:117`, `FirstMile.Services/Commerce/PriceService.cs:186`, `FirstMile.Services/Commerce/PriceService.cs:252`, `FirstMile.Services/Commerce/PriceService.cs:275`, and `FirstMile.Services/Commerce/PriceService.cs:417`.
- `Cart` computes VAT and total from that single `hiddenVat` value at `FirstMile.Services/Commerce/Models/Cart.cs:44`, `FirstMile.Services/Commerce/Models/Cart.cs:50`, `FirstMile.Services/Commerce/Models/Cart.cs:80`, `FirstMile.Services/Commerce/Models/Cart.cs:90`, and `FirstMile.Services/Commerce/Models/Cart.cs:105`.
- Stripe order amounts repeat the same global formula in `FirstMile.Services/Commerce/CartService.cs:284`, `FirstMile.Services/Commerce/CartService.cs:285`, and `firstmile.web/Api/PaymentIntentController.cs:39`.

That architecture cannot express a mixed order where some items are taxable and some are not, so the fix needs to move tax calculation from cart-level percentage only to taxable-subtotal plus line metadata.

### Solution Details

1. **Expose location VAT exemption on cached location models**

   The live-location queries currently hydrate only `Id`, `ParentId`, `Name`, partnership, postcode, and pricebook fields in `FirstMile.Salesforce/Services/UserService.cs:252` and `FirstMile.Salesforce/Services/UserService.cs:311`. `FullLocationModel` currently exposes `IsFuelSurchargeExemptLocation__c` at `FirstMile.Salesforce/Models/FullLocationModel.cs:101`, but no VAT exemption field. Add `IsVATExemptLocation__c` to the location models and the relevant SOQL selects so the selected location carries VAT state without an extra ad hoc lookup.

1. **Expose product-level VAT exemption in pricing queries**

   `ProductService` pricing queries do not currently request any VAT-exempt product field in the core product fetches at `FirstMile.Salesforce/Services/ProductService.cs:21`, `FirstMile.Salesforce/Services/ProductService.cs:35`, `FirstMile.Salesforce/Services/ProductService.cs:85`, and `FirstMile.Salesforce/Services/ProductService.cs:94`. Add `IsVATExempt__c` to the product models and those SOQL queries, then carry that flag onto `LineItem` so every line knows whether VAT should apply.

1. **Expose additional-charge VAT exemption**

   `AdditionalChargeItemResponse` currently only maps price, quantity, product name, product id, and pricebook entry id at `FirstMile.Salesforce/Models/FuelSurgeChargeModel.cs:18`, `FirstMile.Salesforce/Models/FuelSurgeChargeModel.cs:20`, and `FirstMile.Salesforce/Models/FuelSurgeChargeModel.cs:32`. Add `isVatExempt` and copy it onto the synthetic surcharge `LineItem` created in `firstmile.web/Features/CheckoutStepTwoPage/CheckoutStepTwoPageController.cs:51`, `firstmile.web/Features/CheckoutStepTwoPage/CheckoutStepTwoPageController.cs:61`, and `firstmile.web/Features/CheckoutStepTwoPage/CheckoutStepTwoPageController.cs:73`. Note: the API can return 0–N additional-charge lines, so the implementation must iterate all returned items, not assume a single surcharge row.

1. **Mirror location VAT exemption into cart state using the existing location-selection pattern**

   The repo already mirrors fuel-surcharge exemption from the selected location into the session cart in `firstmile.web/Api/LocationController.cs:238` and `firstmile.web/Features/AccountLocationPage/AccountLocationPageController.cs:79`. Reuse the same pattern for location VAT exemption so cart and checkout calculations do not depend on requerying the selected location at render time. Surface that state as `cart.isVatExempt`, and stamp `lineItem.isVatExempt` for each priced or surcharge line.

1. **Replace global VAT math with taxable-subtotal math**

   The current cart summary values come from global formulas in `Cart.cs` and `CartService.cs`, and duplicate logic also exists in:

   - `firstmile.web/Api/CartController.cs:102`, `firstmile.web/Api/CartController.cs:115`, `firstmile.web/Api/CartController.cs:176`, `firstmile.web/Api/CartController.cs:229`, and `firstmile.web/Api/CartController.cs:237`
   - `firstmile.web/Features/CartPage/CartPageController.cs:84`
   - `firstmile.web/Features/CheckoutStepTwoPage/CheckoutStepTwoPageController.cs:250`
   - `firstmile.web/Features/ThankYouPage/ThankYouPageController.cs:24`, `firstmile.web/Features/ThankYouPage/ThankYouPageController.cs:28`, and `firstmile.web/Features/ThankYouPage/ThankYouPageController.cs:50`
   - `firstmile.web/Api/SavedBasketController.cs:184` and `firstmile.web/Api/SavedBasketController.cs:185`

   Introduce helper logic on cart and/or line items to compute:

   - taxable subtotal
   - VAT total based only on taxable lines unless the location is wholly exempt
   - grand total from net subtotal plus computed VAT

   Keep `hiddenVat` as the configured VAT rate label if needed for UI copy, but stop treating it as proof that all lines are taxable. The frontend should continue receiving VAT values while using `cart.isVatExempt` and line-item flags to decide whether to hide the VAT row.

1. **Review order submission only for regressions, not as a primary change target**

   The current order submission path posts Order and OrderItem records in `FirstMile.Services/OrderService.cs:1103`, `FirstMile.Services/OrderService.cs:1194`, and `FirstMile.Services/OrderService.cs:1198`. No explicit server-side VAT math surfaced in the reviewed slice, so the planned change is to verify that posted unit prices and Salesforce-side tax behavior remain correct once the cart and checkout totals are fixed.

## 3. Detailed Task List

### 3.1 Models & Configuration

| #   | File Path                                             | Action | Description                                                                                                         |
| --- | ----------------------------------------------------- | ------ | ------------------------------------------------------------------------------------------------------------------- |
| 1   | `FirstMile.Salesforce/Models/SimpleLocationModel.cs`  | Modify | Add `IsVATExemptLocation__c` so cached location records carry location-level VAT state.                             |
| 2   | `FirstMile.Salesforce/Models/FullLocationModel.cs`    | Modify | Add `IsVATExemptLocation__c` to the full location projection alongside the existing fuel-surcharge exemption field. |
| 3   | `FirstMile.Salesforce/Models/SimpleProductModel.cs`   | Modify | Add `IsVATExempt__c` to product models used by pricing and reorder flows.                                           |
| 4   | `FirstMile.Salesforce/Models/FullProductModel.cs`     | Modify | Carry the product VAT-exempt flag through the richer product projection.                                            |
| 5   | `FirstMile.Salesforce/Models/FuelSurgeChargeModel.cs` | Modify | Add `isVatExempt` to additional-charge response payload mapping.                                                    |
| 6   | `FirstMile.Services/Commerce/Models/Cart.cs`          | Modify | Add cart-level `isVatExempt` state and new computed helpers for taxable subtotal and VAT display rules.             |
| 7   | `FirstMile.Services/Commerce/Models/LineItem.cs`      | Modify | Add per-line `isVatExempt` state so totals can be computed per item rather than from a single cart rate.            |

### 3.2 Services & Business Logic

| #   | File Path                                             | Action | Description                                                                                                                   |
| --- | ----------------------------------------------------- | ------ | ----------------------------------------------------------------------------------------------------------------------------- |
| 1   | `FirstMile.Salesforce/Services/UserService.cs`        | Modify | Extend live-location SOQL projections so `IsVATExemptLocation__c` is available from cached location data.                     |
| 2   | `FirstMile.Salesforce/Services/ProductService.cs`     | Modify | Add `IsVATExempt__c` to all product queries that feed pricing, reorder, and checkout flows.                                   |
| 3   | `FirstMile.Services/Commerce/PriceService.cs`         | Modify | Stamp per-line VAT-exempt flags from location and product data while keeping the configured VAT percentage for taxable lines. |
| 4   | `FirstMile.Services/Commerce/CartService.cs`          | Modify | Replace global VAT total math with taxable-subtotal logic for Stripe-backed payment intent creation.                          |
| 5   | `FirstMile.Salesforce/Services/SurgeChargeService.cs` | Review | Confirm list-vs-single-item endpoint behavior after `isVatExempt` is added to the response contract.                          |

### 3.3 Integration

| #   | File Path                                                                     | Action | Description                                                                                                                 |
| --- | ----------------------------------------------------------------------------- | ------ | --------------------------------------------------------------------------------------------------------------------------- |
| 1   | `FirstMile.Salesforce/Models/FuelSurgeChargeModel.cs`                         | Modify | Update additional-charge response mapping to support VAT-exempt surcharge lines.                                            |
| 2   | `firstmile.web/Features/CheckoutStepTwoPage/CheckoutStepTwoPageController.cs` | Modify | Preserve `isVatExempt` on the synthetic additional-charge line item created during checkout.                                |
| 3   | Salesforce `Account` and `Product2` SOQL projections                          | Modify | Ensure location and product exemption fields are present wherever repricing and checkout selection depend on those records. |

### 3.4 Controllers & Endpoints

| #   | File Path                                                                     | Action | Description                                                                                                                                    |
| --- | ----------------------------------------------------------------------------- | ------ | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | `firstmile.web/Api/LocationController.cs`                                     | Modify | Mirror location VAT exemption into session cart at the same point that fuel-surcharge exemption is currently set.                              |
| 2   | `firstmile.web/Features/AccountLocationPage/AccountLocationPageController.cs` | Modify | Mirror location VAT exemption when users land on location home and the cart is rebound to the selected location.                               |
| 3   | `firstmile.web/Api/CartController.cs`                                         | Modify | Remove assumptions that every cart line uses one global taxable rate when rebuilding cart response payloads.                                   |
| 4   | `firstmile.web/Features/CartPage/CartPageController.cs`                       | Modify | Continue returning VAT values but include `cart.isVatExempt` so the frontend can hide the VAT row for fully exempt carts.                      |
| 5   | `firstmile.web/Features/CheckoutStepTwoPage/CheckoutStepTwoPageController.cs` | Modify | Continue returning VAT values but include cart and item `isVatExempt` flags for frontend VAT-row hiding and exempt-line handling.              |
| 6   | `firstmile.web/Api/PaymentIntentController.cs`                                | Modify | Align API-side payment intent amount calculation with the same taxable-subtotal logic as `CartService`.                                        |
| 7   | `firstmile.web/Features/ThankYouPage/ThankYouPageController.cs`               | Modify | Stop reconstructing per-line VAT-inclusive prices from a single `hiddenVat` percentage when the order contains mixed exempt and taxable lines. |
| 8   | `firstmile.web/Api/SavedBasketController.cs`                                  | Modify | Update saved-basket email VAT display so exempt carts do not show misleading VAT output.                                                       |

### 3.5 UI & Frontend

| #   | File Path                                | Action | Description                                                                                                                              |
| --- | ---------------------------------------- | ------ | ---------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | Cart and checkout backend JSON payloads  | Modify | Add `isVatExempt` flags while keeping existing VAT amount fields so the frontend can hide the VAT row without a breaking payload change. |
| 2   | `firstmile.ui/` and `firstmile.widgets/` | Review | No direct source change expected unless a widget hardcodes VAT-row visibility.                                                           |

### 3.6 Wiring & DI

| #   | File Path             | Action | Description                                                                  |
| --- | --------------------- | ------ | ---------------------------------------------------------------------------- |
| —   | No DI change expected | —      | Existing services and controllers already own the required responsibilities. |

### 3.7 Unit Tests

| #   | Test File Path                                                                           | Tests to Add                                                                                                                                        | Covers                                                    |
| --- | ---------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------- |
| 1   | `FirstMile.Salesforce.Tests/Services/UserServiceTests.cs`                                | `GetLiveLocationsForCurrentUser_MapsLocationVatExemption`, `GetCurrentUserLocationByIdAsync_FullLocation_MapsLocationVatExemption`                  | Location VAT field mapping and cache hydration            |
| 2   | `FirstMile.Salesforce.Tests/Services/ProductServiceTests.cs`                             | `GetProductsByRegionZoneAndProductCodesAsync_MapsProductVatExemption`, `GetByIdsAsync_MapsProductVatExemption`                                      | Product VAT field mapping from Salesforce queries         |
| 3   | `FirstMile.Services.Tests/Commerce/PriceServiceTests.cs`                                 | `GetPriceForAnonymousUser_VatExemptProduct_MarksLineItemNonTaxable`, `GetPriceForOrderAgain_VatExemptLocation_MarksAllLineItemsNonTaxable`          | Price-service propagation of VAT-exempt metadata          |
| 4   | `FirstMile.Services.Tests/Commerce/CartServiceTests.cs`                                  | `SaveCartWithPaymentIntent_MixedTaxableItems_ChargesVatOnTaxableSubtotalOnly`, `SaveCartWithPaymentIntent_VatExemptLocation_ChargesNetSubtotalOnly` | Stripe/payment amount calculation                         |
| 5   | `firstmile.web.Tests/Features/CheckoutStepTwoPage/CheckoutStepTwoPageControllerTests.cs` | `Index_AdditionalChargeVatExempt_DoesNotIncreaseVatTotal`, `Index_VatExemptLocation_SetsIsVatExemptFlag`                                            | Checkout surcharge handling and frontend VAT-row flagging |
| 6   | `firstmile.web.Tests/Features/CartPage/CartPageControllerTests.cs`                       | `Index_MixedTaxableCart_ShowsPartialVat`, `Index_VatExemptLocation_SetsCartIsVatExemptFlag`                                                         | Cart summary output and frontend VAT-row flagging         |
| 7   | `firstmile.web.Tests/Features/ThankYouPage/ThankYouPageControllerTests.cs`               | `Index_MixedTaxOrder_UsesStoredLineTaxStateNotGlobalVatRate`                                                                                        | Thank-you summary regression coverage                     |

Test file location convention:

```text
Source:  FirstMile.Salesforce/Services/UserService.cs
Test:    FirstMile.Salesforce.Tests/Services/UserServiceTests.cs

Source:  FirstMile.Services/Commerce/PriceService.cs
Test:    FirstMile.Services.Tests/Commerce/PriceServiceTests.cs

Source:  firstmile.web/Features/CheckoutStepTwoPage/CheckoutStepTwoPageController.cs
Test:    firstmile.web.Tests/Features/CheckoutStepTwoPage/CheckoutStepTwoPageControllerTests.cs
```

### 3.8 Documentation

| #   | Doc File Path                             | Action | Description                                                                                                             |
| --- | ----------------------------------------- | ------ | ----------------------------------------------------------------------------------------------------------------------- |
| 1   | `docs/src/backend/cart-and-order-flow.md` | Update | Replace the current single-rate VAT description with line-aware taxable-subtotal behavior and location-exemption notes. |
| 2   | `docs/src/tickets/FMI-946-review.md`      | Create | This review document.                                                                                                   |

## 4. QA Verification Notes

### Test Scenarios

| #   | Scenario                            | Steps                                                                                        | Expected Result                                                                                                               |
| --- | ----------------------------------- | -------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| 1   | VAT-exempt location cart            | Select location `Inditex Ireland (Head office)` and open cart                                | VAT is not added to totals; backend still returns VAT fields, but `cart.isVatExempt = true` so the frontend hides the VAT row |
| 2   | VAT-exempt location checkout        | Continue the same exempt location to checkout step two                                       | VAT is not added to checkout totals or payment intent amount; checkout payload exposes `isVatExempt` flags for VAT-row hiding |
| 3   | Mixed taxable and exempt products   | Use non-exempt location `Rob Test Location`, add a VAT-exempt product and a taxable product  | VAT total is calculated only from the taxable product lines                                                                   |
| 4   | VAT-exempt additional charge        | Trigger additional charge for an exempt product line if the API returns `isVatExempt = true` | The additional charge line is added, but it does not contribute VAT                                                           |
| 5   | Standard taxable order regression   | Use a normal taxable location and taxable products only                                      | Cart, checkout, payment, thank-you, and saved-basket outputs still match previous VAT behavior                                |
| 6   | Fuel surcharge exemption regression | Use a fuel-surcharge-exempt location                                                         | Fuel surcharge remains suppressed and VAT logic does not reintroduce the surcharge line                                       |
| 7   | Guest checkout with exempt product  | Complete guest checkout with a VAT-exempt product (no location created yet)                  | Product-level VAT exemption applies; location-level exemption does NOT apply since no account exists                          |
| 8   | Multiple additional charges         | Trigger an order where `ProcessAdditionCharges` returns 2+ lines with mixed VAT exemption    | Each additional charge line respects its own `isVatExempt` flag independently                                                 |

### Edge Cases to Verify

- VAT-exempt location with all products taxable by default: the whole order should still remain VAT exempt (location overrides everything — confirmed).
- Mixed order with promo discount applied: VAT is calculated on the discounted taxable subtotal only if that is the current business rule.
- Additional charges with multiple lines: verify that each line's `isVatExempt` flag is respected independently when location is NOT exempt.
- Guest checkout with VAT-exempt products: product-level exemption must apply even without a location (confirmed).
- Guest checkout with no exempt products: standard VAT applies to all lines.
- Thank-you page and saved-basket email after an exempt order: no stale VAT amount should appear from `TempData` or email rendering.
- `ProcessAdditionCharges` returns 0 additional charges: no surcharge lines added, VAT unaffected.

### Regression Areas

- Cart JSON payloads returned by `CartController`
- Cart page summary output
- Checkout step two summary and Stripe payment-intent creation
- Thank-you page price rendering
- Saved-basket email VAT row
- Existing fuel-surcharge exemption flow on location selection

### Test Data Requirements

- VAT-exempt account: `Inditex Ireland (Head Office)`
- VAT-exempt location: `Inditex Ireland (Head office)`
- Example exempt-location order: `07860834`
- Non-exempt location with exempt product order: `Rob Test Location`
- Example product-level exemption order: `07860836`
- Environment: Salesforce full sandbox / integration environment used by the ticket

## 5. Risks & Concerns

### Security

- Payment totals are safety-critical. All VAT changes must remain server-side and derived from trusted Salesforce fields and the additional-charge payload, not from client-supplied flags.
- `PaymentIntentController` and `CartService` both calculate charge amounts today, so they must stay aligned after the refactor or the UI and charged amount can diverge.

### Compliance

- This ticket changes tax presentation and payment amounts. The implementation must ensure that exempt locations suppress VAT consistently across cart, checkout, emails, and post-purchase summaries, otherwise customers may be shown the wrong tax amount.
- If promo discounts affect taxable basis, QA should explicitly verify whether VAT is expected on discounted taxable subtotal or pre-discount subtotal.

### Performance

- None identified. The planned change adds boolean fields to existing Salesforce projections and replaces in-memory total calculations.

### Breaking Changes

- Any code or frontend assumptions that `hiddenVat` implies every line is taxable will become invalid once mixed taxable and exempt carts are supported.
- Frontend consumers need to switch VAT-row visibility decisions to the new `isVatExempt` flags instead of inferring from VAT amount presence alone.
