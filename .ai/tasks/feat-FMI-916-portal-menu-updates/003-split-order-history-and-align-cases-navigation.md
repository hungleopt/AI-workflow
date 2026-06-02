# 003 - BE - Split order history and align cases navigation

## TASK

Split order history and recurring orders into two portal pages and align support navigation with “Cases” terminology.
Classification: STANDARD

## PATTERN

none

## CONTEXT

- files:
  - `FirstMile.Models/Pages/HomePage.cs`
  - `FirstMile.Models/Pages/AccountOrderHistoryPage.cs`
  - `FirstMile.Models/Pages/AccountCaseListPage.cs`
  - `FirstMile.Models/Pages/AccountReportIssuePage.cs`
  - `FirstMile.Models/Pages/AccountLocationPage.cs`
  - `firstmile.web/Features/AccountOrderHistoryPage/AccountOrderHistoryPageController.cs`
  - `firstmile.web/Features/AccountCaseListPage/AccountCaseListPageController.cs`
  - `firstmile.web/Features/AccountReportIssuePage/AccountReportIssuePageController.cs`
  - `firstmile.web/Features/AccountLocationPage/AccountLocationPageController.cs`
  - `firstmile.web/Features/AccountRecurringOrdersPage/AccountRecurringOrdersPageController.cs`
  - `firstmile.ui/source/_data/location-home.json`
  - `firstmile.ui/source/_data/location-docs.json`
  - `firstmile.ui/source/_data/report-an-issue.json`
- docs:
  - `docs/src/tickets/FMI-916-review.md`
  - `docs/src/backend/feature-flows.md`
  - `docs/src/backend/api-route-reference.md`

## GOAL

Portal users can navigate to order history and recurring orders as separate pages, the order-history page no longer shows the old top tab switcher, and support navigation uses the product-approved “Cases” wording without breaking case-creation flows.

## STEPS

1. Grep the current order-history controller for `RecurringOrdersTabName`, `tab` query handling, and any links that still build `?tab=recurring` destinations.
2. Create the dedicated recurring-orders page/controller that editors can place in `HomePage.LocationMenu`.
3. Remove the old top-tab UX from the order-history page payload and leave that page responsible only for order-history behavior.
4. Initialize the new recurring-orders page with the recurring-orders payload that is currently embedded inside the order-history experience.
5. Update any location-home setup-recurring links so they point at the new recurring-orders destination instead of appending `?tab=recurring`.
6. Align shared navigation, CTA text, and relevant page defaults so “Cases” replaces “Report an issue” where the product expects the rename, while preserving the existing case-reporting endpoints and case-list/report-page relationship.
7. Refresh pattern/sample data and add controller tests for the separated order-history / recurring-orders payloads plus cases terminology.

## DONE WHEN

- [ ] Order history and recurring orders are separate portal pages.
- [ ] The order-history page no longer emits the old top-tab switcher UX.
- [ ] Any setup-recurring link points to the new recurring-orders destination instead of `?tab=recurring`.
- [ ] Shared portal navigation and relevant CTAs use “Cases” where approved by product.
- [ ] Case creation / report-issue flows still work after the terminology update.
- [ ] Compiles without errors.
- [ ] Unit tests pass (per testing-policy.md).
- [ ] No files outside CONTEXT modified.
- [ ] All PATTERN steps completed or marked N/A.
- [ ] No claim made about existing code without citing file:line.
- [ ] Skill files loaded: N/A - no relevant module skill files exist under `.ai/skills/` for this slice.
- [ ] If interface changed: skill file for affected module updated or rewritten, or N/A documented.
- [ ] Standards validated: all applicable gates in `.ai/standards/definition-of-done.md` checked.
- [ ] `003-split-order-history-and-align-cases-navigation.qa.md` generated with accurate affected features and risk level.
- [ ] Executor verified: QA IMPACT matches actual changes made.

## DOC UPDATE

- `docs/src/backend/feature-flows.md` - document the split between order history and recurring orders and the updated support/cases navigation.
- `docs/src/backend/api-route-reference.md` - update notes if any supporting portal route descriptions need to reflect the new page destinations or preserved case/report APIs.
- `docs/src/frontend/location-home.md` - document the location-home frontend block: model interface, section layout, key helpers, order actions, and state/context.

## COMMIT

feat(portal-pages): split order history and recurring orders

- separate recurring orders from order history in the portal IA
- align support navigation wording with the approved “Cases” terminology while preserving existing case flows

Breaking: none
Migration: none
