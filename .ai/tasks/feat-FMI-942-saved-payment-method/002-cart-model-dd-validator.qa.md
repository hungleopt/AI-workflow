# 002 - QA - Cart model and DD validator
Task: .ai/tasks/feat-FMI-942-saved-payment-method/002-cart-model-dd-validator.md
Generated: 2026-05-20
Risk: medium — financial data validation, sensitive data handling

## Affected features
- Cart session state persistence
- Direct Debit data capture validation
- Checkout flow data model

## Test scenarios

### Happy path
- [ ] Valid sort code "123456" passes validation
- [ ] Valid sort code "12-34-56" passes validation (hyphens stripped)
- [ ] Valid account number "12345678" passes validation
- [ ] Valid account holder name "John Smith" passes validation
- [ ] PaymentMethodSelection serializes/deserializes correctly in cart session

### Edge cases
- [ ] Sort code with letters "12AB56" fails validation
- [ ] Sort code too short "12345" fails validation
- [ ] Sort code too long "1234567" fails validation
- [ ] Account number with letters "1234567A" fails validation
- [ ] Account number too short "1234567" fails validation
- [ ] Empty account holder name fails validation
- [ ] Account holder name with SQL injection characters stripped/rejected
- [ ] Cart without PaymentMethodSelection (null) — existing flows unaffected

### Regression checks
- [ ] Existing cart serialization/deserialization still works (items, ReorderContext, Payment)
- [ ] Existing anonymous cart flow unaffected
- [ ] Existing logged-in cart flow unaffected

## Not affected (skip these)
- Stripe API integration (task 001)
- Frontend UI components (task 005)
- Salesforce updates (task 004)
- Order completion workflow (task 004)

## Status
- [ ] Executor verified: QA IMPACT matches actual changes made
- [ ] Tester sign-off
