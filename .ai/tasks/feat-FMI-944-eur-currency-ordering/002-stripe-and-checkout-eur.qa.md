# 002 - QA - Stripe and checkout EUR

## Affected Features

- Stripe payment creation (currency parameter)
- Checkout Step 1 and Step 2 pages (price display, GTM tracking)
- Thank You page GTM tracking
- Cart page total display
- Recurring order price display

## Test Scenarios

| #   | Scenario                               | Steps                                      | Expected                                             |
| --- | -------------------------------------- | ------------------------------------------ | ---------------------------------------------------- |
| 1   | EUR checkout creates EUR payment       | Login EUR account → add to cart → checkout | Stripe payment intent created with `currency: "eur"` |
| 2   | GBP checkout still creates GBP payment | Login GBP account → add to cart → checkout | Stripe payment intent with `currency: "gbp"`         |
| 3   | Checkout dataLayer EUR                 | EUR account at checkout step 1/2           | `currencyCode: "EUR"` in dataLayer                   |
| 4   | Checkout subtotal shows €              | EUR account at checkout                    | Subtotal formatted with €                            |
| 5   | Cart page shows € total                | EUR account views cart page                | Total price shows € symbol                           |
| 6   | Payment completes for EUR              | EUR account completes payment              | Order placed, redirected to thank you page           |

## Regression Areas

- All existing GBP checkout flows (step 1, step 2, payment, thank you)
- Stripe payment intent creation for all payment types (card, invoice)
- GTM analytics tracking
- Recurring order price display
- Cart page display for logged-in and anonymous users
