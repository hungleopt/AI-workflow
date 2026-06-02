# 002 - QA - Rework location home layout and service sections

Task: `.ai/tasks/feat-FMI-916-portal-menu-updates/002-rework-location-home-layout-and-service-sections.md`
Generated: 2026-05-26
Risk: medium - this changes the main logged-in location page layout, responsive behavior, and service grouping.

## Affected features

- Portal location-home desktop layout
- Portal location-home mobile accordions / section collapse behavior
- Ad-hoc services and one-off removal presentation

## Test scenarios

### Happy path

- [ ] Desktop location-home shows the location display inside the white container with correct spacing.
- [ ] Updated headings and icons appear for scheduled services, ad-hoc services, and one-off removals.
- [ ] “Manage your ad-hoc services” renders existing ad-hoc services plus the order-another-service action in the new box.
- [ ] One-off rubbish removal opens the existing popup from the new button-like action.

### Edge cases

- [ ] Mobile location-home collapses sections by default and expands to reveal the current content without overlap.
- [ ] Locations with no ad-hoc services or no one-off removals still show sensible empty states.
- [ ] Partnership logos still align correctly when the location card is raised higher on the page.

### Regression checks

- [ ] Existing order-more-sacks and manage-scheduled-services actions still work.
- [ ] Existing order / order-again / book-collection actions still map to the correct flows.
- [ ] Location switching still refreshes the page data correctly after the layout change.

## Not affected (skip these)

- Shared portal navigation shell styling; that is covered by task 001.
- Separate recurring-orders destination and support-page naming updates; those are covered by task 003.

## Status

- [ ] Executor verified: QA IMPACT matches actual changes made.
- [ ] Tester sign-off
