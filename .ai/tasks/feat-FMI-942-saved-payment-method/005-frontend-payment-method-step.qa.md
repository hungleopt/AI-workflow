# 005 - QA - Frontend payment method step
Task: .ai/tasks/feat-FMI-942-saved-payment-method/005-frontend-payment-method-step.md
Generated: 2026-05-20
Risk: high — PCI compliance (card UI), user-facing payment flow, complex state management

## Affected features
- New customer checkout flow
- Payment page (Step 2)
- Card saving flow
- Direct Debit data entry

## Test scenarios

### Happy path
- [ ] Anonymous user sees payment method selection after customer details
- [ ] Selecting "Direct Debit" shows DD form fields
- [ ] Selecting "Saved Card" renders Stripe PaymentElement
- [ ] Submitting valid DD details saves to cart and proceeds
- [ ] Completing Stripe SetupIntent saves PaymentMethod and proceeds
- [ ] Step 2 shows saved card info when card was saved in Step 1
- [ ] "Place order now" button shown when Direct Debit selected

### Edge cases
- [ ] Switching between DD and Saved Card clears previous selection data
- [ ] Browser back button from Step 2 — payment method selection preserved
- [ ] Stripe SetupIntent failure — error message shown, can retry
- [ ] DD form submit with invalid data — inline errors shown, cannot proceed
- [ ] Network error during `/api/paymentmethod/save` — user-friendly error
- [ ] Stripe Elements not loading (JS blocked) — graceful fallback or clear error

### Regression checks
- [ ] Logged-in user checkout — payment method step NOT shown
- [ ] Logged-in user with existing payment method — normal flow preserved
- [ ] Anonymous checkout without selecting payment method (if allowed) — behaves as current flow
- [ ] Invoice payment option for eligible logged-in accounts still works
- [ ] Marketing mail opt-in still works on Step 2
- [ ] PO number input still works on Step 2

## Not affected (skip these)
- Invoice listing/payment page
- Recurring order setup
- Cart add/remove/update
- Price calculation display
- Header/footer/navigation

## Status
- [ ] Executor verified: QA IMPACT matches actual changes made
- [ ] Tester sign-off
