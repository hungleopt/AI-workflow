# 002 - QA - EUR invoice PDF

## Affected Features

- Invoice PDF download (per-order and monthly templates)
- Invoice header display
- Bank details on invoice
- VAT labels on invoice

## Test Scenarios

| #   | Scenario                  | Steps                              | Expected                                           |
| --- | ------------------------- | ---------------------------------- | -------------------------------------------------- |
| 1   | EUR per-order invoice PDF | Download a per-order EUR invoice   | PDF shows € for all amounts, Euro bank details     |
| 2   | EUR monthly invoice PDF   | Download a monthly EUR invoice     | PDF shows € for all amounts, correct subtotals     |
| 3   | EUR credit note PDF       | Download EUR credit note           | Shows € symbol, "CREDIT NOTE" header               |
| 4   | GBP invoice PDF unchanged | Download a GBP invoice             | PDF shows £, GBP bank details (Sort Code 20-05-75) |
| 5   | Invoice page header       | View invoices page for EUR account | Outstanding amount shows €                         |
| 6   | PDF footer                | Download EUR invoice               | Footer shows correct legal entity info             |

## Regression Areas

- All existing GBP invoice PDF generation
- Monthly invoice detail sheet
- Credit note generation
- Invoice page header outstanding amount
- PDF layout/formatting (ensure € doesn't break table widths)
