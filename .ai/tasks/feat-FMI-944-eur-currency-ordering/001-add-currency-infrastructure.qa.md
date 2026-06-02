# 001 - QA - Currency infrastructure

## Affected Features

- Cart price display (VAT, total price, line item prices)
- Price formatting utility (`MoneyHelper`)
- Salesforce Account/Location data model

## Test Scenarios

| #   | Scenario                      | Steps                                       | Expected                                       |
| --- | ----------------------------- | ------------------------------------------- | ---------------------------------------------- |
| 1   | EUR account cart shows €      | Login as EUR account → add item → view cart | VAT, total, line item prices all show € symbol |
| 2   | GBP account cart shows £      | Login as GBP account → add item → view cart | All prices show £ as before                    |
| 3   | Anonymous cart defaults to £  | Without login → add item → view cart        | Prices show £ (default GBP)                    |
| 4   | Null CurrencyIsoCode defaults | Account with no CurrencyIsoCode → view cart | Prices default to £                            |

## Regression Areas

- All existing GBP cart flows
- Line item pricing after add-to-cart
- Cart summary (VAT row, total row)
- Promo code display format
