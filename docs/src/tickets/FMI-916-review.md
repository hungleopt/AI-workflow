# FMI-916: (June) Portal: Menu updates

> Last Reviewed: 2026-05-29 07:00 UTC  
> Status: In Progress  
> Type: Story

## 1. Questions, Assumptions & Decisions

### Open Questions

None at the moment. Product clarified the remaining scope questions on 2026-05-26.

### Assumptions

- The full Jira scope is in play. The extra user context narrows two priority areas, but it does not replace the rest of the ticket.
- The `Report an issue` to `Cases` rename applies to the shared menu label only; existing page titles, CTA copy, and CMS field labels keep their current wording unless a follow-up ticket changes them.
- The requested page split means order history and recurring orders should no longer share the current in-page tabs UX.
- The split should preserve the current recurring-orders behavior, but it must be delivered as a dedicated recurring-orders page rather than another mode on the order-history page.
- Desktop portal navigation will support no more than six menu items. Any menu expansion beyond six items is out of scope for FMI-916.
- Design will provide additional icons from Figma, and those assets will be added to the SVG sprite rather than sourced from the current icon set.
- Existing service-ordering actions, popups, and APIs should be reused where possible; this ticket is primarily presentation, information architecture, and page wiring rather than new backend ordering logic.
- The logged-in portal shell is the shared `main.fm-t-account` wrapper used by multiple account templates (location-home, report an issue, reporting home, invoices).

### Decisions

- Classify this ticket as `STANDARD`. It spans shared portal navigation, multiple account pages, and the location-home experience, but it does not introduce a new architecture pattern.
- Split implementation into three tasks: shared portal shell/navigation presentation, location-home layout and service-section changes, and route/content-model changes for order-history separation plus support-page naming.
- Treat the current shared navigation helper as the integration seam. `HomePage.LocationMenu` feeds `NavigationHelper.NavMenuProcessing(...)` at `FirstMile.Models/Pages/HomePage.cs:128` and `FirstMile.Services/Helpers/NavigationHelper.cs:17`, and that output is consumed by location, order history, documents, case list, case detail, and report issue controllers.
- **2026-05-29 — "Manage your ad-hoc services" is the existing "Other services" section** (confirmed by Caiti Black). No new data source or backend logic is needed; the implementation is a restyling and relabelling of the current "Other services" block, not a separate feature. The two additional screenshots added on 2026-05-27 (Screenshot 2026-05-27 at 16.34.24.png, Screenshot 2026-05-27 at 16.36.02.png) illustrate this revised treatment — review in Jira before UI work begins.
- **2026-05-27 — Three mobile items explicitly descoped by Caiti Black (comment):**
  - **Mobile navigation styling descoped.** Exact quote: _"given the surprising time this will take descope and leave navigation as is on mobile."_ Only desktop nav changes: remove separators, add active-state green underline and bold text. Mobile dropdown remains unchanged.
  - **Mobile location block layout unchanged.** Exact quote: _"there is no change to the current layout of location on mobile."_ Moving the location display into the white container is a desktop-only change.
  - **Mobile background gradient — no change.** Exact quote: _"there is no background on mobile."_ Mobile stays white; the gradient shell applies to desktop only.
- **2026-05-28 — Estimation approved.** Final agreed estimate: **49.5h total** (FE 23.5h + BE 17.5h + AA 2.5h + QA 6h). Ticket moved to `In Progress`; assigned to Cuong Nguyen Duc.

## 2. Proposed Implementation

### Approach

Keep the existing portal pattern where Optimizely page controllers assemble frontend JSON, but update three layers in parallel:

1. shared portal shell and navigation presentation
2. location-home content shape and responsive layout
3. account-page information architecture for order history / recurring orders / cases

The current implementation is split across page controllers plus frontend pattern styles:

- Shared nav items are built from `HomePage.LocationMenu` and filtered by `NavigationHelper.NavMenuProcessing(...)` at `FirstMile.Models/Pages/HomePage.cs:128` and `FirstMile.Services/Helpers/NavigationHelper.cs:17`.
- That nav payload is injected into multiple account pages, including location home at `firstmile.web/Features/AccountLocationPage/AccountLocationPageController.cs:142`, order history at `firstmile.web/Features/AccountOrderHistoryPage/AccountOrderHistoryPageController.cs:97`, case list at `firstmile.web/Features/AccountCaseListPage/AccountCaseListPageController.cs:86`, and report issue at `firstmile.web/Features/AccountReportIssuePage/AccountReportIssuePageController.cs:90`.
- The desktop/mobile nav styling still relies on light-weight text, separators, and no underline in `firstmile.ui/source/_patterns/organisms/navigation/location-nav-menu.scss:37`, `firstmile.ui/source/_patterns/organisms/navigation/location-nav-menu.scss:39`, and `firstmile.ui/source/_patterns/organisms/navigation/location-nav-menu.scss:70`.
- The location-home page already owns its own background/card behavior through `bgTwoColorPadding`, `bgTwoColor`, and a desktop-only white inner container at `firstmile.ui/source/_patterns/organisms/location-home/location-home.scss:8`, `firstmile.ui/source/_patterns/organisms/location-home/location-home.scss:11`, and `firstmile.ui/source/_patterns/organisms/location-home/location-home.scss:18`, so that page needs to be brought into line with the broader portal-shell change instead of layering another bespoke style override.
- Order history is still one page with an internal recurring-orders tab driven by `RecurringOrdersTabName`, query-string parsing, and a `Tabs` payload at `firstmile.web/Features/AccountOrderHistoryPage/AccountOrderHistoryPageController.cs:30`, `firstmile.web/Features/AccountOrderHistoryPage/AccountOrderHistoryPageController.cs:70`, `firstmile.web/Features/AccountOrderHistoryPage/AccountOrderHistoryPageController.cs:71`, and `firstmile.web/Features/AccountOrderHistoryPage/AccountOrderHistoryPageController.cs:98`.

### Solution Details

1. **Refresh the shared portal shell and shared navigation styling**

   Introduce the new gradient background at the shared account-template level instead of only inside location-home. Keep `main.fm-t-account` as the outer hook and update the shared nav styles so desktop supports one row of up to six items, fewer than four items center correctly, active items show bold text plus underline, hover remains green, and vertical separators are removed. The existing `--center` modifier already exists in `firstmile.ui/source/_patterns/organisms/navigation/location-nav-menu.scss:4`, so the cleanest implementation is to extend that component rather than fork it per page.

1. **Rework location-home into the new desktop/mobile layout**

   The location-home controller currently exposes scheduled services, `OtherServicesTitle`, and `OnOffTitle` through one JSON payload at `firstmile.web/Features/AccountLocationPage/AccountLocationPageController.cs:173` and `firstmile.web/Features/AccountLocationPage/AccountLocationPageController.cs:205`, with editor-configurable labels defined on `FirstMile.Models/Pages/AccountLocationPage.cs:60`, `FirstMile.Models/Pages/AccountLocationPage.cs:63`, `FirstMile.Models/Pages/AccountLocationPage.cs:185`, and `FirstMile.Models/Pages/AccountLocationPage.cs:191`. Rework that payload and the matching `firstmile.ui` location-home pattern so desktop moves the location selector into the white container, scheduled/ad-hoc/one-off sections match the requested heading treatment, and mobile uses collapsed containers to reduce scroll.

1. **Restyle the existing "Other services" section as "Manage your ad-hoc services"** _(confirmed by Caiti Black, 2026-05-29)_

   The "Manage your ad-hoc services" box is not a new feature — it is the current "Other services" section given a new title, surrounding box, and revised collapsed/expanded behavior. No new service API or data source is required. The review sample data at `firstmile.ui/source/_patterns/pages/location-home~mixed.json:278` already includes the `Order another service` row. The implementation work is: rename the section heading, wrap it in the boxed container with default-open on desktop and collapsed on mobile, apply the sack icon to all rows including "Order another service", and match the design screenshots from Jira.

1. **Convert one-off rubbish removals from dropdown-like presentation to a direct CTA that opens the existing popup**

   The ticket explicitly calls for the existing popup to remain. That means the implementation should keep the current order-flow hook-up and change only the rendering contract for one-off removals so the row behaves like a button action rather than another collapsible section.

1. **Split order history and recurring orders into two pages**

   The current controller builds both experiences in one JSON model and toggles between them using the `tab` query string and internal tab state. To match the ticket, remove the top tab from order history and create a dedicated recurring-orders page that can be added independently to `HomePage.LocationMenu`, while keeping the current recurring-orders behavior intact. This also means revisiting any links that currently append `?tab=recurring`, including the setup recurring link generated on location home at `firstmile.web/Features/AccountLocationPage/AccountLocationPageController.cs:369`.

1. **Rename the shared support menu label to “Cases” without breaking case creation flows**

   The current report page still defaults its title to `Report an issue` at `FirstMile.Models/Pages/AccountReportIssuePage.cs:33`, and case-list page configuration still exposes a `ReportAnIssueLink` at `FirstMile.Models/Pages/AccountCaseListPage.cs:14` and `FirstMile.Models/Pages/AccountLocationPage.cs:38`. Preserve the existing case-creation/report APIs, but limit the wording change in FMI-916 to the shared menu label so page titles and existing support CTA copy remain unchanged.

## 3. Detailed Task List

### 3.1 Models & Configuration

| #   | File Path                                              | Action | Description                                                                                                |
| --- | ------------------------------------------------------ | ------ | ---------------------------------------------------------------------------------------------------------- |
| 1   | `FirstMile.Models/Pages/HomePage.cs`                   | Modify | Update `LocationMenu` usage assumptions and editor guidance for the revised portal navigation structure.   |
| 2   | `FirstMile.Models/Pages/AccountLocationPage.cs`        | Modify | Add or rename editor-facing fields needed for updated section headings, descriptions, and icon metadata.   |
| 3   | `FirstMile.Models/Pages/AccountOrderHistoryPage.cs`    | Modify | Remove or reduce configuration that only exists for the in-page recurring-orders tabs UX.                  |
| 4   | `FirstMile.Models/Pages/AccountCaseListPage.cs`        | Review | Confirm existing support-page links remain valid when only the shared menu label changes to `Cases`.       |
| 5   | `FirstMile.Models/Pages/AccountReportIssuePage.cs`     | Review | Confirm page title/default copy stays as `Report an issue` while the shared menu label changes to `Cases`. |
| 6   | `FirstMile.Models/Pages/AccountRecurringOrdersPage.cs` | Create | Introduce the dedicated recurring-orders page model required for the new portal page.                      |

### 3.2 Services & Business Logic

| #   | File Path                                        | Action | Description                                                                                                     |
| --- | ------------------------------------------------ | ------ | --------------------------------------------------------------------------------------------------------------- |
| 1   | `FirstMile.Services/Helpers/NavigationHelper.cs` | Modify | Preserve shared nav generation while supporting the revised menu labels / destinations without duplicate logic. |

### 3.3 Integration

| #   | File Path                            | Action | Description                                                                                     |
| --- | ------------------------------------ | ------ | ----------------------------------------------------------------------------------------------- |
| —   | No new external integration expected | —      | This ticket should reuse existing Salesforce-backed portal data and existing order / case APIs. |

### 3.4 Controllers & Endpoints

| #   | File Path                                                                                   | Action | Description                                                                                                                               |
| --- | ------------------------------------------------------------------------------------------- | ------ | ----------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | `firstmile.web/Features/AccountLocationPage/AccountLocationPageController.cs`               | Modify | Reshape the location-home payload for the new section labels, ad-hoc grouping, removal CTA presentation, and recurring-order link target. |
| 2   | `firstmile.web/Features/AccountOrderHistoryPage/AccountOrderHistoryPageController.cs`       | Modify | Remove top-tab UX from order history and keep only the order-history destination behavior.                                                |
| 3   | `firstmile.web/Features/AccountRecurringOrdersPage/AccountRecurringOrdersPageController.cs` | Create | Provide the dedicated recurring-orders page/controller required by the new portal IA.                                                     |
| 4   | `firstmile.web/Features/AccountCaseListPage/AccountCaseListPageController.cs`               | Review | Keep case-list behavior intact and confirm only the shared navigation label changes to `Cases`.                                           |
| 5   | `firstmile.web/Features/AccountReportIssuePage/AccountReportIssuePageController.cs`         | Review | Preserve report-issue creation flow and confirm page-level copy stays unchanged for FMI-916.                                              |
| 6   | `firstmile.web/Features/AccountDocumentsPage/AccountDocumentsPageController.cs`             | Review | Confirm documents page inherits the shared portal shell/navigation styling without bespoke regressions.                                   |
| 7   | `firstmile.web/Features/AccountCaseDetailPage/AccountCaseDetailPageController.cs`           | Review | Confirm case-detail page inherits the shared portal shell/navigation styling and updated terminology.                                     |

### 3.5 UI & Frontend

| #   | File Path                                                                   | Action | Description                                                                                                                        |
| --- | --------------------------------------------------------------------------- | ------ | ---------------------------------------------------------------------------------------------------------------------------------- |
| 1   | `firstmile.ui/source/_patterns/organisms/navigation/location-nav-menu.scss` | Modify | Implement the new shared desktop/mobile menu presentation: no separators, active underline, bold active text, and centering rules. |
| 2   | `firstmile.ui/source/_patterns/organisms/location-home/location-home.scss`  | Modify | Rework desktop/mobile location-home layout, section containers, spacing, and responsive collapse behavior.                         |
| 3   | `firstmile.ui/source/_patterns/organisms/location-home/location-home.hbs`   | Modify | Update location-home markup hooks if the new section structure needs different containers or ordering.                             |
| 4   | `firstmile.ui/source/_patterns/templates/location-home.hbs`                 | Modify | Ensure the location-home page participates in the shared portal shell updates.                                                     |
| 5   | `firstmile.ui/source/_patterns/templates/report-an-issue.hbs`               | Modify | Ensure support pages participate in the shared portal shell updates.                                                               |
| 6   | `firstmile.ui/source/_patterns/templates/reporting-home.hbs`                | Modify | Ensure other logged-in account pages participate in the shared portal shell updates.                                               |
| 7   | `firstmile.ui/source/_patterns/templates/invoices.hbs`                      | Modify | Ensure invoice/account templates participate in the shared portal shell updates.                                                   |
| 8   | `firstmile.ui/source/_data/location-home.json`                              | Modify | Refresh pattern-lab sample data to reflect new section titles, labels, and menu examples.                                          |
| 9   | `firstmile.ui/source/_data/location-docs.json`                              | Modify | Refresh shared nav sample labels so the relevant menu item reads `Cases`.                                                          |
| 10  | `firstmile.ui/source/_data/report-an-issue.json`                            | Review | Confirm support-page sample content stays on `Report an issue` while navigation examples reflect the menu-label change only.       |

### 3.6 Wiring & DI

| #   | File Path                                          | Action | Description                                                                          |
| --- | -------------------------------------------------- | ------ | ------------------------------------------------------------------------------------ |
| 1   | `firstmile.web/Infrastructure/Startup.Services.cs` | Review | Only needed if a new recurring-orders page/controller introduces new service wiring. |

### 3.7 Unit Tests

| #   | Test File Path                                                                                         | Tests to Add                                                                                                               | Covers                                                    |
| --- | ------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------- |
| 1   | `FirstMile.Services.Tests/Helpers/NavigationHelperTests.cs`                                            | `NavMenuProcessing_WithPortalPages_PreservesExpectedItems`, `NavMenuProcessing_WithCasesLabel_MapsCurrentPageActiveState`  | Shared nav generation and menu-label active-page behavior |
| 2   | `firstmile.web.Tests/Features/AccountLocationPage/AccountLocationPageControllerTests.cs`               | `Index_BuildsLocationHomePayload_WithUpdatedSectionLabels`, `Index_UsesDedicatedRecurringOrdersDestination_WhenConfigured` | Location-home JSON shape and recurring-order link wiring  |
| 3   | `firstmile.web.Tests/Features/AccountOrderHistoryPage/AccountOrderHistoryPageControllerTests.cs`       | `Index_OrderHistoryPage_DoesNotEmitRecurringTabs`, `Index_OrderHistoryPage_KeepsSharedNavigationPayload`                   | Order-history separation and shared shell payload         |
| 4   | `firstmile.web.Tests/Features/AccountRecurringOrdersPage/AccountRecurringOrdersPageControllerTests.cs` | `Index_RecurringOrdersPage_InitializesRecurringOrdersExperience`                                                           | Dedicated recurring-orders page                           |
| 5   | `firstmile.web.Tests/Features/AccountCaseListPage/AccountCaseListPageControllerTests.cs`               | `Index_CaseListPage_KeepsExistingPrimaryActions_WhenMenuLabelChangesToCases`                                               | Case-list regression coverage when only nav label changes |
| 6   | `firstmile.web.Tests/Features/AccountReportIssuePage/AccountReportIssuePageControllerTests.cs`         | `Index_ReportIssuePage_KeepsExistingCopy_WhenMenuLabelChangesToCases`                                                      | Support-flow regression while page copy stays unchanged   |

Test file location convention:

```text
Source:  FirstMile.Services/Helpers/NavigationHelper.cs
Test:    FirstMile.Services.Tests/Helpers/NavigationHelperTests.cs

Source:  firstmile.web/Features/AccountLocationPage/AccountLocationPageController.cs
Test:    firstmile.web.Tests/Features/AccountLocationPage/AccountLocationPageControllerTests.cs

Source:  firstmile.web/Features/AccountOrderHistoryPage/AccountOrderHistoryPageController.cs
Test:    firstmile.web.Tests/Features/AccountOrderHistoryPage/AccountOrderHistoryPageControllerTests.cs
```

### 3.8 Documentation

| #   | Doc File Path                                       | Action | Description                                                                                               |
| --- | --------------------------------------------------- | ------ | --------------------------------------------------------------------------------------------------------- |
| 1   | `docs/src/backend/rendering-and-navigation-flow.md` | Update | Document the revised logged-in portal shell and shared navigation behavior.                               |
| 2   | `docs/src/backend/feature-flows.md`                 | Update | Expand the account-portal section to reflect the order-history / recurring-orders split and cases naming. |
| 3   | `docs/src/tickets/FMI-916-review.md`                | Create | This review document.                                                                                     |

## 4. QA Verification Notes

### Test Scenarios

| #   | Scenario                     | Steps                                                                                                    | Expected Result                                                                                                                                 |
| --- | ---------------------------- | -------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | Shared portal gradient       | Open location home, report issue/cases, invoices, and reporting/account pages as a logged-in portal user | All logged-in portal pages use the new top-to-bottom gradient shell consistently.                                                               |
| 2   | Desktop nav presentation     | Load a portal page with 4 to 6 menu items on desktop                                                     | Items fit on one row, separators are removed, hover remains green, and the active item is bold with a green underline.                          |
| 3   | Short desktop nav centering  | Load a portal page with fewer than 4 menu items                                                          | The shared menu is centered rather than left-aligned.                                                                                           |
| 4   | Location-home desktop layout | Open location home on desktop with partnership logos present                                             | The location display sits inside the white container, spacing remains correct, and logos do not collide with the raised card.                   |
| 5   | Location-home mobile layout  | Open location home on mobile width                                                                       | Collapsible containers reduce scroll, and the requested default-open / default-closed behavior matches the ticket.                              |
| 6   | Ad-hoc services section      | Open location home with existing ad-hoc services and the “order another service” action available        | The new “Manage your ad-hoc services” box renders the existing services and action set with the requested title, copy, and icon treatment.      |
| 7   | One-off removals CTA         | Trigger the one-off rubbish removals action from the location page                                       | The existing popup opens from the new button-style row rather than from a dropdown-like section.                                                |
| 8   | Order history split          | Navigate to order history and recurring orders from the portal menu                                      | Order history and recurring orders are separate pages, and order history no longer shows the top tab switcher.                                  |
| 9   | Cases menu label             | Navigate through portal support pages                                                                    | The shared nav item uses `Cases` while page titles and existing support CTAs continue to use `Report an issue`, without breaking case creation. |

### Edge Cases to Verify

- Exactly six desktop nav items still fit cleanly without wrapping or clipping.
- The UI does not attempt to support a seventh portal menu item in FMI-916.
- RecycleID visibility rules still apply when `NavigationHelper.NavMenuProcessing(...)` filters portal items for non-recycle customers.
- Mobile dropdown navigation still behaves correctly when there is only one visible menu item.
- Locations with no ad-hoc services or no one-off removals still show sensible empty states and do not render broken boxes.
- Existing recurring-order deep links or setup buttons do not send the user back to a now-removed `?tab=recurring` experience.

### Regression Areas

- Shared portal navigation across location, documents, cases, case detail, report issue, and order-history pages
- Location switching within the portal
- Existing order / recurring-order APIs and popup flows
- Case creation and case-detail messaging
- Partnership-logo layout on location and other account pages

### Test Data Requirements

- Portal user with at least one live location
- Portal user with multiple locations and partnership logo enabled
- Portal user with ad-hoc services and one-off rubbish removal options available
- Portal user with order-history records and recurring orders enabled
- Portal user with existing support cases

## 5. Risks & Concerns

### Security

- None identified beyond normal portal access controls. The ticket changes presentation and page wiring, not auth logic, but all new destinations must continue to rely on the existing `[SalesforceAuthorize]` page protection pattern.

### Compliance

- None identified. No new PII flow or retention change is implied by the requested UI updates.

### Performance

- Moderate frontend regression risk if the new desktop nav tries to force six large items onto one row without responsive guardrails.
- The location-home page already has substantial interactive UI, so additional collapse wrappers and section icons should avoid unnecessary re-render or layout-thrashing behavior on mobile.

### Breaking Changes

- Splitting order history and recurring orders into separate pages can break existing links, CMS menu configuration, or any code still appending `?tab=recurring`.
- The shared-menu-only rename to `Cases` creates an intentional mixed-label state, so implementation and QA must ensure that limited scope is applied consistently rather than partially renaming additional support copy.
