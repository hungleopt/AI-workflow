# QA — Plumb VAT exemption data from Salesforce into models and cart state
Task: .ai/tasks/feat-FMI-946-vat-exemption/001-plumb-vat-exemption-data.md
Generated: 2026-05-28
Risk: medium — new Salesforce fields feeding VAT logic; incorrect mapping could cause silent tax miscalculation

## Affected features
- Location selection (cart rebinding to location)
- Product pricing and line item construction
- Additional charges (fuel surcharge and other charges)
- Cart state persistence across pages

## Test scenarios

### Happy path
- [ ] Select a VAT-exempt location → cart shows `IsVATExemptLocation = true`
- [ ] Select a non-exempt location → cart shows `IsVATExemptLocation = false`
- [ ] Add a VAT-exempt product to cart → line item shows `IsVatExempt = true`
- [ ] Add a standard taxable product → line item shows `IsVatExempt = false`
- [ ] Proceed to checkout with additional charges → surcharge line items have `IsVatExempt` value from API response
- [ ] VAT-exempt location with taxable products → ALL line items overridden to `IsVatExempt = true`

### Edge cases
- [ ] Location with `IsVATExemptLocation__c = null` in Salesforce → defaults to `false` (not exempt)
- [ ] Product with `IsVATExempt__c = null` in Salesforce → defaults to `false` (not exempt)
- [ ] `ProcessAdditionCharges` returns 0 additional charges → no surcharge lines added, no crash
- [ ] `ProcessAdditionCharges` returns 3+ lines with mixed exemption → each line has independent flag
- [ ] Guest checkout (no location selected) → no location-level exemption, product-level still applies
- [ ] Switch location from exempt to non-exempt → cart `IsVATExemptLocation` updates immediately

### Regression checks
- [ ] Existing fuel-surcharge exemption still works correctly after changes
- [ ] Location selection still loads all existing fields (name, postcode, partnership, etc.)
- [ ] Product queries still return all existing product fields (no fields dropped from SOQL)
- [ ] Cart serialization/deserialization still works with new properties

## Not affected (skip these)
- Order submission to Salesforce (no changes to order creation)
- Payment processing (amounts not changed in this task)
- Email templates and saved basket emails
- Frontend UI rendering (no payload structure change yet)
- Thank-you page calculations

## Status
- [ ] Executor verified: QA IMPACT matches actual changes made
- [ ] Tester sign-off
