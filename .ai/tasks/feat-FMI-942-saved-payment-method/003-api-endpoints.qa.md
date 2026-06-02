# 003 - QA - API endpoints
Task: .ai/tasks/feat-FMI-942-saved-payment-method/003-api-endpoints.md
Generated: 2026-05-20
Risk: high — payment endpoints, PCI surface, CSRF protection required

## Affected features
- Checkout flow (new customer)
- Stripe payment method saving
- Direct Debit data submission

## Test scenarios

### Happy path
- [ ] POST `/api/setupintent/create` returns 200 with `clientSecret` and `stripeCustomerId`
- [ ] POST `/api/paymentmethod/save` with valid DD details returns 200
- [ ] POST `/api/paymentmethod/save` with valid paymentMethodId returns 200
- [ ] Cart session contains saved payment method after successful save

### Edge cases
- [ ] POST `/api/setupintent/create` when Stripe API is down returns 500 (no sensitive error details)
- [ ] POST `/api/paymentmethod/save` with invalid sort code returns 400 with field-level error
- [ ] POST `/api/paymentmethod/save` with empty paymentMethodId returns 400
- [ ] POST `/api/paymentmethod/save` without CSRF token returns 403
- [ ] POST `/api/setupintent/create` without CSRF token returns 403
- [ ] POST `/api/paymentmethod/save` with SQL injection in account holder name — rejected by validator

### Regression checks
- [ ] Existing POST `/api/paymentintent/create` still works for normal checkout
- [ ] Cart session integrity maintained — other cart data not corrupted

## Not affected (skip these)
- Frontend components (task 005)
- Salesforce integration (task 004)
- Logged-in user checkout (no new endpoints used)
- Invoice payment flow

## Status
- [ ] Executor verified: QA IMPACT matches actual changes made
- [ ] Tester sign-off
