# 006 - BE - Returning customer payment

## TASK
Display saved payment method for returning logged-in customers in checkout
Classification: EPIC

## PRIOR CONTEXT
- context: .ai/memory/feat-FMI-942-saved-payment-method-context.md
- done so far: StripeService, Cart model, API endpoints, order workflow, frontend payment method step

## PATTERN
none

## CONTEXT
- files:
  - firstmile.web/Features/CheckoutStepTwoPage/CheckoutStepTwoPageController.cs
  - firstmile.widgets/src/blocks/checkout/Payment.tsx
  - FirstMile.Services/Stripe/IStripeService.cs
  - FirstMile.Services/Stripe/StripeService.cs
  - FirstMile.Services/Commerce/CartService.cs
  - FirstMile.Services/Commerce/Models/Cart.cs
- docs:
  - docs/src/tickets/FMI-942-review.md

## GOAL
Logged-in customers with a saved card see their card pre-populated in the payment step. Direct Debit customers see "Place order now" button. Customer type is updated to "Existing customer" on repeat orders.

## STEPS
1. Add method to `IStripeService` / `StripeService`: `GetSavedPaymentMethods(string stripeCustomerId)` — retrieves list of PaymentMethods for a customer
2. In `CheckoutStepTwoPageController.Data()`:
   - If logged-in user has `Stripe_ID__c` on account: fetch saved payment methods from Stripe
   - Pass saved card info (last4, brand, expiry) to frontend Payment component props
   - If account `EnqixPaymentMethodC == "Direct Debit"`: signal frontend to show "Place order now" (existing `IsDeferredPayment` already handles this — verify)
3. In `Payment.tsx`:
   - If saved card info provided: show card details (brand icon, last4, expiry) pre-populated
   - User can still edit/change card via Stripe PaymentElement
4. In `CheckoutStepTwoPageController.Confirm()`:
   - If using saved payment method: create PaymentIntent with `payment_method` and `customer` set, confirm off-session or require confirmation
   - Call `stripeService.UpdateCustomerMetadata(customerId, { "customer_type": "Existing customer" })` after successful payment
5. Verify: `ReorderContext.IsDeferredPayment` already returns true for "Direct Debit" — confirm "Place order now" is already shown (may be no change needed for DD display)

## DONE WHEN
- [ ] Logged-in user with saved card sees card info (last4, brand) on payment page
- [ ] Logged-in user can use saved card for payment
- [ ] Logged-in user with Direct Debit sees "Place order now" button
- [ ] Repeat order updates Stripe Customer metadata to "Existing customer"
- [ ] Compiles without errors
- [ ] Unit tests pass (per testing-policy.md)
- [ ] No files outside CONTEXT modified
- [ ] No claim made about existing code without citing file:line
- [ ] `006-returning-customer-payment.qa.md` generated with accurate affected features and risk level

## DOC UPDATE
docs/src/backend/cart-and-order-flow.md — add section on saved payment method display for returning customers

## COMMIT
feat(checkout): display saved payment method for returning customers

- fetch and display saved card details (last4, brand) for logged-in users
- support off-session payment with saved PaymentMethod
- update Stripe customer_type to "Existing customer" on repeat order
- DD customers continue to see deferred payment flow

Breaking: none
Migration: none
