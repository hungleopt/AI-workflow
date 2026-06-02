# 001 - BE - Plumb VAT exemption data from Salesforce into models and cart state

## TASK
Add VAT-exemption fields to Salesforce location/product/additional-charge models, extend SOQL queries to fetch them, and mirror location VAT exemption into the session cart following the existing fuel-surcharge exemption pattern.
Classification: STANDARD

## PATTERN
none

## CONTEXT
- files:
  - FirstMile.Salesforce/Models/SimpleLocationModel.cs
  - FirstMile.Salesforce/Models/FullLocationModel.cs
  - FirstMile.Salesforce/Models/FuelSurgeChargeModel.cs
  - FirstMile.Salesforce/Services/UserService.cs
  - FirstMile.Salesforce/Services/ProductService.cs
  - FirstMile.Services/Commerce/Models/Cart.cs
  - FirstMile.Services/Commerce/Models/LineItem.cs
  - firstmile.web/Api/LocationController.cs
  - firstmile.web/Features/AccountLocationPage/AccountLocationPageController.cs
- docs:
  - docs/src/tickets/FMI-946-review.md

## GOAL
All VAT-exemption metadata (location-level, product-level, additional-charge-level) is available in the application models and the session cart, ready for calculation logic in task 002.

## STEPS
1. Grep to confirm `IsFuelSurchargeExemptLocationC` exists in `FullLocationModel.cs` (~line 94) — use the same pattern for VAT exemption.
2. Add `bool IsVATExemptLocationC` property (mapping Salesforce field `IsVATExemptLocation__c`) to `SimpleLocationModel.cs`.
3. Confirm `FullLocationModel` inherits from `SimpleLocationModel` — no duplicate needed there.
4. In `UserService.cs`, find the SOQL projections for location queries (~line 252, ~line 311) and add `IsVATExemptLocation__c` to the SELECT list.
5. Add `bool IsVATExemptC` property (mapping `IsVATExempt__c`) to the product model used by `ProductService` queries. Grep `SimpleProductModel` or relevant model name to confirm location.
6. In `ProductService.cs`, add `IsVATExempt__c` to all SOQL SELECT lists (methods: `GetProductsByRegionZoneAndProductCodesAsync`, `GetByIdsAsync`, `GetProductsByRegionZoneAndProductCodeListAsync`).
7. Add `bool IsVatExempt` property to `AdditionalChargeItemResponse` in `FuelSurgeChargeModel.cs` (maps `isVatExempt` from the JSON response).
8. Add `bool IsVATExemptLocation` property to `Cart.cs` — same pattern as existing `IsFuelSurchargeExemptLocation` (~line 121).
9. Add `bool IsVatExempt` property to `LineItem.cs` — same pattern as existing `isFuelSurgeCharge` (~line 99).
10. In `LocationController.cs` (~line 244), after the existing `cart.IsFuelSurchargeExemptLocation = location.IsFuelSurchargeExemptLocationC;` line, add: `cart.IsVATExemptLocation = location.IsVATExemptLocationC;`
11. In `AccountLocationPageController.cs` (~line 79), mirror the same assignment as step 10.
12. Stamp `lineItem.IsVatExempt` from product data during pricing (in `PriceService` or wherever line items are constructed from products). Grep for where `isFuelSurgeCharge` is set on line items to find the pattern.
13. For additional-charge line items created in `CheckoutStepTwoPageController` (~lines 51, 61, 73), stamp `IsVatExempt` from the `AdditionalChargeItemResponse.IsVatExempt` value.
14. When location is VAT exempt, override: set ALL line items (products + additional charges) to `IsVatExempt = true` regardless of per-line flags — this is the confirmed location-override rule.

## DONE WHEN
- [ ] `SimpleLocationModel` has `IsVATExemptLocationC` property
- [ ] Location SOQL queries include `IsVATExemptLocation__c`
- [ ] Product model has `IsVATExemptC` property
- [ ] Product SOQL queries include `IsVATExempt__c`
- [ ] `AdditionalChargeItemResponse` has `IsVatExempt` property
- [ ] `Cart` has `IsVATExemptLocation` property
- [ ] `LineItem` has `IsVatExempt` property
- [ ] `LocationController` mirrors VAT exemption to cart (same pattern as fuel surcharge)
- [ ] `AccountLocationPageController` mirrors VAT exemption to cart
- [ ] Line items stamped with VAT exemption from product data
- [ ] Additional-charge line items stamped with VAT exemption from API response
- [ ] Location-level exemption overrides all per-line flags to `true`
- [ ] Compiles without errors
- [ ] Unit tests pass (per testing-policy.md)
- [ ] No files outside CONTEXT modified
- [ ] No claim made about existing code without citing file:line
- [ ] Standards validated: all applicable gates in `.ai/standards/definition-of-done.md` checked
- [ ] `001-plumb-vat-exemption-data.qa.md` generated with accurate affected features and risk level

## DOC UPDATE
- `docs/src/backend/cart-and-order-flow.md` — add note about location/product/additional-charge VAT exemption fields and the location-override rule (if this doc exists; grep first)

## COMMIT
feat(vat): plumb VAT exemption data from Salesforce into models and cart state

- Add IsVATExemptLocation__c to location models and SOQL projections
- Add IsVATExempt__c to product models and SOQL projections
- Add isVatExempt to additional-charge response model
- Mirror location VAT exemption into session cart (same pattern as fuel surcharge)
- Stamp per-line IsVatExempt from product and additional-charge data
- Location-level exemption overrides all per-line flags

Breaking: none
Migration: none
