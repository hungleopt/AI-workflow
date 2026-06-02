# 003 - QA - Frontend GTM currency

## Affected Features

- Google Tag Manager dataLayer events
- Cart add/remove tracking
- Frontend cart component

## Test Scenarios

| #   | Scenario                  | Steps                               | Expected                                                 |
| --- | ------------------------- | ----------------------------------- | -------------------------------------------------------- |
| 1   | EUR add to cart GTM event | Login EUR account → add item        | dataLayer event has `currencyCode: "EUR"`                |
| 2   | GBP add to cart GTM event | Login GBP account → add item        | dataLayer event has `currencyCode: "GBP"`                |
| 3   | Remove from cart GTM      | EUR account → remove item from cart | dataLayer removeFromCart event has `currencyCode: "EUR"` |

## Regression Areas

- All existing GTM tracking for GBP customers
- Cart component rendering and functionality
- Add to cart / remove from cart flows
