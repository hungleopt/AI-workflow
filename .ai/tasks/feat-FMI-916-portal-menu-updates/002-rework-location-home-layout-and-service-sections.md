# 002 - FE - Rework location home layout and service sections

## TASK

Rework the location-home page layout and service sections for the new desktop/mobile portal design.
Classification: STANDARD

## PATTERN

none

## CONTEXT

- files:
  - `FirstMile.Models/Pages/AccountLocationPage.cs`
  - `firstmile.web/Features/AccountLocationPage/AccountLocationPageController.cs`
  - `firstmile.ui/source/_patterns/organisms/location-home/location-home.scss`
  - `firstmile.ui/source/_patterns/organisms/location-home/location-home.hbs`
  - `firstmile.ui/source/_patterns/templates/location-home.hbs`
  - `firstmile.ui/source/_data/location-home.json`
  - `firstmile.ui/source/_patterns/pages/location-home~mixed.json`
- docs:
  - `docs/src/tickets/FMI-916-review.md`
  - `docs/src/backend/feature-flows.md`

## GOAL

The location-home page matches the requested desktop/mobile layout, updates service-section labels and headings, and presents the existing "Other services" section as the new "Manage your ad-hoc services" box (confirmed by Caiti Black, 2026-05-29: this is a restyling/relabelling only, not a new feature) while reusing existing ordering flows.

## STEPS

1. Grep the current location-home controller and SCSS to confirm which labels, section groupings, and responsive behaviors are already driven by `AccountLocationPage` editor fields.
2. Update the location-home controller payload so scheduled services, ad-hoc services, one-off removals, and recurring-order entry points can be rendered in the new structure without introducing a new service API.
3. Extend `AccountLocationPage` editor fields where needed for revised section titles, descriptions, icon hooks, or empty-state copy.
4. Rework the location-home SCSS and markup so **desktop** moves the location display higher into the white container and keeps partnership-logo spacing intact. **The mobile location block layout is unchanged** (explicitly descoped by Caiti Black, 2026-05-27 comment: _"there is no change to the current layout of location on mobile"_).
5. Rename the relevant headings, remove the old visual treatment requested by the ticket, and add icon support for the service section headers.
6. Restyle the existing "Other services" section as "Manage your ad-hoc services": wrap it in the new box container, rename the heading, apply the sack icon to all rows including "Order another service", and make it open by default on desktop. No new service API or data source is needed.
7. Update mobile behavior so the relevant service sections are collapsed by default to reduce scroll and expand into the existing content when tapped.
8. Change the one-off rubbish removal presentation to a button-like CTA that still opens the existing popup / order flow.
9. Refresh location-home pattern data and add controller tests that protect the new payload shape and default-state assumptions.

## DONE WHEN

- [ ] Location-home **desktop** layout places the location display inside the raised white container. Mobile location block layout is **not modified** (explicitly descoped).
- [ ] Partnership-logo spacing remains intact after the layout change.
- [ ] Scheduled / other / one-off headings reflect the new naming and visual treatment.
- [ ] The existing "Other services" section is restyled as "Manage your ad-hoc services" (box, new heading, sack icon on all rows) using the existing data — no new service API introduced. Section is open by default on desktop.
- [ ] Mobile service sections collapse/expand as required by the ticket.
- [ ] One-off rubbish removals trigger the existing popup flow from the new CTA presentation.
- [ ] Compiles without errors.
- [ ] Unit tests pass (per testing-policy.md).
- [ ] No files outside CONTEXT modified.
- [ ] All PATTERN steps completed or marked N/A.
- [ ] No claim made about existing code without citing file:line.
- [ ] Skill files loaded: N/A - no relevant module skill files exist under `.ai/skills/` for this slice.
- [ ] If interface changed: skill file for affected module updated or rewritten, or N/A documented.
- [ ] Standards validated: all applicable gates in `.ai/standards/definition-of-done.md` checked.
- [ ] `002-rework-location-home-layout-and-service-sections.qa.md` generated with accurate affected features and risk level.
- [ ] Executor verified: QA IMPACT matches actual changes made.

## DOC UPDATE

- `docs/src/backend/feature-flows.md` - expand the location-home / portal-services description to reflect the revised section structure and reused ordering actions.
- `docs/src/frontend/location-home.md` - document the location-home frontend block: model interface, section layout, key helpers, order actions, and state/context.

## COMMIT

feat(location-home): rework portal location layout

- reshape the location-home payload and styling to match the new desktop/mobile service-section design
- reuse existing ad-hoc and one-off order flows under the new UI structure

Breaking: none
Migration: none
