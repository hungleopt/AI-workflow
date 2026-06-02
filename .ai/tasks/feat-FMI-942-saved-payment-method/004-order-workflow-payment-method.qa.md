# 004 - QA - Order workflow payment method
Task: .ai/tasks/feat-FMI-942-saved-payment-method/004-order-workflow-payment-method.md
Generated: 2026-05-20
Risk: high — payment/billing logic, Salesforce data integrity, cross-system state

## Affected features
- New customer account creation in Salesforce
- Direct Debit setup on parent account
- Stripe Customer ID storage
- Payment method field on Salesforce Account
- Repeat order customer type update

## Test scenarios

### Happy path
- [ ] New customer with Direct Debit: Salesforce parent account has dd_account_holder__c, dd_account_number__c, dd_sort_code__c, Direct_Debit_authorised__c=true, Direct_Debit_Authorised_Date__c=today, enqix_payment_method="Direct Debit"
- [ ] New customer with Saved Card: Salesforce parent account has Stripe_ID__c, enqix_payment_method="Credit Card"
- [ ] New customer with Saved Card: Salesforce Contact has Stripe_Customer_ID__c
- [ ] Logged-in repeat order: Stripe Customer metadata has customer_type="Existing customer"
- [ ] Salesforce Parent Account ID sent to Stripe Customer metadata

### Edge cases
- [ ] Salesforce UpdateDirectDebit API failure — order still completes (logged, error email sent)
- [ ] Stripe UpdateCustomerMetadata failure — order still completes (non-blocking)
- [ ] No PaymentMethodSelection on cart (null) — existing flow unchanged, no crash
- [ ] PaymentMethodSelection.SelectedMethod = None — no payment method updates attempted

### Regression checks
- [ ] Existing new customer flow without payment method selection still creates account/location/contact
- [ ] Existing logged-in user reorder flow unchanged
- [ ] Existing Direct Debit setup via RecurringOrderService still works
- [ ] Bank statement creation still works for card payments
- [ ] Collection schedule creation still works

## Not affected (skip these)
- Frontend components
- SetupIntent API endpoint
- Cart persistence logic
- Promo code application
- Price calculation

## Status
- [ ] Executor verified: QA IMPACT matches actual changes made
- [ ] Tester sign-off
