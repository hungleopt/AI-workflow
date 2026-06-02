# 006 - QA - Returning customer payment
Task: .ai/tasks/feat-FMI-942-saved-payment-method/006-returning-customer-payment.md
Generated: 2026-05-20
Risk: medium — existing logged-in flow modification, Stripe API dependency

## Affected features
- Logged-in customer payment page
- Saved card display in checkout
- Direct Debit deferred payment display
- Stripe Customer metadata

## Test scenarios

### Happy path
- [ ] Logged-in user with saved card in Stripe sees last4 and brand on payment page
- [ ] Logged-in user can pay with saved card (no re-entry needed)
- [ ] Logged-in user with Direct Debit sees "Place order now" button
- [ ] After repeat order, Stripe Customer metadata shows customer_type="Existing customer"

### Edge cases
- [ ] Logged-in user with Stripe_ID__c but no saved payment methods — shows normal card entry form
- [ ] Logged-in user with expired saved card — Stripe handles gracefully or shows re-entry form
- [ ] Stripe API failure when fetching saved methods — fallback to normal payment form
- [ ] User with both DD and saved card on account — which takes priority? (follow enqix_payment_method)

### Regression checks
- [ ] Anonymous user checkout — this task's changes do not affect anonymous flow
- [ ] Invoice payment for accounts ≥ £1000 with "normal" payment method still works
- [ ] Existing deferred payment flow for "invoice" accounts unchanged
- [ ] Reorder/Book Collection/Add New Service flows unchanged

## Not affected (skip these)
- New customer payment method selection (task 005)
- Direct Debit form entry (task 005)
- Account/Location/Contact creation
- Cart item management
- Price calculation

## Status
- [ ] Executor verified: QA IMPACT matches actual changes made
- [ ] Tester sign-off
