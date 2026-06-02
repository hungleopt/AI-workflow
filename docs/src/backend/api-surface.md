# API Surface

The backend exposes a controller-based API from `firstmile.web/Api`. The folder organization shows the major functional slices currently implemented.

## Main controllers present in the repository

- `AccountController`
- `CartController`
- `CaseController`
- `DocumentsController`
- `InvoicesController`
- `LocationController`
- `OrderAgainController`
- `OrderHistoryController`
- `PaymentIntentController`
- `PriceController`
- `PromoCodeController`
- `RecurringOrderController`
- `ReportingHomeController`
- `ReportIssueController`
- `SavedBasketController`
- `ServiceController`

All API controllers inherit from `BaseApiController`, which applies:

- route prefix `/api/[controller]`
- `application/json` response production
- `ApiController` behavior

Several controllers also override this baseline with explicit absolute routes such as `/api/login`, `/api/order-history`, or `/api/get-recurring-order-popup`.

## Behavioral grouping

### Commerce and pricing

- cart management
- payment intent creation
- promo code processing
- recurring orders
- order history and order again flows
- pricing and service endpoints

### Account and support

- case reporting
- documents and invoices
- locations and reporting home

## What is verified in this pass

The controller inventory and domain groupings are verified from the repository structure, and representative controllers confirm several implementation patterns:

- cart and pricing behavior can include orchestration plus business logic inside controllers
- account endpoints blend authentication, user provisioning, email sending, and redirect decisions
- recurring-order endpoints can return frontend-ready UI payloads, not only domain data
- order-history endpoints enforce location access before querying data

Contract-level endpoint documentation still needs a full controller-by-controller pass.

## Recommended next step for this area

Expand this chapter by controller, with for each controller:

- route and action inventory
- service dependencies
- authentication/authorization requirements
- primary request and response models
- feature links back to corresponding pages or widgets
