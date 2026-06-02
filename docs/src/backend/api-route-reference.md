# API Route Reference

This chapter inventories the current API routes from `firstmile.web/Api`. It is route-oriented rather than full contract documentation.

## Base routing rule

Controllers inheriting from `BaseApiController` default to `/api/[controller]`, but many actions override that with explicit absolute routes.

## Account and identity routes

| Route | Verb | Purpose |
| --- | --- | --- |
| `/api/login` | `POST` | portal login workflow |
| `/api/login-only-main-domain` | `POST` | login flow specific to the main-domain path |
| `/api/resend-password` | `POST` | resend or verify account/reset token flow |
| `/api/logout` | `GET` | sign out and clear portal state |
| `/api/reset-password` | `POST` | reset password flow |
| `/api/send-reset-password-email` | `POST` | trigger reset-password email |

## Cart and pricing routes

| Route | Verb | Purpose |
| --- | --- | --- |
| `/api/cart` | `GET` | get current cart |
| `/api/cart` | `POST` | add items to cart |
| `/api/cart/{id}` | `PATCH` | update a cart line item |
| `/api/cart/{id}` | `DELETE` | remove a cart line item |
| `/api/cart/update-by-postcode` | `PATCH` | reprice cart by postcode |
| `/api/cart/count` | `GET` | get cart item count |
| `/api/price` | `POST` | price a cart payload |
| `/api/paymentintent` | `POST` | create payment intent data |
| `/api/apply-promo-code` | `POST` | apply promo code |
| `/api/mini-cart-email-form` | `POST` | send mini-cart email flow |

## Orders and recurring-order routes

| Route | Verb | Purpose |
| --- | --- | --- |
| `/api/order-history` | `POST` | paged order history |
| `/api/order-items` | `POST` | order line items |
| `/api/order-again` | `POST` | order-again flow |
| `/api/get-recurring-order-popup` | `POST` | recurring-order modal payload |
| `/api/create-recurring-order` | `POST` | create recurring order |
| `/api/get-recurring-orders` | `POST` | list recurring orders |
| `/api/pause-or-cancel-recurring-order` | `POST` | pause or cancel recurring order |
| `/api/skip-recurring-order` | `POST` | skip next recurring order |
| `/api/resume-recurring-order` | `POST` | resume recurring order |

## Documents, invoices, and support routes

| Route | Verb | Purpose |
| --- | --- | --- |
| `/api/get-document-by-account` | `POST` | list account documents |
| `/api/sign-multi-documents` | `POST` | multi-document signing flow |
| `/api/documents/sign` | `POST` | sign a document |
| `/api/case-list` | `POST` | list support cases |
| `/api/case-detail` | `GET` | view support case detail |
| `/api/case-post-comment` | `POST` | post case comment |
| `/api/get-missed-collection` | `GET` | retrieve missed collection data |
| `/api/report-an-issue` | `POST` | create support/report issue request |

## Location and service routes

| Route | Verb | Purpose |
| --- | --- | --- |
| `/api/setup-service` | `POST` | setup service flow |
| `/api/update-employees-number` | `POST` | update employee count |
| `/api/update-location-address` | `POST` | update location address |
| `/api/location` | `GET` | location lookup |
| `/api/service/{serviceId}` | `GET` | service-specific data |
| `/api/get-services` | `GET` | service listing |

## Reporting routes

| Route | Verb | Purpose |
| --- | --- | --- |
| `/api/advanced-reporting` | `POST` | advanced reporting payload |
| `/api/get-reporting` | `POST` | reporting data retrieval |

## Documentation caution

This route list is verified from attributes in controller source files. It does not yet include per-action request models, response DTOs, or authorization rules, which still need a dedicated pass.

## Source anchors

- `firstmile.web/Api`
- `firstmile.web/Api/BaseApiController.cs`
