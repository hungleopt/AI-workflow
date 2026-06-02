# FMI-944: (May) GZ Integration: EUR currency ordering

> Last Reviewed: 2026-05-29 07:44 UTC  
> Status: Is Estimated  
> Type: Story  
> Assignee: Hung Trinh

**Related tickets:** FMI-945 (EUR invoicing), FMI-946 — deployment dependency: cannot go live with FMI-944 until FMI-945 and FMI-946 are also ready to deploy.

## 1. Questions, Assumptions & Decisions

### Open Questions (Needs Answer)

- [ ] For GTM/DataLayer events, should the `currencyCode` reflect the customer's currency (i.e., `EUR` instead of `GBP`)? *(Not explicitly answered yet — assumed yes based on implementation intent.)*
- [ ] The checkout failure for EUR customers — is it a Stripe currency mismatch or portal-side validation? Caiti says "the error didn't go through to Salesforce so the log would be in Optimizely or Stripe". **Need to investigate Stripe/app logs on integration.**
- [ ] Stripe Dashboard currency/bank account settings: Does First Mile need to configure EUR bank account themselves, or should Opti update this as part of the ticket? *(Outstanding question from Caiti Black, 2026-05-27.)*

### Assumptions

- The Euro Price Book is already created and attached to the Location in Salesforce — the portal just needs to display the correct currency symbol.
- No backfilling of existing orders/invoices — only new orders need EUR support.
- All prices returned from Salesforce price book entries are already in the correct currency value — we only need to format/display with the right symbol.
- Stripe supports multi-currency by default; no special activation required. FX conversion fees are accepted by the business.

### Decisions

*(Confirmed by Caiti Black on 2026-05-26 and 2026-05-27 in ticket comments:)*

- **Currency source of truth**: Parent Account's `CurrencyIsoCode` field is the source of truth for the portal.
- **Scope**: GBP/EUR specific implementation only — no generic multi-currency solution required.
- **Precedence**: Salesforce Account `CurrencyIsoCode` takes precedence over site context (e.g., if user is on .ie but account has GBP, use GBP).
- **Guest order flow**: No change needed — EUR currency logic applies only to the logged-in portal flow.
- **New accounts from guest orders**: Not applicable (guest flow is unchanged).
- **Stripe**: Multi-currency works by default. No Stripe account configuration changes needed for payment acceptance. FX conversion fees are acceptable.
- **VAT rates for Ireland**: NOT part of this ticket — covered in FMI-945. Dependency exists.
- **Invoices/credit notes**: Separate ticket — invoice info comes from Salesforce, not CMS.
- **Direct Debit (BACS)**: Not answered explicitly, but BACS is UK-only; EUR customers presumably use card payment only.

## 2. Proposed Implementation

### Approach

Introduce a **currency context** for the **logged-in portal flow only** (guest flow unchanged). The currency is determined from the parent Account's `CurrencyIsoCode` field in Salesforce and propagated through the Cart model, price formatting, Stripe payments, and frontend display. Scope is limited to GBP/EUR — no generic multi-currency solution.

### Solution Details

1. **Add `CurrencyIsoCode` to Salesforce Account model** — `FullAccountModel` needs the new field. The **parent Account** is the source of truth (not Location).
2. **Store currency on Cart** — Add a `CurrencyCode` property to the `Cart` model. Populate it when building the logged-in cart from the Account.
3. **Dynamic price formatting** — Replace hardcoded `CultureInfo("en-GB")` and `£` symbols with a helper that respects the currency code (`en-GB` for GBP, `en-IE` for EUR, or simply symbol lookup).
4. **Stripe integration** — Pass the correct currency (`"gbp"` or `"eur"`) to `PaymentIntentCreateOptions.Currency`. Stripe supports multi-currency by default; no account config changes needed.
5. **Frontend** — The backend already sends formatted prices (e.g., `"£24.00"`) to the frontend as strings. If we format correctly on the backend, the frontend will display correctly. The GTM `currencyCode` needs to be dynamic.
6. **Checkout flow** — Ensure `CheckoutStepOnePageController` and `CheckoutStepTwoPageController` use dynamic currency for dataLayer and price display.

### Scope Exclusions (per PO decisions)

- **Guest order flow** — no changes (remains GBP).
- **VAT rates** — handled in FMI-945.
- **Invoices/credit notes** — separate ticket; data comes from Salesforce.
- **Direct Debit (BACS)** — UK-only payment method, EUR customers use card.

### Key Risk: Hardcoded `£` and `"en-GB"` throughout

The codebase has ~20+ locations where `£` is hardcoded or `CultureInfo("en-GB")` is used for price formatting. A centralized currency formatting utility is needed.

## 3. Detailed Task List

### 3.1 Models & Configuration

| #   | File Path                                         | Action | Description                                                                                                                                                       |
| --- | ------------------------------------------------- | ------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | `FirstMile.Salesforce/Models/FullAccountModel.cs` | Modify | Add `CurrencyIsoCode` property (`[JsonPropertyName("CurrencyIsoCode")]`) — this is the source of truth per PO decision                                            |
| 2   | `FirstMile.Services/Commerce/Models/Cart.cs`      | Modify | Add `CurrencyCode` property (string, default "GBP"). Use it in `vat`, `totalPrice`, `notIncludeVatTotalPrice` getters instead of hardcoded `CultureInfo("en-GB")` |

### 3.2 Services & Business Logic

| #   | File Path                                        | Action | Description                                                                                                                                                                                                  |
| --- | ------------------------------------------------ | ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1   | `FirstMile.Salesforce/Helpers/MoneyHelper.cs`    | Modify | Add `CulturePrice(double price, string currencyCode)` overload. Add `FormatPromoPrice(double, string currencyCode)` overload. Map "EUR" → `CultureInfo("en-IE")` or use `€` symbol, "GBP" → existing `en-GB` |
| 2   | `FirstMile.Services/Commerce/CartService.cs`     | Modify | When building logged-in cart (~line 143+), read `CurrencyIsoCode` from parent Account and set `cart.CurrencyCode`. Guest flow unchanged.                                                                     |
| 3   | `FirstMile.Services/Stripe/StripeService.cs`     | Modify | Accept `currency` parameter in `CreatePaymentIntent` (default `"gbp"`). Pass to `PaymentIntentCreateOptions.Currency`. Stripe supports EUR by default — no account config change.                            |
| 4   | `FirstMile.Services/Commerce/Models/LineItem.cs` | Modify | `price` getter uses `MoneyHelper.CulturePrice` — needs currency-aware version. May need to pass currency from cart context                                                                                   |

### 3.3 Integration

| #   | File Path                                                | Action | Description                                                                                       |
| --- | -------------------------------------------------------- | ------ | ------------------------------------------------------------------------------------------------- |
| 1   | `FirstMile.Salesforce/Services/PricebookEntryService.cs` | Review | No change needed — already returns `UnitPrice` as a number. Currency formatting happens elsewhere |

### 3.4 Controllers & Endpoints

| #   | File Path                                                                     | Action | Description                                                                                                                      |
| --- | ----------------------------------------------------------------------------- | ------ | -------------------------------------------------------------------------------------------------------------------------------- |
| 1   | `firstmile.web/Features/CartPage/CartPageController.cs`                       | Modify | Pass currency-aware formatted prices to frontend                                                                                 |
| 2   | `firstmile.web/Features/CheckoutStepOnePage/CheckoutStepOnePageController.cs` | Modify | Line 65: `currencyCode = "GBP"` → use `cart.CurrencyCode`. Line 200: `ToString("C", new CultureInfo("en-GB"))` → use MoneyHelper |
| 3   | `firstmile.web/Features/CheckoutStepTwoPage/CheckoutStepTwoPageController.cs` | Modify | Line 88: `currencyCode = "GBP"` → use `cart.CurrencyCode`. Line 239: `ToString("C", new CultureInfo("en-GB"))` → use MoneyHelper |
| 4   | `firstmile.web/Features/ThankYouPage/ThankYouPageController.cs`               | Modify | Line 34: `currencyCode = "GBP"` → dynamic from cart/order                                                                        |
| 5   | `firstmile.web/Features/PaymentInvoicePage/PaymentInvoicePageController.cs`   | Modify | Line 99: `$"Pay now £{amount}"` → use currency symbol. Lines 55/65: `Replace("£", "")` → handle both `£` and `€`                 |
| 6   | `firstmile.web/Api/CartController.cs`                                         | Review | Cart already returns formatted `totalPrice`/`vat` from model — no direct change if Cart model is fixed                           |
| 7   | `firstmile.web/Api/RecurringOrderController.cs`                               | Modify | Line 127: hardcoded `£{price}` → use currency from context                                                                       |

### 3.5 UI & Frontend

| #   | File Path                                    | Action | Description                                                                                                    |
| --- | -------------------------------------------- | ------ | -------------------------------------------------------------------------------------------------------------- |
| 1   | `firstmile.widgets/src/helpers/gtm.ts`       | Modify | `currencyCode: "GBP"` → accept dynamic currency from backend prop or global config                             |
| 2   | `firstmile.widgets/src/blocks/cart/Cart.tsx` | Modify | `currencyCode: 'GBP'` in `handlePushDataLayer` → use prop/context currency                                     |
| 3   | Backend cart/checkout views                  | Review | Prices are already rendered as formatted strings from backend — should auto-fix once backend formats correctly |

### 3.6 Wiring & DI

| #   | File Path                                     | Action | Description                                                               |
| --- | --------------------------------------------- | ------ | ------------------------------------------------------------------------- |
| 1   | `FirstMile.Services/Stripe/IStripeService.cs` | Modify | Add `currency` parameter to `CreatePaymentIntent` interface method        |
| 2   | `FirstMile.Services/Commerce/ICartService.cs` | Review | No interface change needed — Cart model property addition is non-breaking |

### 3.7 Unit Tests

| #   | Test File Path                                                                           | Tests to Add                                                                                     | Covers                                   |
| --- | ---------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------ | ---------------------------------------- |
| 1   | `FirstMile.Salesforce.Tests/Helpers/MoneyHelperTests.cs`                                 | `CulturePrice_EurPrice_ReturnsEuroCurrencyString`, `FormatPromoPrice_EurPrice_ReturnsEuroFormat` | New EUR formatting overloads             |
| 2   | `FirstMile.Services.Tests/Commerce/CartServiceTests.cs`                                  | `GetLoggedInCartAsync_EurAccount_SetsCurrencyCodeEUR`                                            | Cart currency assignment from SF account |
| 3   | `firstmile.web.Tests/Features/CheckoutStepOnePage/CheckoutStepOnePageControllerTests.cs` | `Index_EurCart_DataLayerHasEurCurrency`                                                          | Dynamic currencyCode in GTM              |
| 4   | `firstmile.web.Tests/Features/CheckoutStepTwoPage/CheckoutStepTwoPageControllerTests.cs` | `Index_EurCart_DataLayerHasEurCurrency`, `Data_EurCart_DisplaysEuroSubtotal`                     | Dynamic currency in checkout             |
| 5   | `firstmile.web.Tests/Features/PaymentInvoicePage/PaymentInvoicePageControllerTests.cs`   | `Index_EurInvoice_ShowsEuroSubmitLabel`                                                          | EUR payment button label                 |

Test file location convention:
```
Source:  FirstMile.Salesforce/Helpers/MoneyHelper.cs
Test:    FirstMile.Salesforce.Tests/Helpers/MoneyHelperTests.cs

Source:  FirstMile.Services/Commerce/CartService.cs
Test:    FirstMile.Services.Tests/Commerce/CartServiceTests.cs
```

### 3.8 Documentation

| #   | Doc File Path                        | Action | Description   |
| --- | ------------------------------------ | ------ | ------------- |
| 1   | `docs/src/tickets/FMI-944-review.md` | Create | This document |

## 4. QA Verification Notes

### Test Scenarios

| #   | Scenario                             | Steps                                                  | Expected Result                                          |
| --- | ------------------------------------ | ------------------------------------------------------ | -------------------------------------------------------- |
| 1   | EUR logged-in customer sees € prices | Log in as Euro Account Test 1, add products to cart    | All prices display with € symbol                         |
| 2   | EUR customer can proceed to checkout | Add products to cart, click Checkout                   | User reaches checkout step 1 without errors              |
| 3   | EUR customer can complete payment    | Complete checkout flow with Stripe                     | Payment intent created in EUR, order placed successfully |
| 4   | GBP customer unaffected              | Log in as existing GBP account, add products to cart   | All prices display with £ symbol as before               |
| 5   | Cart summary shows correct currency  | View cart page with EUR products                       | VAT, subtotal, total all show €                          |
| 6   | GTM tracking sends correct currency  | Check dataLayer events during add-to-cart and checkout | `currencyCode: "EUR"` for Euro customers                 |
| 7   | Guest flow unchanged                 | Order as guest on .ie site                             | Prices display as before (GBP), no EUR logic applied     |

### Edge Cases to Verify

- Mixed-currency scenario: Can a user with EUR account see any GBP products? (Expected: No — they only see Euro price book)
- Session cart created before login: If anonymous cart has items, what happens on login to EUR account?
- Promo codes: Do discounts work correctly with EUR prices?
- Recurring orders: Does the recurring order flow respect EUR currency?

### Regression Areas

- All existing GBP customer flows (ordering, cart, checkout, payment)
- Promo code application and display
- Order history display
- Cart postcode change flow
- GTM/analytics tracking

### Test Data Requirements

- Euro Account: `Euros Account Test 1` (Account ID: `001Pu00000iUFLtIAO`)
- Euro Location: `Euros Location Test 1` (Account ID: `001Pu00000iUCDrIAO`)
- Test Order: `07860837` (Order ID: `801Pu00000o78ihIAA`)
- Environment: Integration (full sandbox)

## 5. Risks & Concerns

### Security

- Stripe currency mismatch: Ensure the payment intent currency matches what was displayed to the customer. A mismatch could allow paying less in a weaker currency. **Validate currency server-side before creating payment intent.**

### Compliance

- EUR invoices may need different VAT registration details (Ireland vs UK entity). This is covered in FMI-945 — **deployment dependency exists.**

### Performance

- None identified. The `CurrencyIsoCode` field is fetched as part of existing Account queries — no additional API calls needed.

### Breaking Changes

- The `Cart` model properties `vat`, `totalPrice`, `price` (on LineItem) change their output format for EUR customers. Frontend components that parse these strings (e.g., `Replace("£", "")` in PaymentInvoicePageController) must be updated to handle both currencies.

### Deployment Dependencies

- **Cannot deploy FMI-944 to production until FMI-945 and FMI-946 are also ready** (confirmed by Caiti Black, 2026-05-27). All three tickets must be deployed together.
- `MoneyHelper.CulturePrice` return value changes format for EUR — any test assertions checking `"£..."` format will need currency-aware assertions.
- `IStripeService.CreatePaymentIntent` signature change requires updating all callers and mocks.
