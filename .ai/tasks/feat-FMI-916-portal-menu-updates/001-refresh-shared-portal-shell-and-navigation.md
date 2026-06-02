# 001 - BE - Refresh shared portal shell and navigation

## TASK

Refresh the shared logged-in portal shell and navigation presentation across account pages.
Classification: STANDARD

## PATTERN

none

## CONTEXT

- files:
  - `FirstMile.Models/Pages/HomePage.cs`
  - `FirstMile.Services/Helpers/NavigationHelper.cs`
  - `firstmile.web/Features/AccountLocationPage/AccountLocationPageController.cs`
  - `firstmile.web/Features/AccountOrderHistoryPage/AccountOrderHistoryPageController.cs`
  - `firstmile.web/Features/AccountCaseListPage/AccountCaseListPageController.cs`
  - `firstmile.web/Features/AccountCaseDetailPage/AccountCaseDetailPageController.cs`
  - `firstmile.web/Features/AccountReportIssuePage/AccountReportIssuePageController.cs`
  - `firstmile.web/Features/AccountDocumentsPage/AccountDocumentsPageController.cs`
  - `firstmile.ui/source/_patterns/organisms/navigation/location-nav-menu.scss`
  - `firstmile.ui/source/_patterns/templates/location-home.hbs`
  - `firstmile.ui/source/_patterns/templates/report-an-issue.hbs`
  - `firstmile.ui/source/_patterns/templates/reporting-home.hbs`
  - `firstmile.ui/source/_patterns/templates/invoices.hbs`
- docs:
  - `docs/src/tickets/FMI-916-review.md`
  - `docs/src/backend/rendering-and-navigation-flow.md`

## GOAL

All logged-in portal pages share the new gradient shell (desktop only — mobile stays white) and the shared navigation matches the requested desktop presentation. Mobile navigation is descoped: the mobile dropdown and its layout remain unchanged; only the desktop active-state underline/bold/separator changes apply.

## STEPS

1. Grep the shared portal templates and account-page controllers to confirm which pages already use `main.fm-t-account` plus `NavigationHelper.NavMenuProcessing(...)` before editing styles.
2. Update the shared portal shell styling at the template / shared-style layer so all logged-in portal pages use the requested top-to-bottom gradient **on desktop only**. Ensure the mobile white background is explicitly preserved and not changed.
3. Update `location-nav-menu.scss` so **desktop** supports up to six items on one row, fewer than four items center correctly, separators are removed, active items are bold with a green underline, and hover remains green.
4. **Do not modify mobile navigation dropdown layout or behavior.** Mobile navigation is explicitly descoped per Caiti Black (2026-05-27 comment).
5. Verify each account-page controller still feeds the shared nav model and does not require page-specific menu markup changes beyond shared shell hooks.
6. Refresh pattern-lab / sample template data if the shared nav examples need new labels or menu counts to exercise the new styling.
7. Add or update controller/helper tests that protect shared navigation payload assumptions, then perform a visual validation pass on the affected portal pages.

## DONE WHEN

- [ ] Logged-in portal pages use the new gradient shell through shared template/styling hooks on **desktop only**. Mobile background remains white.
- [ ] Desktop shared nav shows up to six items on one row without separators.
- [ ] Desktop shared nav centers correctly when fewer than four items are present.
- [ ] Active shared-nav items show bold text and the requested underline treatment on desktop.
- [ ] Mobile navigation dropdown layout and behavior are **not modified** (explicitly descoped).
- [ ] Compiles without errors.
- [ ] Unit tests pass (per testing-policy.md).
- [ ] No files outside CONTEXT modified.
- [ ] All PATTERN steps completed or marked N/A.
- [ ] No claim made about existing code without citing file:line.
- [ ] Skill files loaded: N/A - no relevant module skill files exist under `.ai/skills/` for these slices.
- [ ] If interface changed: skill file for affected module updated or rewritten, or N/A documented.
- [ ] Standards validated: all applicable gates in `.ai/standards/definition-of-done.md` checked.
- [ ] `001-refresh-shared-portal-shell-and-navigation.qa.md` generated with accurate affected features and risk level.
- [ ] Executor verified: QA IMPACT matches actual changes made.

## DOC UPDATE

- `docs/src/backend/rendering-and-navigation-flow.md` - document the shared portal shell hook and revised shared navigation presentation rules.
- `docs/src/frontend/location-home.md` - document the location-home frontend block: model interface, section layout, key helpers, order actions, and state/context.

## COMMIT

feat(portal-ui): refresh shared shell and navigation

- apply the new logged-in portal gradient and shared navigation presentation across account pages
- keep navigation generation centralized in the existing helper/controller pipeline

Breaking: none
Migration: none
