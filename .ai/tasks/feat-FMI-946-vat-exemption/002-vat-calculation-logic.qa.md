# QA — Update VAT calculations to use per-line exemption and taxable-subtotal math
Task: .ai/tasks/feat-FMI-946-vat-exemption/002-vat-calculation-logic.md
Generated: 2026-05-28
Risk: high — payment amounts and VAT display changed; incorrect math causes overcharge/undercharge and potential compliance issues

## Affected features
- Cart page totals (subtotal, VAT, total)
- Checkout step two summary
- Stripe payment intent creation (charged amount)
- Thank-you page order summary
- Saved-basket email VAT display
- Guest checkout flow

## Test scenarios

### Happy path
- [ ] VAT-exempt location → Cart shows VAT = £0.00, total = net subtotal
- [ ] VAT-exempt location → Checkout shows VAT = £0.00
- [ ] VAT-exempt location → Stripe payment intent = net subtotal (no VAT added)
- [ ] VAT-exempt location → Thank-you page shows £0.00 VAT
- [ ] Non-exempt location, all taxable products → VAT calculated at standard rate on full subtotal
- [ ] Non-exempt location, mixed products → VAT calculated only on taxable product lines
- [ ] Non-exempt location, VAT-exempt additional charge → charge does not contribute to VAT total
- [ ] Guest checkout with exempt product → product-level exemption applies, VAT = 0 for that line only

### Edge cases
- [ ] Cart with only VAT-exempt products on non-exempt location → VAT = £0.00
- [ ] Cart with only additional charges (no regular products) and exempt charges → VAT = £0.00
- [ ] Promo discount applied to taxable product → VAT calculated on discounted price
- [ ] Empty cart (0 items) → no crash, VAT = £0.00
- [ ] Location switch from exempt to non-exempt mid-session → VAT recalculated immediately
- [ ] Location switch from non-exempt to exempt mid-session → VAT drops to £0.00
- [ ] Very large order (many lines, mixed exemption) → totals mathematically correct
- [ ] Rounding: verify VAT is rounded consistently (e.g., £0.003 scenario)

### Regression checks
- [ ] Standard fully-taxable order → exact same totals as before the change
- [ ] Cart page JSON response still contains all existing fields (no breaking removal)
- [ ] Payment intent amount matches what customer sees in cart total
- [ ] Checkout step two → payment intent amount matches checkout total
- [ ] Thank-you page renders correctly for standard taxable orders
- [ ] Saved-basket email renders correctly for standard taxable orders
- [ ] Fuel-surcharge exempt location → surcharge still suppressed correctly
- [ ] Order again flow → repriced lines carry correct VAT exemption flags

## Not affected (skip these)
- Order submission to Salesforce (net prices unchanged)
- Account/location creation
- User registration and login
- Product catalogue browsing (non-pricing pages)
- CMS content pages
- Frontend JS/CSS (backend payload only — frontend task if needed would be separate)

## Status
- [ ] Executor verified: QA IMPACT matches actual changes made
- [ ] Tester sign-off
