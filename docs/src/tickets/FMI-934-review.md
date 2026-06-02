# FMI-934: Saved quote lead opportunities should not set Ecommerce Assisted

> Reviewed: 2026-05-28  
> Status: Unverified via API (Jira issue endpoint returned 404/401 with current credentials)  
> Type: Unverified via API

## 1. Questions, Assumptions & Decisions

### Open Questions (Needs Answer)

Items below may need product owner confirmation before the dev team can provide estimation.

- [ ] Should lead-level Ecommerce Assisted behavior remain unchanged for saved basket leads? Current code still sets lead Ecommerce Assisted when SavedQuoteLead is true (FirstMile.Services/OrderService.cs:98).

### Assumptions

- The latest Jira comment from Caiti is the source of truth and overrides previous behavior: for saved basket leads, Opportunity must keep Saved Quote Lead checked but must not set Ecommerce Assisted.
- We should preserve the existing behavior where checkout conversion for non-saved-quote flows can still mark Ecommerce Assisted.
- This is a small correction to existing mapping logic, not a contract redesign.

### Decisions

- Apply a minimal behavior fix in OrderService opportunity mapping so Ecommerce Assisted is not set when the opportunity is linked to a saved quote lead.
- Keep SavedQuoteLead and Notes mapping for saved basket leads.
- Add/adjust unit tests to lock this rule and avoid regression.

## 2. Proposed Implementation

### Approach

Adjust conditional mapping in OrderService so saved quote lead flows set only SavedQuoteLead (plus Notes), while Ecommerce Assisted is only set for website checkout opportunities that are not saved quote lead conversions.

### Solution Details

- Architecture decisions:
  - Keep all logic in the current orchestration point OrderCompleteWorkflow; no new services required.
  - Keep existing Salesforce DTOs unless omitting Ecommerce Assisted in updates requires nullable behavior.
- Data flow:
  - Current creation paths set both flags for cart.LeadId in two places:
    - FirstMile.Services/OrderService.cs:146-149
    - FirstMile.Services/OrderService.cs:189-192
  - Existing-opportunity update path currently always sets Ecommerce Assisted true:
    - FirstMile.Services/OrderService.cs:200-203
- Integration points:
  - Opportunity create payload: FirstMile.Services/Salesforce/Order/PostOpportunityRequest.cs:157-163
  - Opportunity update payload: FirstMile.Salesforce/Models/UpdateOpportunityModel.cs:6-10
- Error handling strategy:
  - Preserve current error handling pattern; no new exception paths introduced.

## 3. Detailed Task List

### 3.1 Models & Configuration

| #   | File Path                                             | Action                      | Description                                                                                                                   |
| --- | ----------------------------------------------------- | --------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| 1   | FirstMile.Salesforce/Models/UpdateOpportunityModel.cs | Evaluate Modify (if needed) | Consider nullable EcommerceAssisted field only if we must omit field updates for saved quote leads on existing opportunities. |

### 3.2 Services & Business Logic

| #   | File Path                          | Action | Description                                                                                                                                                        |
| --- | ---------------------------------- | ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1   | FirstMile.Services/OrderService.cs | Modify | In both opportunity creation branches, when cart.LeadId indicates saved quote lead flow, set SavedQuoteLead true and Notes, but do not set EcommerceAssisted true. |
| 2   | FirstMile.Services/OrderService.cs | Modify | In existing opportunity update branch, avoid forcing EcommerceAssisted true for saved quote lead conversion; keep stage transition behavior.                       |
| 3   | FirstMile.Services/OrderService.cs | Review | Confirm whether lead patch behavior at line 98 should stay as-is or be aligned to the opportunity rule.                                                            |

### 3.3 Integration

| #   | File Path                                                     | Action      | Description                                                                                                 |
| --- | ------------------------------------------------------------- | ----------- | ----------------------------------------------------------------------------------------------------------- |
| 1   | FirstMile.Services/Salesforce/Order/PostOpportunityRequest.cs | Verify only | Ensure fields map to Salesforce names already in use: eCommerce_Assisted__c, Saved_Quote_Lead__c, Notes__c. |

### 3.4 Controllers & Endpoints

| #   | File Path | Action | Description                                 |
| --- | --------- | ------ | ------------------------------------------- |
| 1   | N/A       | None   | No controller or endpoint changes required. |

### 3.5 UI & Frontend

| #   | File Path | Action | Description                      |
| --- | --------- | ------ | -------------------------------- |
| 1   | N/A       | None   | Backend Salesforce mapping only. |

### 3.6 Wiring & DI

| #   | File Path | Action | Description                       |
| --- | --------- | ------ | --------------------------------- |
| 1   | N/A       | None   | Existing DI wiring is sufficient. |

### 3.7 Unit Tests

| #   | Test File Path                                | Tests to Add                                                                               | Covers                                                                                        |
| --- | --------------------------------------------- | ------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------- |
| 1   | FirstMile.Services.Tests/OrderServiceTests.cs | OrderCompleteWorkflow_NewCustomerWithLeadId_DoesNotSetOpportunityEcommerceAssisted         | Regression for saved quote lead opportunity rule.                                             |
| 2   | FirstMile.Services.Tests/OrderServiceTests.cs | OrderCompleteWorkflow_ProspectWithLeadId_ExistingOpportunity_DoesNotForceEcommerceAssisted | Existing opportunity path does not incorrectly tick Ecommerce Assisted for saved quote leads. |
| 3   | FirstMile.Services.Tests/OrderServiceTests.cs | OrderCompleteWorkflow_ProspectWithoutLeadId_ExistingOpportunity_SetsEcommerceAssisted      | Preserve existing website conversion behavior for non-saved-quote flow.                       |

Test file location convention:

Source:  FirstMile.Services/OrderService.cs
Test:    FirstMile.Services.Tests/OrderServiceTests.cs

### 3.8 Documentation

| #   | Doc File Path                           | Action | Description                                                                                                    |
| --- | --------------------------------------- | ------ | -------------------------------------------------------------------------------------------------------------- |
| 1   | docs/src/backend/cart-and-order-flow.md | Update | Refine current wording at line 373 so it reflects conditional Ecommerce Assisted behavior (not unconditional). |

Guidelines:

- Existing docs are intentionally brief; expand the flow note for conditional opportunity flag logic.
- Include edge case note for saved quote lead conversion.

## 4. QA Verification Notes

### Test Scenarios

| #   | Scenario                                                   | Steps                                                           | Expected Result                                                                     |
| --- | ---------------------------------------------------------- | --------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| 1   | Saved quote lead checkout creates opportunity              | Create saved basket lead, then checkout                         | Opportunity has Saved Quote Lead checked; Ecommerce Assisted is not checked.        |
| 2   | Existing non-saved-quote opportunity converted via website | Use existing account/opportunity path without saved quote lead  | Ecommerce Assisted remains/ticks according to existing conversion rule.             |
| 3   | Existing saved-quote lead with existing opportunity        | Checkout with lead-linked flow where opportunity already exists | Stage updates to Handed Over; Ecommerce Assisted is not force-enabled by this flow. |

### Edge Cases to Verify

- cart.LeadId is present but lead record cannot be resolved.
- Existing opportunity already has Ecommerce Assisted true before update.

### Regression Areas

- Opportunity stage update behavior for existing opportunities.
- Notes__c mapping from saved quote line items.
- Lead patch path during checkout conversion.

### Test Data Requirements

- One saved basket lead with SavedQuoteLead true.
- One existing opportunity/account pair without saved quote lead conversion context.
- One existing opportunity/account pair with saved quote lead conversion context.

## 5. Risks & Concerns

### Security

- None identified (mapping logic only; no auth/session/billing changes).

### Compliance

- Low risk. Notes mapping may include user-entered data; ensure existing PII handling/logging practices remain unchanged.

### Performance

- None identified; no additional external calls are required for the minimal fix.

### Breaking Changes

- Low risk if limited to conditional flag mapping in OrderService.
- Medium risk only if UpdateOpportunityModel is changed to nullable behavior, because serialization behavior may change.
