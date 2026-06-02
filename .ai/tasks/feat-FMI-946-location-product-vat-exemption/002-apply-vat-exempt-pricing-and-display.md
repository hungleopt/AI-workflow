# 002 - BE - Apply VAT exempt pricing and display

## TASK
Apply VAT exemption metadata to cart, checkout, payment, thank-you, and saved-basket totals and display behavior.
Classification: STANDARD

## PRIOR CONTEXT
- review: `docs/src/tickets/FMI-946-review.md`
- depends on: `.ai/tasks/feat-FMI-946-location-product-vat-exemption/001-plumb-vat-exemption-data.md`

## PATTERN
none

## CONTEXT
- files:
  - `FirstMile.Services/Commerce/Models/Cart.cs`
  - `FirstMile.Services/Commerce/Models/LineItem.cs`
  - `FirstMile.Services/Commerce/PriceService.cs`
  - `FirstMile.Services/Commerce/CartService.cs`
  - `firstmile.web/Api/CartController.cs`
  - `firstmile.web/Api/PaymentIntentController.cs`
  - `firstmile.web/Api/SavedBasketController.cs`
  - `firstmile.web/Features/CartPage/CartPageController.cs`
  - `firstmile.web/Features/CheckoutStepTwoPage/CheckoutStepTwoPageController.cs`
  - `firstmile.web/Features/ThankYouPage/ThankYouPageController.cs`
  - `docs/src/backend/cart-and-order-flow.md`
- docs:
  - `docs/src/tickets/FMI-946-review.md`

## GOAL
VAT is calculated only on taxable lines, VAT-exempt locations suppress VAT correctly across cart and checkout flows, existing VAT amount fields remain available, and the frontend gets `isVatExempt` flags it can use to hide the VAT row.

## STEPS
1. Grep every remaining `hiddenVat`, `cart.vat`, and duplicated VAT formula usage before editing so the full regression surface stays in scope.
2. Replace cart-level VAT total math with helper logic that calculates taxable subtotal, VAT amount, and grand total from per-line taxability.
3. Update `CartService` and `PaymentIntentController` to charge the same VAT amount that cart and checkout display.
4. Update `CartController`, `CartPageController`, and `CheckoutStepTwoPageController` so they keep returning VAT values while also exposing `isVatExempt` flags for fully exempt and mixed-tax carts.
5. Update `ThankYouPageController` and `SavedBasketController` so post-purchase and email summaries do not reconstruct VAT from a single global percentage when the order contains mixed taxability.
6. Implement the confirmed UX: keep VAT fields in payloads, but expose `cart.isVatExempt` and `item.isVatExempt` so the frontend hides the VAT row when the location is exempt.
7. Update the backend flow documentation in `docs/src/backend/cart-and-order-flow.md` to describe the new line-aware VAT behavior and location-based exemption rule.
8. Add or extend unit tests covering mixed taxable/exempt carts, VAT-exempt locations, exempt additional charges, and post-checkout summary behavior.

## DONE WHEN
- [ ] Cart VAT and total values are derived from taxable subtotal rather than all line items.
- [ ] VAT-exempt locations do not add VAT to cart, checkout, payment-intent, thank-you, or saved-basket totals.
- [ ] Mixed taxable/exempt carts apply VAT only to the taxable lines.
- [ ] Checkout surcharge lines respect `isVatExempt` in the final VAT total.
- [ ] `CartService` and `PaymentIntentController` charge the same amount shown in cart and checkout.
- [ ] Cart and checkout payloads keep VAT values while exposing `isVatExempt` flags for frontend VAT-row hiding.
- [ ] `docs/src/backend/cart-and-order-flow.md` documents the new VAT behavior.
- [ ] Compiles without errors.
- [ ] Unit tests pass (per testing-policy.md).
- [ ] No files outside CONTEXT modified.
- [ ] All PATTERN steps completed or marked N/A.
- [ ] No claim made about existing code without citing file:line.
- [ ] Skill files loaded: N/A - no module skill files exist under `.ai/skills/` for the touched modules.
- [ ] If interface changed: skill file for affected module updated or rewritten, or N/A documented.
- [ ] Standards validated: all applicable gates in `.ai/standards/definition-of-done.md` checked.
- [ ] `002-apply-vat-exempt-pricing-and-display.qa.md` generated with accurate affected features and risk level.
- [ ] Executor verified: QA IMPACT matches actual changes made.

## DOC UPDATE
- `docs/src/backend/cart-and-order-flow.md` - replace the current single-rate VAT description with location-aware and line-aware taxable-subtotal behavior.

## COMMIT
feat(cart-tax): apply VAT exemption to totals and summaries

- compute VAT only from taxable cart lines and exempt locations
- align checkout, payment, thank-you, and saved-basket summaries with the same server-side total rules while preserving payload compatibility for the frontend

Breaking: none
Migration: none