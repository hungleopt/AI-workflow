# 001 - BE - Invoice models and statement

## CONTEXT

- Ticket: FMI-945
- Branch: `feat/FMI-945-eur-invoicing`
- Review: `docs/src/tickets/FMI-945-review.md`
- Depends on: FMI-944 (currency infrastructure — `MoneyHelper` overloads)

## FILES IN SCOPE

- `FirstMile.Services/Salesforce/Invoices/GetInvoiceByIdResponse.cs`
- `FirstMile.Salesforce/Models/InvoiceModel.cs`
- `FirstMile.Salesforce/Services/AccountStatementService.cs`
- `FirstMile.Salesforce.Tests/Services/AccountStatementServiceTests.cs`
- `FirstMile.Salesforce.Tests/Helpers/MoneyHelperTests.cs`

## TASK

1. **GetInvoiceByIdResponse** — Add `CurrencyIsoCode` property:
   ```csharp
   [JsonPropertyName("CurrencyIsoCode")]
   public string? CurrencyIsoCode { get; set; }
   ```

2. **InvoiceModel** — Add `CurrencyIsoCode` property:
   ```csharp
   [JsonPropertyName("CurrencyIsoCode")]
   public string? CurrencyIsoCode { get; set; }
   ```

3. **AccountStatementService** — Update hardcoded `£` in amount formatting:
   - Line 170: `Amount = $"£-{x.PaymentAmountC}"` → Keep as `£` (bank statements are always GBP for now, per business logic)
   - Line 206: `Amount = $"£-{...}"` → Keep as `£` (bank receipts)
   - Line 220: `Amount = $"£{x.AmountInclVatFc}"` → Use `MoneyHelper.GetCurrencySymbol(x.CurrencyIsoCode)` + amount
   - Line 221: `OutstandingAmount = $"£{x.AmountOutstandingC}"` → Use `MoneyHelper.GetCurrencySymbol(x.CurrencyIsoCode)` + amount

4. **Salesforce SOQL** — Ensure the invoice query in `AccountStatementService` includes `CurrencyIsoCode` in its SELECT statement. Check the existing query and add the field.

5. Unit tests:
   - `AccountStatementServiceTests`: verify EUR invoice formats with `€` symbol
   - `MoneyHelperTests`: verify `GetCurrencySymbol` returns correct symbols

## DONE WHEN

- [ ] `GetInvoiceByIdResponse` has `CurrencyIsoCode` property
- [ ] `InvoiceModel` has `CurrencyIsoCode` property
- [ ] `AccountStatementService` formats invoice amounts with dynamic currency symbol
- [ ] Salesforce query includes `CurrencyIsoCode` field
- [ ] Unit tests pass for EUR and GBP formatting in statements
- [ ] Compiles without errors
- [ ] Unit tests pass (per testing-policy.md)
- [ ] No files outside CONTEXT modified
- [ ] No claim made about existing code without citing file:line
