# 002 - BE - EUR invoice and credit note PDF

## CONTEXT

- Ticket: FMI-945
- Branch: `feat/FMI-945-eur-invoicing`
- Review: `docs/src/tickets/FMI-945-review.md`
- Depends on: `001-invoice-models-and-statement`

## FILES IN SCOPE

- `firstmile.web/Features/AccountInvoicesPage/AccountInvoicesPageController.cs`
- `firstmile.web.Tests/Features/AccountInvoicesPage/AccountInvoicesPageControllerTests.cs`

## TASK

1. **Download action** (~line 270-340): Pass `invoice.CurrencyIsoCode` to template methods.

2. **TemplatePerOrder** method — Replace all hardcoded `£` with dynamic currency symbol:
   - Line 405: `£{line.UnitPrice}` → `{currencySymbol}{line.UnitPrice}`
   - Line 406: `£{line.Amount}` → `{currencySymbol}{line.Amount}`
   - Line 462: `£{invoice.AmountExclVAT}` → `{currencySymbol}{invoice.AmountExclVAT}`
   - Line 466: `£{invoice.VAT}` → `{currencySymbol}{invoice.VAT}`
   - Line 470: `£{invoice.AmountIncludeVAT}` → `{currencySymbol}{invoice.AmountIncludeVAT}`
   - Add method parameter: `string? currencyCode = null`
   - Compute symbol at start: `var currencySymbol = MoneyHelper.GetCurrencySymbol(currencyCode);`

3. **TemplatePerOrder — Bank details section**: Conditionally show different bank details based on currency:
   - GBP (default): `Sort Code 20-05-75, Account No. 33305252`
   - EUR: Show IBAN/BIC details (exact values TBD from business — use placeholder until confirmed)
   - Consider: If `LegalEntity` already has the correct bank details for each entity, use those instead of hardcoded values.

4. **TemplatePerOrder — VAT label**: If currency is EUR, potentially change `VAT @20%` label. (Depends on answer to open question about Irish VAT rate — for now, use the rate from Salesforce data.)

5. **TemplateMonthlyAsync** method — Same pattern:
   - Line 540: `£{line.UnitPrice}` → `{currencySymbol}{line.UnitPrice}`
   - Line 541: `£{line.Amount}` → `{currencySymbol}{line.Amount}`
   - Line 563: `Subtotal: £{locationSubtotal:#.##}` → `Subtotal: {currencySymbol}{locationSubtotal:#.##}`
   - Line 578: `Subtotal: £{subtotal:#.##}` → `Subtotal: {currencySymbol}{subtotal:#.##}`
   - Line 625: `£{invoice.AmountExclVAT}` → `{currencySymbol}{invoice.AmountExclVAT}`
   - Line 629: `£{invoice.VAT}` → `{currencySymbol}{invoice.VAT}`
   - Line 633: `£{invoice.AmountIncludeVAT}` → `{currencySymbol}{invoice.AmountIncludeVAT}`

6. **IndexAsync** (~line 69): `outstandingAmount = "£" + amount` → use `MoneyHelper.GetCurrencySymbol(...)` (note: outstanding is for the whole account — may need to determine primary currency from account).

7. **Credit note PDF template**: Apply the same EUR currency symbol, bank details, and VAT label changes to the credit note template rendering method. Credit notes follow the same layout — use `CurrencyIsoCode` from the credit note's invoice header to determine EUR vs GBP. Reference the attached `Euro Credit Note - Final V.pdf` for exact differences (highlighted in yellow).

8. Unit tests:
   - Test EUR invoice PDF contains `€` symbols
   - Test EUR credit note PDF contains `€` symbols and "CREDIT NOTE" header
   - Test GBP invoice/credit note PDFs still contain `£` symbols
   - Test null currency defaults to `£`

## DONE WHEN

- [ ] `TemplatePerOrder` uses dynamic currency symbol from invoice `CurrencyIsoCode`
- [ ] `TemplateMonthlyAsync` uses dynamic currency symbol
- [ ] EUR invoices show Euro bank details (IBAN/BIC from attached template)
- [ ] EUR credit notes show Euro bank details and `€` symbol
- [ ] GBP invoices and credit notes unchanged
- [ ] Outstanding amount on index page uses account currency
- [ ] VAT registration number remains unchanged for EUR (confirmed by PO)
- [ ] Unit tests verify EUR and GBP PDF content for both invoices and credit notes
- [ ] Compiles without errors
- [ ] Unit tests pass (per testing-policy.md)
- [ ] No files outside CONTEXT modified
- [ ] No claim made about existing code without citing file:line
- [ ] No files outside CONTEXT modified
- [ ] No claim made about existing code without citing file:line
