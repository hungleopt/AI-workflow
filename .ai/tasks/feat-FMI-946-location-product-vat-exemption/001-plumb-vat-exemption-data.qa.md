# 001 - QA - Plumb VAT exemption data
Task: `.ai/tasks/feat-FMI-946-location-product-vat-exemption/001-plumb-vat-exemption-data.md`
Generated: 2026-05-26
Risk: medium - billing metadata now comes from additional Salesforce fields and checkout surcharge payloads.

## Affected features
- Location selection and cart session state
- Product pricing metadata
- Checkout surcharge metadata

## Test scenarios

### Happy path
- [ ] Selecting a VAT-exempt location stores `cart.isVatExempt = true` on the session cart and the flag survives a cart reload.
- [ ] Pricing a VAT-exempt product marks the resulting line item with `item.isVatExempt = true`.
- [ ] Checkout step two maps `isVatExempt` from the additional-charge response onto the generated surcharge line.

### Edge cases
- [ ] Direct-location access and location-home access both stamp the same VAT exemption value onto the cart.
- [ ] Non-exempt locations and products continue to hydrate as taxable.
- [ ] Missing or null exemption fields from Salesforce default safely to taxable behavior unless the ticket explicitly requires another fallback.

### Regression checks
- [ ] Existing VAT amount fields remain in the cart and checkout payloads even when `isVatExempt` flags are added.
- [ ] Fuel-surcharge exemption still populates correctly when selecting a location.
- [ ] Existing location and product queries still return the records required by cart and reorder flows.

## Not affected (skip these)
- Full VAT amount calculation and Stripe totals after metadata is present; that belongs to task 002.
- Salesforce order submission totals unless metadata plumbing breaks the flow.

## Status
- [ ] Executor verified: QA IMPACT matches actual changes made.
- [ ] Tester sign-off