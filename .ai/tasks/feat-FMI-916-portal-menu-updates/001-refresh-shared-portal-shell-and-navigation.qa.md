# 001 - QA - Refresh shared portal shell and navigation

Task: `.ai/tasks/feat-FMI-916-portal-menu-updates/001-refresh-shared-portal-shell-and-navigation.md`
Generated: 2026-05-26
Risk: medium - this changes shared shell and navigation styling across multiple logged-in portal pages.

## Affected features

- Shared logged-in portal page shell
- Shared portal navigation on location, order history, documents, cases, and support pages

## Test scenarios

### Happy path

- [ ] Logged-in portal pages show the requested gradient shell consistently.
- [ ] Desktop shared navigation renders up to six items on one row with no separators.
- [ ] Active shared navigation items show bold text and underline treatment.
- [ ] Hover behavior still turns navigation items green.

### Edge cases

- [ ] Portal pages with fewer than four visible nav items center the menu correctly.
- [ ] Portal pages with exactly six visible nav items do not clip or wrap unexpectedly.
- [ ] Mobile dropdown navigation still works when there is one item and when there are multiple items.

### Regression checks

- [ ] Documents, case detail, and report/support pages still receive the correct shared nav payload.
- [ ] RecycleID filtering rules in the shared navigation helper still behave correctly.
- [ ] Partnership-logo layouts are not visually broken by the shared shell change.

## Not affected (skip these)

- Location-home section restructuring and ad-hoc service-box behavior; those are covered by task 002.
- Order-history / recurring-orders destination split; that is covered by task 003.

## Status

- [ ] Executor verified: QA IMPACT matches actual changes made.
- [ ] Tester sign-off
