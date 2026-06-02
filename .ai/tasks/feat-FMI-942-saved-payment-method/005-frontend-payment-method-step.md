# 005 - FE - Frontend payment method step

## TASK
Build frontend payment method selection step for new customers in checkout
Classification: EPIC

## PRIOR CONTEXT
- context: .ai/memory/feat-FMI-942-saved-payment-method-context.md
- done so far: StripeService, Cart model, API endpoints, order workflow

## PATTERN
none

## CONTEXT
- files:
  - firstmile.widgets/src/blocks/checkout/Checkout.tsx
  - firstmile.widgets/src/blocks/checkout/Payment.tsx
  - firstmile.widgets/src/blocks/checkout/CheckoutDetailsForm.tsx
  - firstmile.widgets/src/blocks/invoices/InvoicePayment.tsx
  - firstmile.web/Features/CheckoutStepOnePage/CheckoutStepOnePageController.cs
  - firstmile.web/Features/CheckoutStepTwoPage/CheckoutStepTwoPageController.cs
- docs:
  - docs/src/tickets/FMI-942-review.md

## GOAL
New customers see a payment method selection step during checkout where they can choose Direct Debit (custom form) or Saved Card (Stripe SetupIntent element). Selected method persists and flows into the payment step.

## STEPS
1. Create `firstmile.widgets/src/blocks/checkout/PaymentMethodSetup.tsx`:
   - Radio button: "Direct Debit" / "Saved Card"
   - If Direct Debit: show `DirectDebitForm` component
   - If Saved Card: fetch SetupIntent from `/api/setupintent/create`, render Stripe PaymentElement with SetupIntent
   - On submit: call `/api/paymentmethod/save` with selected data
2. Create `firstmile.widgets/src/blocks/checkout/DirectDebitForm.tsx`:
   - Inputs: Account Holder Name, Account Number (masked after entry), Sort Code (XX-XX-XX format)
   - Client-side validation: numeric only, correct lengths
   - DD mandate agreement checkbox with legal text
3. Modify `CheckoutDetailsForm.tsx` or `Checkout.tsx`:
   - After customer details step, for anonymous users only, show `PaymentMethodSetup`
   - Condition: `!cart.ReorderContext?.IsLoggedIn` (passed via props from controller)
4. Modify `Payment.tsx`:
   - If `paymentMethodSelection.selectedMethod === 'savedCard'` and `paymentMethodId` exists: use stored PaymentMethod for PaymentIntent confirmation
   - If `paymentMethodSelection.selectedMethod === 'directDebit'`: show "Place order now" for the payment step (first order still paid by card unless DD only)
   - Add "Save card for future" checkbox option when no prior card selection
5. Update `CheckoutStepOnePageController.cs` → `Data()` method:
   - Pass `isNewCustomer` flag and payment method setup configuration to frontend
6. Update `CheckoutStepTwoPageController.cs` → `Data()` method:
   - Pass saved payment method info (type, last4 for display) to Payment component
7. Ensure card form uses Stripe Elements exclusively — never custom input fields for card data
8. Modify `Checkout.tsx` step indicator:
   - For logged-in users: remove numbered steps (Step 1/2/3), show just "Checkout" label
   - For new customers: retain numbered steps (3 visible pages)
   - Use Stripe Card Elements with same UI styling as current checkout payment page

## DONE WHEN
- [ ] New (anonymous) customers see payment method selection step
- [ ] Direct Debit form captures account holder, account number, sort code
- [ ] DD mandate checkbox present with appropriate text
- [ ] Saved Card option renders Stripe PaymentElement with SetupIntent
- [ ] Card data entered only into Stripe Elements — no custom card inputs
- [ ] Logged-in users do NOT see payment method selection step
- [ ] Logged-in users see just "Checkout" without numbered steps (new AC, May 22)
- [ ] Selected payment method persists across checkout steps
- [ ] Step 2 Payment respects saved card selection
- [ ] Compiles without errors
- [ ] No files outside CONTEXT modified
- [ ] No claim made about existing code without citing file:line
- [ ] `005-frontend-payment-method-step.qa.md` generated with accurate affected features and risk level

## DOC UPDATE
none required — frontend components, patterns self-documenting

## COMMIT
feat(ui): add payment method selection step for new customers

- PaymentMethodSetup component with DD form or Stripe SetupIntent
- DirectDebitForm with validation and mandate checkbox
- Payment component respects pre-selected saved card
- Only shown to anonymous/new customers

Breaking: none
Migration: none
