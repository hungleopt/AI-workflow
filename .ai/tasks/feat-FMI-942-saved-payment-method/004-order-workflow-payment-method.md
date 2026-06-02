# 004 - BE - Order workflow payment method

## TASK
Extend OrderCompleteWorkflow to persist payment method selection to Salesforce and Stripe
Classification: EPIC

## PRIOR CONTEXT
- context: .ai/memory/feat-FMI-942-saved-payment-method-context.md
- done so far: StripeService, Cart model, API endpoints

## PATTERN
none

## CONTEXT
- files:
  - FirstMile.Services/OrderService.cs
  - FirstMile.Services/IOrderService.cs
  - FirstMile.Salesforce/Interfaces/IAccountService.cs
  - FirstMile.Salesforce/Services/AccountService.cs
  - FirstMile.Salesforce/Models/UpdateAccountDirectDebitModel.cs
  - FirstMile.Services/Stripe/IStripeService.cs
  - FirstMile.Services/Commerce/Models/Cart.cs
- docs:
  - docs/src/tickets/FMI-942-review.md
  - docs/src/backend/cart-and-order-flow.md

## GOAL
When a new customer completes checkout with a payment method selection, the order workflow persists: (1) DD details to Salesforce parent account, OR (2) Stripe Customer ID to Salesforce **Parent Account** (`stripe_customer_ID` field), and updates `enqix_payment_method` accordingly. Stripe Customer ID is account-scoped, not user-scoped. For logged-in repeat orders, customer_type metadata is updated to "Existing customer" in Stripe.

## STEPS
1. grep `OrderCompleteWorkflow` in OrderService.cs to locate the new-user branch (~line 80-150)
2. After account creation in the `!cart.ReorderContext.IsLoggedIn` branch, add payment method handling:
   - If `cart.PaymentMethodSelection?.SelectedMethod == DirectDebit`:
     - Call `accountService.UpdateDirectDebitAsync(...)` with DD fields from cart + `EnqixPaymentMethodC = "Direct Debit"`, `DirectDebitAuthorisedC = true`, `DirectDebitAuthorisedDateC = TODAY`
   - If `cart.PaymentMethodSelection?.SelectedMethod == SavedCard`:
     - Call `accountService.UpdateAccountAsync(new UpdateAccountModel(cart.PaymentMethodSelection.StripeCustomerId), accountId)` (stores `stripe_customer_ID` on Parent Account)
     - Confirm: set `enqix_payment_method = "Credit Card"` on account
     - Send Salesforce Parent Account ID back to Stripe Customer metadata
3. For logged-in user branch (existing flow): after payment, call `stripeService.UpdateCustomerMetadata(customerId, { "customer_type": "Existing customer" })` if Stripe Customer exists
4. grep `UpdateAccountModel` to check current fields — add `StripeCustomerIdC` mapped to `stripe_customer_ID` if missing
5. Ensure DD fields from cart are sanitized before sending to Salesforce (already validated by DirectDebitValidator at API layer, but defense-in-depth)

## DONE WHEN
- [ ] New customer + Direct Debit → parent account DD fields updated in Salesforce
- [ ] New customer + Direct Debit → `enqix_payment_method = "Direct Debit"` on account
- [ ] New customer + Saved Card → `stripe_customer_ID` updated on Parent Account
- [ ] New customer + Saved Card → `enqix_payment_method = "Credit Card"` on account
- [ ] Logged-in repeat order → Stripe Customer metadata updated to "Existing customer"
- [ ] Salesforce Parent Account ID sent back to Stripe Customer metadata
- [ ] Compiles without errors
- [ ] Unit tests pass (per testing-policy.md)
- [ ] No files outside CONTEXT modified
- [ ] No claim made about existing code without citing file:line
- [ ] `004-order-workflow-payment-method.qa.md` generated with accurate affected features and risk level

## DOC UPDATE
docs/src/backend/cart-and-order-flow.md — update OrderCompleteWorkflow section with payment method persistence logic

## COMMIT
feat(order): persist payment method to Salesforce and Stripe on order completion

- new customer + DD: updates parent account DD fields and enqix_payment_method
- new customer + saved card: stores Stripe Customer ID on Contact and Account
- repeat order: updates Stripe Customer metadata to "Existing customer"
- sends Salesforce Parent Account ID to Stripe Customer record

Breaking: none
Migration: none
