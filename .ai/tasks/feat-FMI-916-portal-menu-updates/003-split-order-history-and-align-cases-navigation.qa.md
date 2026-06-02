# 003 - QA - Split order history and align cases navigation

Task: `.ai/tasks/feat-FMI-916-portal-menu-updates/003-split-order-history-and-align-cases-navigation.md`
Generated: 2026-05-26
Risk: medium - this changes portal destination structure and support-page terminology.

## Affected features

- Order history portal page
- Recurring orders portal destination
- Cases / support navigation and related CTAs

## Test scenarios

### Happy path

- [ ] Users can open order history and recurring orders as separate portal destinations.
- [ ] The order-history page no longer shows the old top tab switcher.
- [ ] Setup-recurring links from location-home land on the recurring-orders destination.
- [ ] Shared navigation and primary support links use “Cases” where expected.

### Edge cases

- [ ] Existing users with recurring orders enabled can still access and use recurring-orders actions after the split.
- [ ] Existing users without recurring orders still see a sensible recurring-orders page state.
- [ ] Support pages keep working even if only the navigation label changes while some backend field names remain `ReportAnIssue*`.

### Regression checks

- [ ] Order-history API calls still load and page correctly.
- [ ] Recurring-order popup and recurring-order list actions still function.
- [ ] Case list, case detail, and report-issue submission flows still work after the terminology update.

## Not affected (skip these)

- Shared portal shell / nav styling; that is covered by task 001.
- Location-home layout and ad-hoc service-box redesign; that is covered by task 002.

## Status

- [ ] Executor verified: QA IMPACT matches actual changes made.
- [ ] Tester sign-off
