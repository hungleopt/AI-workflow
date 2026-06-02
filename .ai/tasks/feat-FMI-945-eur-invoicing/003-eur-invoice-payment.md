# 003 - BE - EUR invoice payment

## CONTEXT

- Ticket: FMI-945
- Branch: `feat/FMI-945-eur-invoicing`
- Review: `docs/src/tickets/FMI-945-review.md`
- Depends on: `001-invoice-models-and-statement`, FMI-944 Stripe currency support

## FILES IN SCOPE

- `firstmile.web/Features/PaymentInvoicePage/PaymentInvoicePageController.cs`
- `firstmile.web/Api/InvoicesController.cs`
- `firstmile.web.Tests/Features/PaymentInvoicePage/PaymentInvoicePageControllerTests.cs`

## TASK

1. **PaymentInvoicePageController.Index** — Fix amount parsing (lines 55, 65):
   - Currently: `subItem[1].Replace("£", "")` — only handles `£`
   - Change to: `subItem[1].Replace("£", "").Replace("€", "")` or better: use regex `Regex.Replace(subItem[1], @"[£€]", "")`
   - Consider: The form data format `"INV001-£12.5"` should ideally not contain currency symbols. If the frontend sends the symbol, handle both. If we can control the format, prefer sending raw numbers.

2. **PaymentInvoicePageController.Index** — Fix submit button label (line 99):
   - Currently: `submitBtnLabel = $"Pay now £{amount}"`
   - Need to determine the currency of the invoices being paid. All selected invoices should be same currency.
   - Get currency from cart or from the invoice data. Use: `$"Pay now {MoneyHelper.GetCurrencySymbol(currencyCode)}{amount}"`

3. **PaymentInvoicePageController.Index** — Fix error message (line 108):
   - Currently: `"Amount must be greater than £0 and no more than £999,999.99"`
   - Make dynamic or use generic: `"Amount must be greater than 0 and no more than 999,999.99"`

4. **PaymentInvoicePageController** — Stripe currency:
   - `CartService.SaveCartWithPaymentIntent` is called — it should already use the cart's `CurrencyCode` after FMI-944. Verify this works for invoice payments too. The cart's `CurrencyCode` should be set from the account.

5. **InvoicesController** — Verify outstanding amount includes currency awareness. The controller gets `outstandingAmount` from the invoices API. Check if it needs a currency symbol prefix.

6. Unit tests:
   - Test EUR invoice payment parses `€12.5` amounts correctly
   - Test submit button shows `"Pay now €20"` for EUR invoices
   - Test GBP payment still works as before

## DONE WHEN

- [ ] Amount parsing handles both `£` and `€` symbols
- [ ] Submit button label uses correct currency symbol
- [ ] Error message is currency-aware or generic
- [ ] Stripe payment for EUR invoices uses `"eur"` currency
- [ ] Unit tests pass for EUR and GBP invoice payments
- [ ] Compiles without errors
- [ ] Unit tests pass (per testing-policy.md)
- [ ] No files outside CONTEXT modified
- [ ] No claim made about existing code without citing file:line
