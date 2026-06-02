# <Project> — Architecture Decision Log

When a constraint or rule is added to AGENTS.md, log the rationale here.

## Format

```
### DECISION-NNN: <title>
Date: YYYY-MM-DD
Status: active | superseded by DECISION-NNN
Context: <what prompted this>
Decision: <what was decided>
Rationale: <why — tradeoffs>
Consequences: <what this constrains or enables>
```

<!-- Add decisions below as they are made -->

### DECISION-001: labelsAndTexts new fields are optional with fallbacks

Date: 2026-06-01
Status: active
Context: FMI-916 added six new string fields to `LocationHomeModel.labelsAndTexts` for the restructured section headings. Existing CMS pages that have not been updated do not include these fields in their JSON payload.
Decision: Declare all six new fields as optional (`?`) in the TypeScript interface and destructure them with named defaults at the usage site in `LocationHome.tsx`.
Rationale: Making fields required would crash any deployed location-home page until every CMS instance is updated. Optional + fallback lets the FE ship and serve safe defaults until CMS editors publish the new values.
Consequences: All future `labelsAndTexts` additions to location-home must follow the same optional-with-fallback pattern until the CMS publish cycle is guaranteed to precede the FE deploy.

### DECISION-002: ad-hoc services section uses rc-collapse Collapse, not a custom accordion

Date: 2026-06-01
Status: active
Context: FMI-916 redesigned the "Other services" section into a bordered accordion. The codebase already ships `rc-collapse` via the shared `Collapse` component.
Decision: Use the existing `Collapse` component with a JSX `renderCollapseItemHeader` for the icon/title/description header, rather than building a new custom accordion.
Rationale: Reuses the established pattern; keeps animation and keyboard behavior consistent with other accordions in the portal.
Consequences: The collapse header must be a React element, not a plain string — `getLocationCollapseItems` accepts `ReactElement` for `label`.

### DECISION-003: one-off removals rendered as a standalone CTA button, not a collapsible section

Date: 2026-06-01
Status: active
Context: FMI-916 changed one-off rubbish removals from a dropdown/collapsible to a direct action. The existing order-flow popup is preserved.
Decision: Render `renderOneOffWasteRemovalServicesButton()` below the collapse — a plain `<button>` that dispatches `orderOneOffRemovalServices`. No separate table or accordion entry.
Rationale: Ticket explicitly calls for the existing popup to remain; only the entry point changes. A button is the simplest correct implementation.
Consequences: `getLocationHomeRemovalTableData` was deleted from `GetLocationTableData.tsx` — do not re-add it. The `orderOneOffRemovalServices` command requires BE recognition (`ButtonCommandType`) and must ship simultaneously with the BE PR.
