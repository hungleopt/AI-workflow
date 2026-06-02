# Controller Patterns

This chapter captures the current controller patterns confirmed from representative controllers in `firstmile.web/Api`.

## Pattern 1: Thin base class, rich derived controllers

`BaseApiController` only applies common routing and API metadata. Real behavior is implemented directly in each controller, so cross-cutting consistency depends on convention rather than a heavy shared base class.

## Pattern 2: Controllers often shape frontend-ready payloads

The API layer does not always expose raw domain objects.

Examples:

- `OrderHistoryController` returns portal-friendly strings like formatted currency and long-form dates
- `RecurringOrderController` returns button metadata, popup structures, and UI step definitions
- `AccountController` returns status objects and redirect URLs tailored to frontend flow control

This matters when documenting the API because the response format is often a presentation contract, not only a domain contract.

## Pattern 3: Business logic leakage into controllers

`CartController` explicitly contains a TODO stating that cart manipulation should be refactored into `CartService`. The controller currently:

- merges items into existing cart state
- carries cart-type-specific behavior
- recalculates quantity state and VAT context
- chooses between full cart payloads and mini-cart payloads
- conditionally reapplies promo codes

This means the current behavior cannot be documented accurately by reading services alone.

## Pattern 4: Access checks are repeated inside feature controllers

Even when a controller has `SalesforceAuthorize`, methods still often do additional validation, such as:

- ensuring an account exists
- ensuring a location is selected
- verifying the current user can access a location before loading data

## Pattern 5: Session and current-user context are first-class inputs

Controllers commonly work through:

- the session cart
- selected location state
- current account context
- cached or synchronized Salesforce contact data

The API is therefore stateful from the perspective of the signed-in portal session.

## Documentation guidance

For controller-level documentation, capture these fields for each endpoint:

- explicit route
- required account or location context
- whether it returns domain data or frontend-ready UI data
- dependent services
- session or current-user state assumptions
