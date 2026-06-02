# 002 - BE - Cart model and DD validator

## TASK
Add PaymentMethodSelection model to Cart and create DirectDebit validator
Classification: EPIC

## PRIOR CONTEXT
- context: .ai/memory/feat-FMI-942-saved-payment-method-context.md
- done so far: StripeService SetupIntent methods added

## PATTERN
none

## CONTEXT
- files:
  - FirstMile.Services/Commerce/Models/Cart.cs
  - FirstMile.Services/Helpers/
  - FirstMile.Salesforce/Models/UpdateAccountDirectDebitModel.cs
- docs:
  - docs/src/tickets/FMI-942-review.md

## GOAL
Cart model supports payment method selection state (DD details or saved card reference). DirectDebit input is validated server-side with strict numeric/format rules.

## STEPS
1. Create `FirstMile.Services/Commerce/Models/PaymentMethodSelection.cs` with:
   - `PaymentMethodType` enum: `None`, `DirectDebit`, `SavedCard`
   - `SelectedMethod` property
   - DD fields: `AccountHolderName`, `AccountNumber`, `SortCode`
   - Stripe fields: `SetupIntentId`, `PaymentMethodId`, `StripeCustomerId`
2. Add `PaymentMethodSelection? PaymentMethodSelection` property to `Cart` class (Cart.cs line ~125 area)
3. Create `FirstMile.Services/Helpers/DirectDebitValidator.cs`:
   - `ValidateSortCode(string)` — must be exactly 6 digits (strip hyphens/spaces first)
   - `ValidateAccountNumber(string)` — must be exactly 8 digits
   - `ValidateAccountHolderName(string)` — non-empty, max 18 chars, no special injection chars
   - Return validation result with error messages
4. Ensure DD fields are never logged — add `[JsonIgnore]` or custom serialization exclusion for sensitive fields in log contexts

## DONE WHEN
- [ ] `PaymentMethodSelection` model created with enum and fields
- [ ] `Cart` class includes `PaymentMethodSelection` property
- [ ] `DirectDebitValidator` validates sort code (6 digits), account number (8 digits), account holder name
- [ ] No bank account numbers appear in log output
- [ ] Compiles without errors
- [ ] Unit tests pass (per testing-policy.md)
- [ ] No files outside CONTEXT modified
- [ ] No claim made about existing code without citing file:line
- [ ] `002-cart-model-dd-validator.qa.md` generated with accurate affected features and risk level

## DOC UPDATE
none required — internal models only

## COMMIT
feat(models): add PaymentMethodSelection to Cart and DirectDebit validator

- new PaymentMethodSelection model with DD and saved card fields
- DirectDebitValidator enforces UK sort code (6 digits) and account number (8 digits) format
- sensitive DD fields excluded from logging

Breaking: none
Migration: none
