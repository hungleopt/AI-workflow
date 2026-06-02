# FMI-945: (June) GZ Integration: EUR invoicing and credit notes

> Last Reviewed: 2026-05-29 07:50 UTC  
> Status: Is Estimated  
> Type: Story  
> Assignee: Hung Trinh

**Related tickets:** FMI-944 (EUR currency ordering), FMI-946 (VAT exemption) — deployment dependency: cannot go live with FMI-945 until FMI-944 and FMI-946 are also ready to deploy. FMI-944 must be implemented first as it introduces the `CurrencyIsoCode` field and currency formatting helpers that FMI-945 depends on.

## 1. Questions, Assumptions & Decisions

### Open Questions (Needs Answer)

- [ ] What are the **correct VAT labels/rates** for EUR invoices? Ireland's standard VAT rate is 23% (vs UK's 20%). Are there multiple rates for different service types? *(Caiti: "TBC on requirement" — 2026-05-26)*
- [ ] Does the `Owning Entity` linked to EUR invoices already have the correct Irish bank details and VAT info configured in Salesforce, or do we need to add these to the entity record?
- [ ] **PO# missing on invoices** — Caiti noted "Side note, why are PO# missing - please fix this too" in the updated description. Is this a separate fix or part of this ticket's scope?

### Assumptions

- The `CurrencyIsoCode` field on the **Invoice Header** Salesforce object is the definitive source for determining GBP vs EUR display.
- FMI-944 will be implemented first, providing the `CurrencyIsoCode` field on Account/Location models and the currency-aware `MoneyHelper` utilities.
- **Customer confirmed**: existing invoices will be migrated in Salesforce to use the correct `CurrencyIsoCode`. Therefore, all invoices (including historic) will have the field populated — no special "legacy GBP fallback" logic needed beyond defaulting null to GBP.
- The numeric values in Salesforce invoice fields (`AmountExclVAT__c`, `VAT__c`, `AmountInclVAT_f__c`) are already in the correct currency — we only need to change the display symbol.
- The EUR invoice PDF uses the same layout/structure as the GBP invoice — only the currency symbol, bank details, and VAT labels change. *(Confirmed by template attachments.)*
- Credit notes follow the same pattern — EUR credit notes use `€` symbol and EUR-specific text. *(Confirmed by credit note template attachment.)*

### Decisions

*(Confirmed by Caiti Black in comments and attachments, 2026-05-26 and 2026-05-27:)*

- **Euro Invoice template**: Provided as attachment `Euro Invoice - Final V .pdf` (differences highlighted in yellow vs GBP version).
- **Euro Credit Note template**: Provided as attachment `Euro Credit Note - Final V.pdf`.
- **Euro bank details**: Included in the attached PDF templates (IBAN/BIC format). Dev team should extract from the PDFs.
- **VAT registration number**: **No change** — same VAT number used on EUR invoices as GBP.
- **Credit notes**: Yes, credit notes also need EUR currency display (confirmed, template provided).
- **Payment page**: The "Pay now" button should show `€` symbol (e.g., "Pay now €198306.85").
- **Scope expanded**: Ticket title updated to "(June) GZ Integration: EUR invoicing **and credit notes**".
- **Deployment dependency**: Cannot deploy until FMI-944 and FMI-946 are also ready (confirmed by Caiti on FMI-944).

## 2. Proposed Implementation

### Approach

Modify the invoice and **credit note** display, download (PDF generation), and payment flows to:
1. Read `CurrencyIsoCode` from the Salesforce Invoice Header object.
2. Use currency-aware formatting for all monetary values (€ for EUR, £ for GBP).
3. Render currency-specific bank details (IBAN/BIC for EUR, Sort Code/Account No for GBP) and VAT labels on the PDF.
4. Apply the same EUR treatment to **credit note** PDFs (confirmed by PO, template provided).
5. Handle EUR currency in the invoice payment flow (Stripe integration).
6. Investigate and fix missing PO# on invoices (noted by PO in description).

### Solution Details

1. **Add `CurrencyIsoCode` to invoice models** — Add the field to `GetInvoiceByIdResponse` and `InvoiceModel`.
2. **Invoice list display** — `AccountStatementService` hardcodes `£` in amount formatting. Replace with currency-aware formatting based on invoice's `CurrencyIsoCode`.
3. **Invoice PDF generation** — The `TemplatePerOrder` and `TemplateMonthlyAsync` methods in `AccountInvoicesPageController` have ~15 hardcoded `£` symbols. Replace with the correct symbol based on invoice currency. Conditionally show Euro bank details (IBAN/BIC from template) and correct VAT labels.
4. **Credit note PDF generation** — Apply the same EUR currency handling to credit note template rendering. Credit notes follow the same pattern as invoices but with "CREDIT NOTE" header.
5. **Invoice payment page** — `PaymentInvoicePageController` parses amounts using `Replace("£", "")` and formats with `$"Pay now £{amount}"`. Must handle both `£` and `€`.
6. **Stripe payment for invoices** — Use the invoice currency when creating the payment intent via `CartService.SaveCartWithPaymentIntent`.

### Key differences from EUR template (highlighted in attached PDFs):
- Currency symbol: `€` instead of `£`
- Bank details section: IBAN/BIC format instead of Sort Code/Account No
- VAT labels: rate TBC (awaiting confirmation — may be 23% for Ireland)
- VAT registration number: **unchanged** (same as GBP)
- Layout/structure: **unchanged** (same template layout)

### Key areas with hardcoded `£`:
- `AccountInvoicesPageController.cs`: lines 69, 405, 406, 462, 466, 470, 540, 541, 563, 578, 625, 629, 633
- `AccountStatementService.cs`: lines 170, 206, 220, 221
- `PaymentInvoicePageController.cs`: lines 55, 65, 99, 108
- `InvoicesController.cs`: implicit via `outstandingAmount`

## 3. Detailed Task List

### 3.1 Models & Configuration

| #   | File Path                                                          | Action | Description                                                                              |
| --- | ------------------------------------------------------------------ | ------ | ---------------------------------------------------------------------------------------- |
| 1   | `FirstMile.Services/Salesforce/Invoices/GetInvoiceByIdResponse.cs` | Modify | Add `[JsonPropertyName("CurrencyIsoCode")] public string? CurrencyIsoCode { get; set; }` |
| 2   | `FirstMile.Salesforce/Models/InvoiceModel.cs`                      | Modify | Add `[JsonPropertyName("CurrencyIsoCode")] public string? CurrencyIsoCode { get; set; }` |

### 3.2 Services & Business Logic

| #   | File Path                                                  | Action | Description                                                                                                                                                                    |
| --- | ---------------------------------------------------------- | ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1   | `FirstMile.Salesforce/Services/AccountStatementService.cs` | Modify | Lines 170, 206, 220, 221: Replace hardcoded `£` with currency symbol based on invoice `CurrencyIsoCode`. Pass currency from invoice data when building `AccountStatementModel` |
| 2   | `FirstMile.Salesforce/Helpers/MoneyHelper.cs`              | Modify | Add `GetCurrencySymbol(string currencyCode)` method returning `"£"` for GBP, `"€"` for EUR. (Builds on FMI-944 work)                                                           |
| 3   | `FirstMile.Services/Commerce/CartService.cs`               | Modify | `SaveCartWithPaymentIntent` — pass currency to Stripe service (builds on FMI-944 Stripe currency parameter)                                                                    |

### 3.3 Integration

| #   | File Path                                                              | Action | Description                                                         |
| --- | ---------------------------------------------------------------------- | ------ | ------------------------------------------------------------------- |
| 1   | `FirstMile.Services/Salesforce/Invoices/ISalesForceInvoicesService.cs` | Review | Ensure `GetInvoiceById` SOQL query includes `CurrencyIsoCode` field |
| 2   | Salesforce SOQL queries for invoice listing                            | Modify | Add `CurrencyIsoCode` to SELECT in invoice queries                  |

### 3.4 Controllers & Endpoints

| #   | File Path                                                                     | Action | Description                                                                                                                                                                                                                                                                          |
| --- | ----------------------------------------------------------------------------- | ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1   | `firstmile.web/Features/AccountInvoicesPage/AccountInvoicesPageController.cs` | Modify | **Download action**: Pass currency to template methods. **TemplatePerOrder**: Replace ~5 `£` with dynamic symbol; conditionally render Euro bank details. **TemplateMonthlyAsync**: Replace ~6 `£` with dynamic symbol. **IndexAsync**: Line 69 `"£" + amount` → use currency symbol |
| 2   | `firstmile.web/Features/PaymentInvoicePage/PaymentInvoicePageController.cs`   | Modify | Line 55/65: Parse amount with both `£` and `€` support. Line 99: Format with correct currency symbol. Line 108: Update error message for multi-currency                                                                                                                              |
| 3   | `firstmile.web/Api/InvoicesController.cs`                                     | Modify | Pass currency to outstanding amount formatting                                                                                                                                                                                                                                       |

### 3.5 UI & Frontend

| #   | File Path                   | Action | Description                                                                            |
| --- | --------------------------- | ------ | -------------------------------------------------------------------------------------- |
| 1   | Invoice list page component | Review | If amount is passed as pre-formatted string from backend, no FE change needed          |
| 2   | Invoice payment widget      | Review | `submitBtnLabel` is passed from backend — should auto-fix if backend formats correctly |

### 3.6 Wiring & DI

| #   | File Path                | Action | Description |
| --- | ------------------------ | ------ | ----------- |
| —   | No wiring changes needed | —      | —           |

### 3.7 Unit Tests

| #   | Test File Path                                                                           | Tests to Add                                                                                                                            | Covers                      |
| --- | ---------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- | --------------------------- |
| 1   | `FirstMile.Salesforce.Tests/Services/AccountStatementServiceTests.cs`                    | `GetInvoicesAndBankStatements_EurInvoice_FormatsWithEuroSymbol`                                                                         | EUR formatting in statement |
| 2   | `FirstMile.Salesforce.Tests/Helpers/MoneyHelperTests.cs`                                 | `GetCurrencySymbol_EUR_ReturnsEuro`, `GetCurrencySymbol_GBP_ReturnsPound`, `GetCurrencySymbol_Null_DefaultsPound`                       | Currency symbol helper      |
| 3   | `firstmile.web.Tests/Features/AccountInvoicesPage/AccountInvoicesPageControllerTests.cs` | `Download_EurInvoice_PdfContainsEuroSymbol`, `Download_EurInvoice_ShowsEuroBankDetails`, `Download_EurCreditNote_PdfContainsEuroSymbol` | EUR invoice/credit note PDF |
| 4   | `firstmile.web.Tests/Features/PaymentInvoicePage/PaymentInvoicePageControllerTests.cs`   | `Index_EurInvoices_ParsesEuroAmounts`, `Index_EurInvoices_ShowsEuroPayButton`                                                           | EUR payment flow            |

Test file location convention:
```
Source:  FirstMile.Salesforce/Services/AccountStatementService.cs
Test:    FirstMile.Salesforce.Tests/Services/AccountStatementServiceTests.cs

Source:  firstmile.web/Features/AccountInvoicesPage/AccountInvoicesPageController.cs
Test:    firstmile.web.Tests/Features/AccountInvoicesPage/AccountInvoicesPageControllerTests.cs
```

### 3.8 Documentation

| #   | Doc File Path                        | Action | Description   |
| --- | ------------------------------------ | ------ | ------------- |
| 1   | `docs/src/tickets/FMI-945-review.md` | Create | This document |

## 4. QA Verification Notes

### Test Scenarios

| #   | Scenario                       | Steps                                                  | Expected Result                                                   |
| --- | ------------------------------ | ------------------------------------------------------ | ----------------------------------------------------------------- |
| 1   | EUR invoice displays € on list | Log in as Euro Account, navigate to Invoices page      | Outstanding amount and invoice amounts show € symbol              |
| 2   | EUR invoice PDF download       | Click download on a EUR invoice                        | PDF shows €, Euro bank details (IBAN/BIC), correct VAT labels     |
| 3   | EUR invoice payment            | Select EUR invoices, click Pay Now                     | Payment page shows "Pay now €X", Stripe processes in EUR          |
| 4   | GBP invoice unaffected         | Log in as existing GBP account, view/download invoices | All invoices display £ as before                                  |
| 5   | Mixed account statement        | View statement with historic GBP and new EUR invoices  | Each line shows correct currency based on its own CurrencyIsoCode |
| 6   | EUR credit note display        | View credit notes list for EUR account                 | Credit notes show € symbol in amounts                             |
| 7   | EUR credit note PDF download   | Download a credit note for EUR invoice                 | Shows €, "CREDIT NOTE" header, Euro bank details, correct VAT     |
| 8   | GBP credit note unaffected     | Download a credit note for GBP account                 | Shows £ as before                                                 |

### Edge Cases to Verify

- Historic invoices for Irish accounts — customer will migrate these to have correct `CurrencyIsoCode`; verify they display correctly after migration
- Invoice with null/empty `CurrencyIsoCode` — should default to GBP (£)
- Credit notes linked to EUR invoices
- Monthly invoices with mixed locations (if possible)
- Invoice payment with multiple invoices selected (all same currency expected)

### Regression Areas

- All existing GBP invoice display, download, and payment flows
- Account statement page (CSV export)
- Invoice payment confirmation flow
- Legal entity footer on PDF (VAT number, company reg)

### Test Data Requirements

- Euro Account: `Euros Account Test 1` (Account ID: `001Pu00000iUFLtIAO`)
- Euro Location: `Euros Location Test 1` (Account ID: `001Pu00000iUCDrIAO`)
- EUR invoice: needs to be created in Salesforce with `CurrencyIsoCode = EUR`
- Euro bank details (to be provided by business)
- Euro invoice template (attached to Jira ticket — needs to be shared)
- Environment: Integration (full sandbox)

## 5. Risks & Concerns

### Security

- Invoice payment amount parsing: Currently uses `Replace("£", "")` to extract numeric values from form submissions. If we add `€` support, ensure validation prevents injection of unexpected characters. **Server-side validation of parsed amounts is critical.**
- Stripe currency: Must match the invoice currency — a EUR invoice paid with a GBP payment intent would fail or charge the wrong amount.

### Compliance

- **VAT rates**: Ireland has a different standard VAT rate (23%) compared to UK (20%). If the VAT is calculated server-side, this needs to be accounted for. However, if Salesforce already provides the correct VAT amount, this is a display-only concern.
- **Legal entity**: EUR invoices may need to reference a different legal entity (Irish company registration, Irish VAT number, IBAN bank details). The current `LegalEntity` approach via `OwningEntityC` should handle this if configured correctly in Salesforce.
- **Bank details**: EUR payments likely go to a different bank account (IBAN/BIC format instead of Sort Code/Account No). The PDF template must show the correct details.

### Performance

- None identified. The additional `CurrencyIsoCode` field is a simple string field — no performance impact.

### Breaking Changes

- `AccountStatementModel.Amount` and `OutstandingAmount` format changes from always `£` to dynamic. Any frontend parsing of these strings needs updating.
- `PaymentInvoicePageController` form data format: items are submitted as `"INV001-£12.5"`. If EUR invoices use `€`, the parsing logic must handle both. Consider changing to a more robust format (e.g., `"INV001-12.5-EUR"`) to avoid symbol-parsing issues.
