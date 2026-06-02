# 003 - BE - API endpoints

## TASK
Create API endpoints for SetupIntent creation and payment method selection persistence
Classification: EPIC

## PRIOR CONTEXT
- context: .ai/memory/feat-FMI-942-saved-payment-method-context.md
- done so far: StripeService SetupIntent methods, Cart model with PaymentMethodSelection, DirectDebitValidator

## PATTERN
none

## CONTEXT
- files:
  - firstmile.web/Api/PaymentIntentController.cs
  - firstmile.web/Api/BaseApiController.cs
  - FirstMile.Services/Stripe/IStripeService.cs
  - FirstMile.Services/Commerce/ICartService.cs
  - FirstMile.Services/Commerce/Models/PaymentMethodSelection.cs
  - FirstMile.Services/Helpers/DirectDebitValidator.cs
- docs:
  - docs/src/tickets/FMI-942-review.md

## GOAL
New API endpoints allow the frontend to: (1) create a Stripe SetupIntent for card saving, (2) save the selected payment method (DD or saved card) to the cart session.

## STEPS
1. grep `BaseApiController` to confirm base class pattern and any CSRF/auth middleware
2. Create `firstmile.web/Api/SetupIntentController.cs`:
   - POST endpoint `/api/setupintent/create`
   - Accept: email, firstName, lastName from cart (or read from session cart)
   - Call `_stripeService.CreateSetupIntent(...)` 
   - Return: `{ clientSecret, stripeCustomerId }`
   - Error: return 500 with generic message (no Stripe error details exposed)
3. Create `firstmile.web/Api/PaymentMethodController.cs`:
   - POST endpoint `/api/paymentmethod/save`
   - Accept: `PaymentMethodSelectionRequest` (method type, DD details OR paymentMethodId + setupIntentId)
   - If DirectDebit: validate via `DirectDebitValidator`, return 400 on failure
   - If SavedCard: validate paymentMethodId is non-empty
   - Save to cart session via `cartService`
   - Return: 200 OK
4. Ensure both endpoints validate anti-forgery tokens (follow existing pattern from other API controllers)
5. Ensure no sensitive data (account numbers, card tokens) appears in response error messages or logs

## DONE WHEN
- [ ] POST `/api/setupintent/create` returns Stripe SetupIntent client secret
- [ ] POST `/api/paymentmethod/save` validates and persists DD details to cart
- [ ] POST `/api/paymentmethod/save` validates and persists saved card reference to cart
- [ ] Invalid DD input returns 400 with specific field error messages
- [ ] No bank account numbers or card details in logs or error responses
- [ ] Anti-forgery / CSRF protection applied
- [ ] Compiles without errors
- [ ] Unit tests pass (per testing-policy.md)
- [ ] No files outside CONTEXT modified
- [ ] No claim made about existing code without citing file:line
- [ ] `003-api-endpoints.qa.md` generated with accurate affected features and risk level

## DOC UPDATE
docs/src/backend/cart-and-order-flow.md — add SetupIntent endpoint documentation

## COMMIT
feat(api): add SetupIntent and PaymentMethod selection endpoints

- POST /api/setupintent/create — creates Stripe SetupIntent for card saving
- POST /api/paymentmethod/save — persists DD or saved card selection to cart
- DirectDebit validation on server side, no raw card data server-side
- CSRF protection applied

Breaking: none
Migration: none
