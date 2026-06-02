# 001 - BE - Stripe SetupIntent

## TASK
Add Stripe SetupIntent and anonymous customer creation capabilities to StripeService
Classification: EPIC

## PRIOR CONTEXT
- context: .ai/memory/feat-FMI-942-saved-payment-method-context.md
- done so far: none — first task

## PATTERN
none

## CONTEXT
- files:
  - FirstMile.Services/Stripe/IStripeService.cs
  - FirstMile.Services/Stripe/StripeService.cs
  - FirstMile.Services/Stripe/StripeConfiguration.cs
- docs:
  - docs/src/tickets/FMI-942-review.md

## GOAL
StripeService can create SetupIntents for saving cards, create Stripe Customers for anonymous users with 'New customer' metadata, and retrieve PaymentMethod details from confirmed SetupIntents.

## STEPS
1. grep `IStripeService` to confirm current interface methods
2. Add to `IStripeService`: `Task<SetupIntent?> CreateSetupIntent(string email, string firstName, string lastName)`, `Task<Customer> CreateCustomerForNewUser(string email, string firstName, string lastName)`, `Task<PaymentMethod?> GetPaymentMethodFromSetupIntent(string setupIntentId)`
3. Implement `CreateCustomerForNewUser` in `StripeService`: creates Stripe Customer with metadata `customer_type = "New customer"` and email/name
4. Implement `CreateSetupIntent`: creates customer first (or reuses if email matches), then creates SetupIntent with `customer` and `usage = "off_session"`
5. Implement `GetPaymentMethodFromSetupIntent`: retrieves the SetupIntent and returns its PaymentMethod
6. Add `UpdateCustomerMetadata(string customerId, Dictionary<string, string> metadata)` for later updating customer type to "Existing customer"
7. Ensure no raw card data is handled — all tokenization happens client-side via Stripe Elements

## DONE WHEN
- [ ] `IStripeService` has new method signatures
- [ ] `StripeService` implements SetupIntent creation with customer association
- [ ] Stripe Customer created with `metadata['customer_type'] = 'New customer'`
- [ ] No raw card numbers, CVV, or expiry dates appear in any server-side code
- [ ] Compiles without errors
- [ ] Unit tests pass (per testing-policy.md)
- [ ] No files outside CONTEXT modified
- [ ] No claim made about existing code without citing file:line
- [ ] `001-stripe-setup-intent.qa.md` generated with accurate affected features and risk level

## DOC UPDATE
none required — endpoint not yet exposed

## COMMIT
feat(stripe): add SetupIntent and anonymous customer creation

- add CreateSetupIntent, CreateCustomerForNewUser, GetPaymentMethodFromSetupIntent to StripeService
- Stripe Customer metadata includes customer_type for new/existing distinction
- no raw card data touches server — PCI DSS compliant

Breaking: none
Migration: none
