# 003 - QA - EUR invoice payment

## Affected Features

- Invoice payment page (Pay Now flow)
- Stripe payment creation for invoices
- Invoice amount parsing from form data

## Test Scenarios

| #   | Scenario                    | Steps                                   | Expected                                       |
| --- | --------------------------- | --------------------------------------- | ---------------------------------------------- |
| 1   | Pay EUR invoice             | Select EUR invoice → click Pay Now      | Payment page shows "Pay now €X", Stripe in EUR |
| 2   | Pay GBP invoice             | Select GBP invoice → click Pay Now      | Payment page shows "Pay now £X", Stripe in GBP |
| 3   | Pay multiple EUR invoices   | Select 2+ EUR invoices → click Pay Now  | Total is summed correctly, shown in €          |
| 4   | EUR payment completes       | Complete Stripe payment for EUR invoice | Payment captured, redirected to confirmation   |
| 5   | Zero amount validation      | Submit with no valid items              | Error message displayed (generic, no currency) |
| 6   | Excessive amount validation | Somehow trigger > 999,999.99            | Error message displayed                        |

## Regression Areas

- All existing GBP invoice payment flows
- Multi-invoice selection and payment
- Payment confirmation/redirect
- Stripe payment capture for invoices
- Form data parsing (invoice ID extraction)
