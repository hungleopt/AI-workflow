# Memory — feat-FMI-942-saved-payment-method

Created: 2026-05-20
Expiry: 2026-06-19 (30 days)
Branch: feat/FMI-942-saved-payment-method

## Summary

Adding a saved payment method step (Direct Debit or Saved Card) for new customers during checkout. DD details stored in Salesforce, card details tokenized via Stripe SetupIntent.

## Key Decisions

- Card data MUST use Stripe Elements (SetupIntent + PaymentElement) — raw card data never touches server (PCI DSS compliance)
- Direct Debit data is sent server-side but validated strictly (sort code 6 digits, account number 8 digits)
- New Stripe Customer created for anonymous users with `metadata['customer_type'] = 'New customer'`
- On repeat orders, metadata updated to `'Existing customer'`
- `enqix_payment_method` values: `"Direct Debit"` or `"Credit Card"` (matching existing patterns in RecurringOrderService)
- Stripe Customer ID stored on both Account (`Stripe_ID__c`) and Contact (`Stripe_Customer_ID__c`)
- Salesforce Parent Account ID stored in Stripe Customer metadata

## Open Questions (blocking implementation)

1. PCI: Confirm "capture card details" = Stripe Elements tokenization (not server-side storage)
2. BACS mandate: Is legal disclaimer text required for DD authorization checkbox?
3. New customer definition: `!IsLoggedIn` or `!HasParentAccountId`?
4. Step 2 card reuse: off-session charge or re-confirmation required?
5. Stripe Customer for anonymous: create at SetupIntent time or order completion?

## Architecture Notes

- Existing `StripeService.CreatePaymentIntent` creates Stripe Customer only for logged-in users (file: `FirstMile.Services/Stripe/StripeService.cs:38`)
- Existing `AccountService.UpdateDirectDebitAsync` handles DD field update (file: `FirstMile.Salesforce/Services/AccountService.cs:35`)
- `ReorderContext.IsDeferredPayment` returns true for "direct debit" (file: `FirstMile.Services/Commerce/Models/Cart.cs:173`)
- Order workflow splits at `!cart.ReorderContext.IsLoggedIn` (file: `FirstMile.Services/OrderService.cs:80`)
- `UpdateAccountModel` already has `Stripe_ID__c` field (file: `FirstMile.Salesforce/Models/UpdateAccountModel.cs:19`)

## Progress

- [x] Review document created: docs/src/tickets/FMI-942-review.md
- [x] Task files generated (6 tasks)
- [ ] Open questions answered by PO
- [ ] Implementation started
