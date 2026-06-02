# 002 - QA - Apply VAT exempt pricing and display
Task: `.ai/tasks/feat-FMI-946-location-product-vat-exemption/002-apply-vat-exempt-pricing-and-display.md`
Generated: 2026-05-26
Risk: high - this changes billing totals, Stripe amount calculation, and multiple user-visible summaries.

## Affected features
- Cart summary and cart API payloads
- Checkout step two totals and surcharge handling
- Stripe payment-intent amount calculation
- Thank-you page and saved-basket email summaries
- Frontend VAT-row hiding based on `isVatExempt` flags

## Test scenarios

### Happy path
- [ ] VAT-exempt location: cart, checkout, and payment amount show no VAT charge, while payloads still include VAT fields and set `isVatExempt = true`.
- [ ] Mixed taxable/exempt order: VAT is charged only on the taxable products and additional-charge lines.
- [ ] Fully taxable order: totals remain unchanged from pre-ticket behavior.

### Edge cases
- [ ] Promo or discount applied to a mixed-tax cart still yields the correct taxable VAT amount.
- [ ] Additional-charge line marked `isVatExempt = true` does not increase VAT total.
- [ ] Thank-you page shows the same effective totals that were charged at checkout.
- [ ] Saved-basket email does not show stale VAT for a VAT-exempt cart.

### Regression checks
- [ ] `PaymentIntentController` and `CartService` produce identical totals for the same cart.
- [ ] Fuel-surcharge exemption behavior still works for exempt locations.
- [ ] Standard taxable checkout remains chargeable and completes successfully.
- [ ] Frontend VAT-row logic can rely on `cart.isVatExempt` and `item.isVatExempt` without losing access to VAT amount fields.

## Not affected (skip these)
- Salesforce order posting shape, unless checkout totals or line metadata changes reveal an integration issue.
- Frontend styling changes in `firstmile.ui` unless a widget hardcodes VAT row visibility.

## Status
- [ ] Executor verified: QA IMPACT matches actual changes made.
- [ ] Tester sign-off