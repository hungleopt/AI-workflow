# FMI-942: (May/June) New customer flow ; adding a saved payment method

> Last Reviewed: 2026-05-28 09:23 UTC  
> Status: In Progress  
> Type: Story

## 1. Questions, Assumptions & Decisions

### Open Questions (Needs Answer)

Items below may need product owner confirmation before the dev team can provide estimation.

- [ ] **PCI Compliance — Card Data Handling:** The ticket states "IF saved card then capture all card details including billing address." Under PCI DSS, our servers must NEVER receive or store raw card numbers, CVV, or expiry dates. **Confirm that "capture card details" means using Stripe's SetupIntent + PaymentElement (tokenized), where card data goes directly to Stripe and we only store the Stripe PaymentMethod ID.** If the intention is to store raw card details server-side, this is a PCI violation and cannot be implemented.
- [ ] **PCI Compliance — Direct Debit Account Number:** The ticket stores full bank account numbers (`dd_account_number__c`) in Salesforce. While DD data is not governed by PCI DSS, it may fall under FCA/BACS compliance. **Is Salesforce configured with field-level encryption for these sensitive financial fields?** If not, this is a compliance risk.
- [ ] **New Customer Definition:** The ticket says "new customers only (none logged in users)." However, it also says "without a Salesforce Parent Account ID." These are different conditions — a user could be logged in but not have a Parent Account ID yet (e.g., prospect). **Which condition defines "new customer"?** Recommendation: `!cart.ReorderContext.IsLoggedIn` (not logged in).
- [ ] **Stripe SetupIntent vs PaymentIntent:** For Step 1 (saving card for future), we need a Stripe SetupIntent (not a PaymentIntent). For Step 2 (taking first payment), we already use a PaymentIntent. **Confirm we should use SetupIntent in Step 1, then optionally use the saved PaymentMethod in Step 2's PaymentIntent.**
- [ ] **Saved Card Reuse in Step 2:** If a customer saves a card in Step 1, should Step 2 auto-populate and charge that same card? Or should the customer re-enter/confirm? The mockup shows card details pre-filled — **confirm Stripe off-session charge is acceptable here or if the customer must confirm via the PaymentElement.**
- [ ] **Direct Debit — Mandate Collection:** Setting `Direct_Debit_authorised__c = TRUE` implies the customer has agreed to a DD mandate. **Is there a legal disclaimer / terms checkbox required in the UI before setting this flag?** DD mandate requirements are regulated by BACS.
- [ ] **Payment Method Mutual Exclusivity:** Can a customer set up BOTH Direct Debit AND a saved card? Or is it one or the other per account? The ticket says "either Direct Debit or a saved credit/debit card" — confirming it's mutually exclusive.
- [ ] **Stripe Customer — New vs Existing Metadata:** Currently `StripeService.CreatePaymentIntent` only creates a Stripe customer for logged-in users. For new customers, no Stripe customer is created. **Should we now create a Stripe Customer for anonymous/new users at Step 1 (saved card flow)?**

### Assumptions

- Card details will be collected exclusively via Stripe Elements (PaymentElement / SetupIntent) — raw card data never touches our backend, maintaining PCI DSS SAQ-A compliance.
- The "Save card" flow will use Stripe SetupIntent to tokenize and attach the PaymentMethod to a Stripe Customer, then use that PaymentMethod for the initial PaymentIntent in Step 2.
- Direct Debit data (account holder, account number, sort code) will be sent server-side and stored in Salesforce. This is not PCI-scoped but is sensitive financial data requiring input validation and TLS-only transmission.
- The new "payment method selection" step will be inserted as a new intermediate checkout step (between current Step 1 and Step 2) or as an additional section within the existing Step 1 page.
- `enqix_payment_method` values are: `"Direct Debit"` and `"Credit Card"` (matching existing usage in `RecurringOrderService`).
- Stripe customer identity is account-scoped, not user-scoped: every flow that creates or reuses a Stripe customer must sync that Stripe customer ID to the Salesforce parent account, and must sync the Salesforce parent account ID back to Stripe for future payments. **Confirmed (May 26):** `stripe_customer_ID` field is on Parent Account; Stripe Customer ID is specific to the account and not the user.

### Decisions

- ~~Per ticket comment from Caiti Black: this is for estimation only — no code changes until estimation is approved.~~ **Estimate approved (May 22)** — Caiti confirmed "Please go ahead with it."
- User's additional context: PCI compliance must be carefully checked — this is flagged as a critical concern throughout this review.
- **Stripe Customer ID location confirmed (May 26):** `stripe_customer_ID` is stored on the **Salesforce Parent Account** (not Contact). "Stripe Customer ID is specific to the account and not the user." — confirmed by Caiti Black.
- **Stripe Card Elements UI (May 26):** Cuong proposed rendering Stripe Card Elements in steps 2 and 3 using the same UI as the current checkout payment page (Stripe Elements have limited customization). Approved by Tuyen.
- **Logged-in user checkout steps (May 22, AC update):** Remove numbered steps for logged-in users — logged-in customers go straight to step three, so it should just say "Checkout" instead of showing numbered steps.

## 2. Proposed Implementation

### Approach

This feature adds a multi-step payment method setup flow for new (anonymous) customers during checkout. The architecture splits into:

1. **Frontend**: New React component for payment method selection (DD form or Stripe SetupIntent)
2. **Backend**: New API endpoint(s) for SetupIntent creation, updated order workflow for payment method persistence
3. **Stripe Integration**: SetupIntent creation, Customer creation for anonymous users, PaymentMethod attachment, and Stripe customer metadata sync with Salesforce parent account ID
4. **Salesforce Integration**: Update the Salesforce parent account with DD details or credit card payment method, persist the Stripe customer ID on that parent account for all accounts, and send the Salesforce parent account ID back to Stripe customer metadata for future payments

### Solution Details

**Architecture:**
- New checkout step inserted between current Step 1 (customer details) and Step 2 (payment) — conceptually "Step 1.5"
- Payment method selection state stored in session cart (`Cart.PaymentMethodSelection`)
- Stripe SetupIntent created server-side when "Saved Card" is selected
- DD details validated server-side before being sent to Salesforce

**Data Flow (Saved Card):**
1. User selects "Saved Card" → Frontend requests SetupIntent from new API endpoint
2. Stripe PaymentElement renders with SetupIntent client secret
3. User completes card form → Stripe tokenizes directly (PCI compliant)
4. SetupIntent confirmed → PaymentMethod ID stored in cart session
5. Step 2: PaymentIntent created with `payment_method` from SetupIntent, `customer` = new Stripe Customer
6. On order completion: Stripe Customer ID saved to the Salesforce parent account, the Salesforce parent account ID saved back onto the Stripe customer record, and `enqix_payment_method = "Credit Card"` set on the account

**Data Flow (Direct Debit):**
1. User selects "Direct Debit" → Custom form captures DD details
2. Details validated server-side (sort code format, account number length)
3. DD details stored in cart session (temporary, cleared after order)
4. Step 2: First payment still taken by card (upfront), DD is for future recurring
5. On order completion: DD details written to Salesforce parent account fields, the Stripe Customer ID saved to that same parent account, the Salesforce parent account ID saved back onto the Stripe customer record, and `enqix_payment_method = "Direct Debit"` set on the account

**Error Handling:**
- SetupIntent failure: show Stripe error message, allow retry
- DD validation failure: inline form errors
- Salesforce update failure: log error + send error email (existing pattern), do not block order

**Integration Points:**
- `StripeService` — new methods: `CreateSetupIntent`, `CreateCustomerForAnonymous`, `AttachPaymentMethod`
- `AccountService.UpdateDirectDebitAsync` — already exists, reuse
- `OrderService.OrderCompleteWorkflow` — extend for payment method persistence
- New React component in `firstmile.widgets/src/blocks/checkout/`

## 3. Detailed Task List

### 3.1 Models & Configuration

| #   | File Path                                                      | Action | Description                                                                                                            |
| --- | -------------------------------------------------------------- | ------ | ---------------------------------------------------------------------------------------------------------------------- |
| 1   | `FirstMile.Services/Commerce/Models/Cart.cs`                   | Modify | Add `PaymentMethodSelection` model to cart (selected method type, DD details, SetupIntent ID, Stripe PaymentMethod ID) |
| 2   | `FirstMile.Services/Commerce/Models/PaymentMethodSelection.cs` | Create | New model: `PaymentMethodType` enum (DirectDebit, SavedCard), DD fields, Stripe references                             |
| 3   | `FirstMile.Services/Stripe/StripeAppSettingsConfiguration.cs`  | Verify | Confirm existing config supports SetupIntent (no changes expected)                                                     |

### 3.2 Services & Business Logic

| #   | File Path                                            | Action | Description                                                                                                                                                                                                                          |
| --- | ---------------------------------------------------- | ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1   | `FirstMile.Services/Stripe/IStripeService.cs`        | Modify | Add `CreateSetupIntent`, `CreateCustomerForNewUser`, `GetPaymentMethodFromSetupIntent`, and a method to update Stripe customer metadata with Salesforce parent account ID                                                            |
| 2   | `FirstMile.Services/Stripe/StripeService.cs`         | Modify | Implement SetupIntent creation, anonymous Customer creation with account-scoped metadata, PaymentMethod retrieval, and Stripe customer metadata sync for Salesforce parent account ID                                                |
| 3   | `FirstMile.Services/OrderService.cs`                 | Modify | In `OrderCompleteWorkflow` — after parent account resolution/creation, update `enqix_payment_method`, DD details, save Stripe Customer ID on the Salesforce parent account, and push the Salesforce parent account ID back to Stripe |
| 4   | `FirstMile.Services/Helpers/DirectDebitValidator.cs` | Create | Validate UK sort code (6 digits, XX-XX-XX format) and account number (8 digits). Input sanitization.                                                                                                                                 |
| 5   | `FirstMile.Services/Commerce/CartService.cs`         | Modify | Update `SaveCartWithPaymentIntent` to handle SetupIntent → PaymentIntent flow when saved card selected                                                                                                                               |

### 3.3 Integration

| #   | File Path                                                      | Action | Description                                                                                                                                 |
| --- | -------------------------------------------------------------- | ------ | ------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | `FirstMile.Salesforce/Models/UpdateAccountDirectDebitModel.cs` | Verify | Confirm the existing account update model covers DD fields and whether it should also carry `stripe_customer_ID` for parent-account updates |
| 2   | `FirstMile.Salesforce/Interfaces/IAccountService.cs`           | Modify | Add or extend parent-account update method to persist `stripe_customer_ID` alongside DD/payment-method fields for all accounts              |
| 3   | `FirstMile.Salesforce/Models/UpdateAccountDirectDebitModel.cs` | Modify | Add parent-account Stripe customer ID field if the existing model does not already expose it                                                |
| 4   | `FirstMile.Salesforce/Interfaces/IContactService.cs`           | Verify | **Confirmed: No Stripe ID persistence on Contact** — stripe_customer_ID is on Parent Account only (Caiti, May 26)                           |

### 3.4 Controllers & Endpoints

| #   | File Path                                                                     | Action | Description                                                                                                                              |
| --- | ----------------------------------------------------------------------------- | ------ | ---------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | `firstmile.web/Api/SetupIntentController.cs`                                  | Create | POST endpoint to create Stripe SetupIntent for anonymous users, returns client secret                                                    |
| 2   | `firstmile.web/Api/PaymentMethodController.cs`                                | Create | POST endpoint to save selected payment method to cart (DD details or Stripe PaymentMethod ID)                                            |
| 3   | `firstmile.web/Features/CheckoutStepTwoPage/CheckoutStepTwoPageController.cs` | Modify | In `Data()` method — pass saved payment method info to frontend; in `Confirm()` — use saved PaymentMethod for PaymentIntent if available |
| 4   | `firstmile.web/Features/CheckoutStepOnePage/CheckoutStepOnePageController.cs` | Modify | Add payment method selection section data to the step one page model                                                                     |

### 3.5 UI & Frontend

| #   | File Path                                                       | Action | Description                                                                                                   |
| --- | --------------------------------------------------------------- | ------ | ------------------------------------------------------------------------------------------------------------- |
| 1   | `firstmile.widgets/src/blocks/checkout/PaymentMethodSetup.tsx`  | Create | New component: radio selection (Direct Debit / Saved Card), conditional DD form or Stripe SetupIntent element |
| 2   | `firstmile.widgets/src/blocks/checkout/DirectDebitForm.tsx`     | Create | Account holder, account number, sort code inputs with validation                                              |
| 3   | `firstmile.widgets/src/blocks/checkout/Payment.tsx`             | Modify | Handle pre-selected payment method from Step 1 — if saved card exists, use stored PaymentMethod ID            |
| 4   | `firstmile.widgets/src/blocks/checkout/Checkout.tsx`            | Modify | Integrate new payment method step into checkout flow for anonymous users only                                 |
| 5   | `firstmile.widgets/src/blocks/checkout/CheckoutDetailsForm.tsx` | Modify | Add payment method setup section after customer details (for new customers only)                              |
| 6   | `firstmile.widgets/src/blocks/checkout/Checkout.tsx`            | Modify | Remove numbered step indicators for logged-in users — show just "Checkout" instead of "Step 1/2/3" (new AC)   |

### 3.6 Wiring & DI

| #   | File Path                                          | Action | Description                                                                               |
| --- | -------------------------------------------------- | ------ | ----------------------------------------------------------------------------------------- |
| 1   | `firstmile.web/Startup.cs` or DI registration file | Modify | Register new `SetupIntentController` and `PaymentMethodController` if not auto-registered |

### 3.7 Unit Tests

| #   | Test File Path                                                  | Tests to Add                                                                                                                                                                                                                             | Covers                                                                  |
| --- | --------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| 1   | `FirstMile.Services.Tests/Stripe/StripeServiceTests.cs`         | `CreateSetupIntent_ReturnsClientSecret`, `CreateCustomerForNewUser_SetsMetadata_NewCustomer`, `CreateCustomerForNewUser_SetsMetadata_ExistingCustomer`                                                                                   | Stripe SetupIntent and Customer creation                                |
| 2   | `FirstMile.Services.Tests/Helpers/DirectDebitValidatorTests.cs` | `Validate_ValidSortCode_ReturnsTrue`, `Validate_InvalidSortCode_ReturnsFalse`, `Validate_ValidAccountNumber_ReturnsTrue`, `Validate_ShortAccountNumber_ReturnsFalse`, `Validate_AlphaInAccountNumber_ReturnsFalse`                       | DD input validation                                                     |
| 3   | `FirstMile.Services.Tests/OrderServiceTests.cs`                 | `OrderCompleteWorkflow_DirectDebit_UpdatesParentAccountPaymentMethodAndStripeCustomerId`, `OrderCompleteWorkflow_SavedCard_UpdatesParentAccountAndStripeMetadata`, `OrderCompleteWorkflow_ExistingAccount_ResyncsStripeAndSalesforceIds` | Payment method persistence and bi-directional ID sync in order workflow |
| 4   | `firstmile.web.Tests/Api/SetupIntentControllerTests.cs`         | `Create_ReturnsClientSecret`, `Create_HandlesStripeError`                                                                                                                                                                                | API endpoint                                                            |
| 5   | `firstmile.web.Tests/Api/PaymentMethodControllerTests.cs`       | `Save_DirectDebit_ValidInput_SavesToCart`, `Save_DirectDebit_InvalidSortCode_ReturnsBadRequest`, `Save_SavedCard_StoresPaymentMethodId`                                                                                                  | Payment method save endpoint                                            |

### 3.8 Documentation

| #   | Doc File Path                             | Action | Description                                                             |
| --- | ----------------------------------------- | ------ | ----------------------------------------------------------------------- |
| 1   | `docs/src/backend/cart-and-order-flow.md` | Update | Add section on payment method selection step, SetupIntent flow, DD flow |
| 2   | `docs/src/tickets/FMI-942-review.md`      | Create | This review document                                                    |

## 4. QA Verification Notes

### Test Scenarios

| #   | Scenario                                                | Steps                                                                                                                                                                | Expected Result                                                                                                                                                                                                            |
| --- | ------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | New customer selects Direct Debit                       | Add items → Checkout → Enter customer details → Select "Direct Debit" → Fill DD form → Proceed to payment → Pay by card → Complete                                   | Order created, parent account has DD fields populated, `enqix_payment_method = "Direct Debit"`, Stripe Customer ID stored on the Salesforce parent account, and Salesforce parent account ID stored on the Stripe customer |
| 2   | New customer selects Saved Card                         | Add items → Checkout → Enter customer details → Select "Saved Card" → Enter card in Stripe element → Proceed to payment → Confirm payment with saved card → Complete | Order created, Stripe Customer has saved PaymentMethod, `enqix_payment_method = "Credit Card"`, Stripe Customer ID stored on the Salesforce parent account, and Salesforce parent account ID stored on the Stripe customer |
| 3   | Existing account checkout resyncs account-level IDs     | Log in with an existing account that already has a Salesforce parent account → Checkout with either payment path → Complete order                                    | Existing parent account retains/updates `stripe_customer_ID` as needed and Stripe customer metadata contains the Salesforce parent account ID                                                                              |
| 4   | Logged-in customer does NOT see payment method step     | Log in → Add items → Checkout                                                                                                                                        | Payment method selection step is NOT shown; normal checkout flow. Step labels show just "Checkout" without numbered steps.                                                                                                 |
| 5   | New customer — no saved card → option to save at Step 2 | Select DD in Step 1 → Proceed to payment → Checkbox "Save card for future" → Complete                                                                                | Card saved to Stripe if checkbox ticked                                                                                                                                                                                    |
| 6   | Repeat order with saved card                            | Log in (has saved card) → Add items → Checkout → Payment page                                                                                                        | Saved card details shown pre-populated in payment form                                                                                                                                                                     |
| 7   | Repeat order with Direct Debit                          | Log in (has DD method) → Add items → Checkout → Payment page                                                                                                         | "Place order now" button shown (no card payment form)                                                                                                                                                                      |
| 8   | Stripe SetupIntent failure                              | Enter invalid card details in Step 1 saved card form                                                                                                                 | Error message shown, user can retry                                                                                                                                                                                        |
| 9   | DD validation failure                                   | Enter invalid sort code (e.g., "ABC") or short account number                                                                                                        | Inline validation error, cannot proceed                                                                                                                                                                                    |

### Edge Cases to Verify

- User navigates back from Step 2 to Step 1 — does payment method selection persist?
- User changes payment method selection after initially choosing one — is previous data cleared?
- Browser back button behavior during SetupIntent confirmation
- Session timeout during multi-step checkout — graceful handling
- Concurrent tab submission (duplicate SetupIntent creation)
- Stripe Customer already exists for email (from previous abandoned checkout)
- Existing account has mismatched Stripe customer ID in Salesforce parent account vs Stripe metadata — confirm reconciliation rule

### Regression Areas

- Existing logged-in user checkout flow (must remain unchanged)
- Invoice payment method for accounts with `enqix_payment_method = "normal"` and order ≥ £1000
- Reorder flow for existing customers
- Direct Debit setup via recurring orders (`RecurringOrderService`) — existing flow must still work
- Anonymous cart → logged-in cart transition

### Test Data Requirements

- Test Stripe account with test mode enabled
- Salesforce sandbox with DD fields on Account
- Test account with no existing payment method
- Test account with existing DD setup
- Test account with existing saved card in Stripe

## 5. Risks & Concerns

### Security

- **CRITICAL — PCI DSS Compliance:** Raw card data (PAN, CVV, expiry) must NEVER be transmitted to or stored on our servers. Implementation MUST use Stripe Elements (client-side tokenization). The ticket's wording "capture all card details" could be misinterpreted as server-side storage — this would violate PCI DSS and could result in fines of $5,000–$100,000/month. Implementation must use SetupIntent + PaymentElement exclusively.
- **Sensitive Financial Data (DD):** Bank account numbers and sort codes are sensitive. While not PCI-scoped, they must be: (1) transmitted over TLS only, (2) validated/sanitized server-side, (3) not logged or exposed in error messages, (4) stored with appropriate access controls in Salesforce.
- **Input Validation:** DD account number and sort code must be strictly validated (numeric only, correct length) to prevent injection attacks against Salesforce API.
- **CSRF Protection:** New POST endpoints (`SetupIntentController`, `PaymentMethodController`) must have anti-forgery token validation.

### Compliance

- **BACS DD Mandate:** Setting `Direct_Debit_authorised__c = TRUE` requires the customer to have explicitly agreed to a Direct Debit mandate. This typically requires specific legal text and an explicit consent checkbox. Failure to collect proper authorization could result in rejected mandates and regulatory issues.
- **GDPR:** Card tokenization via Stripe means no PII card data stored on our side. DD details (name, account number, sort code) stored in Salesforce — confirm Salesforce DPA covers this.
- **Strong Customer Authentication (SCA):** Stripe handles SCA automatically via PaymentElement/SetupIntent — no additional implementation needed.

### Performance

- SetupIntent creation adds one additional Stripe API call per new customer checkout — acceptable latency (~200-500ms).
- No N+1 queries introduced. Order workflow already handles account/contact creation sequentially.
- Bi-directional ID sync adds one additional Stripe customer update call and one Salesforce parent-account update path; both should be idempotent to avoid duplicate writes on retries.
- Session cart size will increase slightly with payment method selection data — negligible impact.

### Breaking Changes

- **No breaking changes to existing API contracts.** New endpoints are additive.
- Existing checkout flow for logged-in users is unchanged.
- `OrderCompleteWorkflow` changes are additive but now affect all account-level sync paths where Stripe and Salesforce IDs must be kept aligned.
- Frontend changes add new components — existing Payment component modifications are backward-compatible (conditional on payment method selection being present in props).
