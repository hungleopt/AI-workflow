# Location Home

Widget: `firstmile.widgets/src/blocks/location-home/LocationHome.tsx`
SCSS: `firstmile.ui/source/_patterns/organisms/location-home/location-home.scss`
Last updated: 2026-06-01 (FMI-916)

---

## Public interface — `LocationHomeModel`

Extends `LocationPagesLayoutModel` (provides `locationList`, `navMenu`, `initialData`, `getPartnershipLogoByLocationIdApiUrl`, `timestamp`).

Key own fields:

| Field                             | Type                            | Notes                                                                           |
| --------------------------------- | ------------------------------- | ------------------------------------------------------------------------------- |
| `accountId`                       | `string`                        | Required. Salesforce account ID.                                                |
| `setupButton`                     | `ButtonModel?`                  | CTA shown when no scheduled services exist.                                     |
| `orderButton`                     | `ButtonModel?`                  | CTA inside the ad-hoc services accordion.                                       |
| `scheduleTable`                   | `TableModel<ScheduleRowModel>?` | Scheduled-services table.                                                       |
| `statusTable`                     | `TableModel<StatusRowModel>?`   | Delivery-status table.                                                          |
| `shreddingTable`                  | `TableModel<ProductRowModel>?`  | Ad-hoc (shredding/other) products table.                                        |
| `scheduleMap`                     | `LocationMapModel?`             | Map shown next to schedule table.                                               |
| `scheduledServicesSectionSetup`   | object?                         | Config for "Order more sacks" and "Manage scheduled services" CTAs.             |
| `labelsAndTexts`                  | see below                       | Merged type: `ScheduledServicesManagementModel['labelsAndTexts']` + own fields. |
| `getServiceOrProductOrderFlowUrl` | `string?`                       | API URL for fetching order-flow data.                                           |
| `miniCartSetup`                   | `MiniCartSetup?`                | Props forwarded to `MiniCart` on checkout.                                      |

### `labelsAndTexts` own fields (all optional — FMI-916)

```ts
scheduleServiceTitle?: string;        // fallback: 'Scheduled Services'
otherServicesTitle?: string;          // fallback: 'Other waste and recycling services'
adHocServicesTitle?: string;          // fallback: 'Manage your ad hoc services'
adHocServicesDescription?: string;    // fallback: ''
oneOffRemovalServicesTitle?: string;  // fallback: 'One-Off waste removal services'
oneOffRemovalServiceDescription?: string; // fallback: ''
```

All six are optional so existing CMS pages without those fields do not crash (see DECISION-001).

---

## Page layout (FMI-916)

Three sections rendered inside `.fm-o-location-home__inner`:

1. **Scheduled Services** — `renderScheduledServicesSection()`
   - Static heading with calendar icon + `scheduleServiceTitle`
   - `scheduleTable` + `statusTable` via `renderDataTableAndMap()`
   - "Order more sacks" and "Manage scheduled services" CTAs when data present

2. **Other waste and recycling services** — `.fm-o-location-home__other-services-section`
   - Heading with truck icon + `otherServicesTitle`
   - `rc-collapse` `Collapse` component (open on desktop, closed on mobile)
   - Header built by `renderCollapseItemHeader()` using `adHocServicesTitle` / `adHocServicesDescription`
   - Content is `shreddingTable` + optional `orderButton`
   - One-off removal standalone CTA below collapse: `renderOneOffWasteRemovalServicesButton()`

3. **Navigation** — `NavigationMenu` at top of `.fm-o-location-home__inner`
   - `styleModifier={navMenu.length > 3 ? ['more-than-3-items'] : []}` (FMI-916: was `> 4`)

---

## Key helpers / imports

| Symbol                              | From                                    |
| ----------------------------------- | --------------------------------------- |
| `getLocationHomeShreddingTableData` | `@src/helpers/GetLocationTableData`     |
| `getLocationHomeScheduleTableData`  | `@src/helpers/GetLocationTableData`     |
| `getLocationHomeStatusTableData`    | `@src/helpers/GetLocationTableData`     |
| `getLocationCollapseItems`          | `@src/helpers/location-dropdown-helper` |
| `renderDataTableAndMap`             | `@src/helpers/location-dropdown-helper` |
| `ScheduledServicesManagement`       | `./ScheduledServicesManagement`         |
| `LocationHomeContext`               | `@src/app/location-home-context`        |

`getLocationHomeRemovalTableData` was **removed** in FMI-916 — one-off removals are now a button, not a table.

---

## Order actions

| Action           | `ButtonCommandType`          | `ProductsOrServicesIds`    |
| ---------------- | ---------------------------- | -------------------------- |
| Order more sacks | `orderProductsV2`            | `AnyProductsOrServices`    |
| One-off removal  | `orderOneOffRemovalServices` | `AnyOneOffRemovalServices` |
| Add service      | `addService`                 | n/a                        |

`orderOneOffRemovalServices` requires a matching BE `ButtonCommandType` — must be deployed simultaneously.

---

## State / context

Uses `locationHomeContextReducer` / `LocationHomeContext`. Key state slices: `scheduledServicesDataTable`, `locationId`, `apiCallInProgress`, `scheduledServicesManagement`, `canOrderMoreSacks`, `canAddMoreServices`, `partnershipLogo`.

Location switch triggers `loadDataByLocationAsync(id)` which refreshes all table data via `getLocationAsync`.
