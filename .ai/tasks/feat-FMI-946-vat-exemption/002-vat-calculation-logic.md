# 002 - BE - Update VAT calculations to use per-line exemption and taxable-subtotal math

## TASK
Replace the global single-rate VAT calculation with taxable-subtotal math that respects per-line `IsVatExempt` flags. Update all cart, checkout, payment-intent, thank-you, and saved-basket controllers to produce correct VAT totals for mixed-exemption orders.
Classification: STANDARD

## PRIOR CONTEXT
- done so far: task 001 plumbed VAT exemption data into models and cart state

## PATTERN
none

## CONTEXT
- files:
  - FirstMile.Services/Commerce/Models/Cart.cs
  - FirstMile.Services/Commerce/Models/LineItem.cs
  - FirstMile.Services/Commerce/PriceService.cs
  - FirstMile.Services/Commerce/CartService.cs
  - firstmile.web/Api/CartController.cs
  - firstmile.web/Api/PaymentIntentController.cs
  - firstmile.web/Api/SavedBasketController.cs
  - firstmile.web/Features/CartPage/CartPageController.cs
  - firstmile.web/Features/CheckoutStepTwoPage/CheckoutStepTwoPageController.cs
  - firstmile.web/Features/ThankYouPage/ThankYouPageController.cs
- docs:
  - docs/src/tickets/FMI-946-review.md

## GOAL
VAT is calculated correctly for all order types: fully exempt (location-level), partially exempt (mixed products), fully taxable. Payment intent amounts match cart totals. Frontend receives `isVatExempt` flags to control VAT-row visibility.

## STEPS
1. Grep `Cart.cs` for `vat`, `hiddenVat`, `hiddenTotalPrice`, `notIncludeVatTotalPrice` to map existing VAT calculation flow.
2. Add computed property to `Cart.cs`: `TaxableSubtotal` — sum of `hiddenPrice * quantity` for line items where `IsVatExempt == false`.
3. Add computed property to `Cart.cs`: `ComputedVat` — `MoneyHelper.Tax(TaxableSubtotal, hiddenVat)` — VAT only on taxable lines.
4. If `IsVATExemptLocation == true`, ensure `ComputedVat` returns 0 (location override — all lines exempt).
5. Update `Cart.cs` `vat` property to use `ComputedVat` instead of global `MoneyHelper.Tax(price, hiddenVat)`.
6. Update `Cart.cs` `hiddenTotalPrice` / `totalPrice` to use net subtotal + `ComputedVat`.
7. Grep `CartService.cs` for payment amount calculation (~line 284-285). Update to use taxable-subtotal-based VAT instead of global rate.
8. Grep `PaymentIntentController.cs` (~line 39) for Stripe amount calculation. Align with same taxable-subtotal logic as `CartService`.
9. Grep `CartController.cs` for VAT/total calculations (~lines 102, 115, 176, 229, 237). Update each to use per-line exemption logic.
10. Grep `CartPageController.cs` (~line 84) for cart summary output. Include `cart.IsVATExemptLocation` in the response model for frontend VAT-row hiding.
11. Grep `CheckoutStepTwoPageController.cs` (~line 250) for checkout summary. Include exemption flags in response.
12. Grep `ThankYouPageController.cs` (~lines 24, 28, 50). Stop reconstructing per-line VAT-inclusive prices from single global `hiddenVat` when lines have mixed exemption.
13. Grep `SavedBasketController.cs` (~lines 184-185). Update saved-basket email VAT display for exempt carts.
14. Add `isVatExempt` to cart JSON response payloads (both cart-level and per-line) so frontend can control VAT-row visibility without a breaking payload change.
15. Ensure guest checkout (no location) correctly applies product-level exemption only: `cart.IsVATExemptLocation` remains `false`, individual line items use their `IsVatExempt` flags.

## DONE WHEN
- [ ] `Cart.vat` computed from taxable lines only (not global rate on full subtotal)
- [ ] `Cart.totalPrice` / `hiddenTotalPrice` correctly sums net + computed VAT
- [ ] VAT-exempt location → VAT = £0.00 across cart, checkout, payment, thank-you
- [ ] Mixed taxable/exempt products on non-exempt location → VAT computed on taxable subtotal only
- [ ] Stripe payment intent amount matches cart total (no mismatch between UI and charge)
- [ ] `PaymentIntentController` and `CartService` payment calculations are aligned
- [ ] Cart JSON response includes `isVatExempt` flag (cart-level and per-line)
- [ ] Thank-you page handles mixed-exemption orders correctly
- [ ] Saved-basket email suppresses VAT for exempt carts
- [ ] Guest checkout: product-level exemption works, location-level does not apply
- [ ] Existing fully-taxable orders produce identical results to current behavior (regression)
- [ ] Compiles without errors
- [ ] Unit tests pass (per testing-policy.md)
- [ ] No files outside CONTEXT modified
- [ ] No claim made about existing code without citing file:line
- [ ] Standards validated: all applicable gates in `.ai/standards/definition-of-done.md` checked
- [ ] `002-vat-calculation-logic.qa.md` generated with accurate affected features and risk level

## DOC UPDATE
- `docs/src/backend/cart-and-order-flow.md` — update VAT calculation section to describe taxable-subtotal approach and location-override rule

## COMMIT
feat(vat): replace global VAT rate with per-line taxable-subtotal calculation

- Cart VAT computed from taxable lines only (non-exempt products/charges)
- Location-level exemption → entire order VAT = 0
- Product-level exemption → mixed orders with partial VAT
- Payment intent amount aligned with cart total
- Cart JSON response includes isVatExempt flags for frontend
- Guest checkout: product-level exemption only (no location override)

Breaking: none (isVatExempt added as new fields; existing VAT fields preserved)
Migration: none
