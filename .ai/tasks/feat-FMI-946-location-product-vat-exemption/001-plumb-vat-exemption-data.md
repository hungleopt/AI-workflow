# 001 - BE - Plumb VAT exemption data

## TASK
Expose location, product, and additional-charge VAT exemption metadata to the cart and checkout flow.
Classification: STANDARD

## PATTERN
none

## CONTEXT
- files:
  - `FirstMile.Salesforce/Models/SimpleLocationModel.cs`
  - `FirstMile.Salesforce/Models/FullLocationModel.cs`
  - `FirstMile.Salesforce/Models/SimpleProductModel.cs`
  - `FirstMile.Salesforce/Models/FullProductModel.cs`
  - `FirstMile.Salesforce/Models/FuelSurgeChargeModel.cs`
  - `FirstMile.Salesforce/Services/UserService.cs`
  - `FirstMile.Salesforce/Services/ProductService.cs`
  - `FirstMile.Services/Commerce/Models/Cart.cs`
  - `FirstMile.Services/Commerce/Models/LineItem.cs`
  - `FirstMile.Services/Commerce/PriceService.cs`
  - `firstmile.web/Api/LocationController.cs`
  - `firstmile.web/Features/AccountLocationPage/AccountLocationPageController.cs`
  - `firstmile.web/Features/CheckoutStepTwoPage/CheckoutStepTwoPageController.cs`
- docs:
  - `docs/src/tickets/FMI-946-review.md`

## GOAL
Selected locations, priced products, and additional-charge lines all carry explicit `isVatExempt` metadata before any cart total calculation or UI rendering relies on them.

## STEPS
1. Grep the current location and product SOQL projections to confirm the exemption fields are not already selected.
2. Add `IsVATExemptLocation__c` to the location models and extend the `UserService` location queries that feed cached live-location state.
3. Add `IsVATExempt__c` to the product models and extend the `ProductService` queries used by anonymous pricing, reorder pricing, and product lookup by id.
4. Add `isVatExempt` to `AdditionalChargeItemResponse` and carry that value onto the synthetic surcharge line item created in checkout step two.
5. Add cart-level and line-level `isVatExempt` fields so location and product exemption state can survive in session state.
6. Mirror location VAT exemption into the cart wherever location selection currently mirrors fuel-surcharge exemption.
7. Update `PriceService` so line items are marked VAT exempt from either the selected location or the product record before totals are calculated.
8. Add or extend unit tests for location-model hydration, product-model hydration, and price-service propagation of VAT-exempt metadata.

## DONE WHEN
- [ ] `IsVATExemptLocation__c` is mapped on the location models used by cart and location selection.
- [ ] `UserService` live-location queries select the location VAT exemption field.
- [ ] `IsVATExempt__c` is mapped on the product models used by pricing.
- [ ] `ProductService` pricing queries select the product VAT exemption field.
- [ ] `AdditionalChargeItemResponse` can deserialize `isVatExempt`.
- [ ] Session cart state stores `cart.isVatExempt` alongside the existing location-based surcharge exemption pattern.
- [ ] `LineItem` instances carry `item.isVatExempt` metadata before cart totals are computed.
- [ ] Compiles without errors.
- [ ] Unit tests pass (per testing-policy.md).
- [ ] No files outside CONTEXT modified.
- [ ] All PATTERN steps completed or marked N/A.
- [ ] No claim made about existing code without citing file:line.
- [ ] Skill files loaded: N/A - no module skill files exist under `.ai/skills/` for the touched modules.
- [ ] If interface changed: skill file for affected module updated or rewritten, or N/A documented.
- [ ] Standards validated: all applicable gates in `.ai/standards/definition-of-done.md` checked.
- [ ] `001-plumb-vat-exemption-data.qa.md` generated with accurate affected features and risk level.
- [ ] Executor verified: QA IMPACT matches actual changes made.

## DOC UPDATE
none required until VAT calculation behavior changes in the follow-up task.

## COMMIT
feat(cart-tax): plumb VAT exemption metadata

- map location, product, and surcharge VAT exemption flags into `cart.isVatExempt` and `item.isVatExempt` state
- preserve per-line VAT-exempt metadata for follow-up pricing changes and frontend VAT-row hiding

Breaking: none
Migration: none