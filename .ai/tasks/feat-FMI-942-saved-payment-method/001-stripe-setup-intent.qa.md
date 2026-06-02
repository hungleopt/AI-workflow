# 001 - QA - Stripe SetupIntent
Task: .ai/tasks/feat-FMI-942-saved-payment-method/001-stripe-setup-intent.md
Generated: 2026-05-20
Risk: high — payment/billing logic, PCI compliance surface

## Affected features
- Stripe payment integration
- New customer checkout flow (card saving)
- Stripe Customer creation

## Test scenarios

### Happy path
- [ ] SetupIntent is created successfully with a valid client secret
- [ ] Stripe Customer is created with correct email, name, and `customer_type = 'New customer'` metadata
- [ ] PaymentMethod can be retrieved from a confirmed SetupIntent

### Edge cases
- [ ] Stripe API timeout — returns null gracefully without throwing unhandled exception
- [ ] Invalid email format — handles Stripe validation error
- [ ] Customer already exists for email — does not create duplicate (or handles gracefully)

### Regression checks
- [ ] Existing `CreatePaymentIntent` for logged-in users still works unchanged
- [ ] Existing Stripe customer lookup by SalesforceID metadata still works

## Not affected (skip these)
- Direct Debit flow (separate task)
- Invoice payment flow
- Frontend checkout UI (separate task)
- Salesforce order creation

## Status
- [ ] Executor verified: QA IMPACT matches actual changes made
- [ ] Tester sign-off
