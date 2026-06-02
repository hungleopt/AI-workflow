# 001 - BE - Add currency infrastructure

## CONTEXT

- Ticket: FMI-944
- Branch: `feat/FMI-944-eur-currency-ordering`
- Review: `docs/src/tickets/FMI-944-review.md`
- Depends on: nothing (first task)

## FILES IN SCOPE

- `FirstMile.Salesforce/Models/FullAccountModel.cs`
- `FirstMile.Salesforce/Helpers/MoneyHelper.cs`
- `FirstMile.Services/Commerce/Models/Cart.cs`
- `FirstMile.Services/Commerce/Models/LineItem.cs`
- `FirstMile.Services/Commerce/CartService.cs`
- `FirstMile.Salesforce.Tests/Helpers/MoneyHelperTests.cs`
- `FirstMile.Services.Tests/Commerce/CartServiceTests.cs`

## TASK

1. Add `CurrencyIsoCode` property to `FullAccountModel` (parent Account is the source of truth per PO decision — Location not needed):
   ```csharp
   [JsonPropertyName("CurrencyIsoCode")]
   public string? CurrencyIsoCode { get; set; }
   ```

2. Add currency-aware overloads to `MoneyHelper`:
   - `CulturePrice(double price, string? currencyCode)` — uses `CultureInfo("en-IE")` for EUR, `CultureInfo("en-GB")` for GBP/null/default
   - `FormatPromoPrice(double promoPrice, string? currencyCode)` — uses `€` for EUR
   - `GetCurrencySymbol(string? currencyCode)` — returns `"€"` for EUR, `"£"` for everything else

3. Add `CurrencyCode` property to `Cart` model (default `"GBP"`). Update:
   - `vat` getter: use `MoneyHelper.CulturePrice(totalVat, CurrencyCode)` instead of `ToString("C", new CultureInfo("en-GB"))`
   - `totalPrice` getter: use `MoneyHelper.CulturePrice(hiddenTotalPrice, CurrencyCode)` with `"£0"` fallback → `GetCurrencySymbol(CurrencyCode) + "0"` fallback
   - `notIncludeVatTotalPrice` getter: same pattern

4. Update `LineItem.price` getter to accept currency context. Options:
   - Add `CurrencyCode` property to `LineItem` (set when cart is built)
   - Or pass through a static/contextual approach
   - Use `MoneyHelper.CulturePrice(newPrice, CurrencyCode)`

5. In `CartService.GetLoggedInCartAsync`, after retrieving `account` (~line 121):
   ```csharp
   cart.CurrencyCode = account?.CurrencyIsoCode ?? "GBP";
   ```
   Also set `CurrencyCode` on each LineItem when building/returning the cart.
   **Note:** Guest flow is unchanged — only logged-in portal flow uses dynamic currency.

6. Unit tests:
   - `MoneyHelperTests`: test EUR formatting returns `€12.50`, GBP returns `£12.50`, null defaults to GBP
   - `CartServiceTests`: test that EUR account sets `CurrencyCode = "EUR"` on cart

## DONE WHEN

- [ ] `FullAccountModel` has `CurrencyIsoCode` property
- [ ] `MoneyHelper` has currency-aware overloads
- [ ] `Cart.vat`, `Cart.totalPrice`, `Cart.notIncludeVatTotalPrice` use dynamic currency formatting
- [ ] `LineItem.price` formats with correct currency
- [ ] `CartService` populates `CurrencyCode` from parent Account's Salesforce data
- [ ] Unit tests pass for EUR and GBP formatting
- [ ] Compiles without errors
- [ ] Unit tests pass (per testing-policy.md)
- [ ] No files outside CONTEXT modified
- [ ] No claim made about existing code without citing file:line
