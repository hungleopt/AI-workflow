# 001 - QA - Invoice models and statement

## Affected Features

- Invoice list page (amounts column)
- Account statement display
- Salesforce invoice data model

## Test Scenarios

| #   | Scenario                      | Steps                                               | Expected                                     |
| --- | ----------------------------- | --------------------------------------------------- | -------------------------------------------- |
| 1   | EUR invoice shows € on list   | Login EUR account → Invoices page                   | EUR invoices show € amounts                  |
| 2   | GBP invoice shows £ on list   | Login GBP account → Invoices page                   | All amounts show £                           |
| 3   | Mixed invoices (if possible)  | Account with both GBP and EUR invoices              | Each shows correct symbol                    |
| 4   | Null currency defaults to £   | Invoice without CurrencyIsoCode                     | Displays with £ (safe default)               |
| 5   | Migrated historic EUR invoice | Previously GBP invoice now with CurrencyIsoCode=EUR | Displays with €                              |
| 6   | Outstanding amount header     | View Invoices page header                           | Shows correct currency for outstanding total |

## Regression Areas

- Invoice list display for all existing accounts
- Account statement CSV export
- Statement filtering by type/status
- Pay Now button functionality
