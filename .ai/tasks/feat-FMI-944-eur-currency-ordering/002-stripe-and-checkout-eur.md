# 002 - BE - Stripe and checkout EUR

## CONTEXT

- Ticket: FMI-944
- Branch: `feat/FMI-944-eur-currency-ordering`
- Review: `docs/src/tickets/FMI-944-review.md`
- Depends on: `001-add-currency-infrastructure`

## FILES IN SCOPE

- `FirstMile.Services/Stripe/IStripeService.cs`
- `FirstMile.Services/Stripe/StripeService.cs`
- `FirstMile.Services/Commerce/CartService.cs` (SaveCartWithPaymentIntent method)
- `firstmile.web/Features/CheckoutStepOnePage/CheckoutStepOnePageController.cs`
- `firstmile.web/Features/CheckoutStepTwoPage/CheckoutStepTwoPageController.cs`
- `firstmile.web/Features/ThankYouPage/ThankYouPageController.cs`
- `firstmile.web/Features/CartPage/CartPageController.cs`
- `firstmile.web/Api/RecurringOrderController.cs`
- `firstmile.web.Tests/Features/CheckoutStepOnePage/CheckoutStepOnePageControllerTests.cs`
- `firstmile.web.Tests/Features/CheckoutStepTwoPage/CheckoutStepTwoPageControllerTests.cs`

## TASK

1. **Stripe service** — Modify `IStripeService.CreatePaymentIntent` signature to accept `string currency = "gbp"`:
   ```csharp
   Task<PaymentIntent?> CreatePaymentIntent(long amount, bool isLogin, string? firstName, string? lastName, string? email, string? accountId, string currency = "gbp");
   ```
   In `StripeService.CreatePaymentIntent`, use the `currency` parameter:
   ```csharp
   Currency = currency, // was hardcoded "gbp"
   ```
   **Note:** Stripe supports multi-currency by default. No Stripe account config changes needed (confirmed by PO).

2. **CartService.SaveCartWithPaymentIntent** — Pass `cart.CurrencyCode?.ToLowerInvariant() ?? "gbp"` to `CreatePaymentIntent`.
   **Security:** Validate that currency is only `"gbp"` or `"eur"` server-side before passing to Stripe.

3. **CheckoutStepOnePageController** (line ~65):
   - Change `currencyCode = "GBP"` → `currencyCode = cart?.CurrencyCode ?? "GBP"`
   - Line ~200: Change `ToString("C", new CultureInfo("en-GB"))` → `MoneyHelper.CulturePrice(sum, cart?.CurrencyCode)`

4. **CheckoutStepTwoPageController** (line ~88):
   - Change `currencyCode = "GBP"` → `currencyCode = cart?.CurrencyCode ?? "GBP"`
   - Line ~239: Change `ToString("C", new CultureInfo("en-GB"))` → `MoneyHelper.CulturePrice(sum, cart?.CurrencyCode)`

5. **ThankYouPageController** (line ~34):
   - Change `currencyCode = "GBP"` → dynamic from TempData or passed value

6. **CartPageController** (line ~74):
   - `totalPrice = cart.totalPrice ?? "£0"` — the fallback "£0" should use `MoneyHelper.GetCurrencySymbol(cart.CurrencyCode) + "0"`

7. **RecurringOrderController** (line ~127):
   - Change `"Estimated Cost - £{price} per month"` → use currency symbol from context

8. Unit tests:
   - Test that EUR cart creates payment intent with `"eur"` currency
   - Test that checkout dataLayer has `currencyCode: "EUR"` for EUR cart

## DONE WHEN

- [ ] `IStripeService.CreatePaymentIntent` accepts currency parameter
- [ ] `StripeService` passes currency to Stripe API
- [ ] `CartService.SaveCartWithPaymentIntent` passes cart currency
- [ ] Checkout controllers use dynamic `currencyCode` in dataLayer
- [ ] Checkout controllers use currency-aware price formatting
- [ ] CartPageController uses dynamic currency fallback
- [ ] RecurringOrderController uses dynamic currency symbol
- [ ] Compiles without errors
- [ ] Unit tests pass (per testing-policy.md)
- [ ] No files outside CONTEXT modified
- [ ] No claim made about existing code without citing file:line
