# Open Questions

This book documents implemented behavior, but several important areas still need explicit verification before they can be described as settled architecture.

## Areas still requiring deeper confirmation

- database migration strategy and whether any migrations live outside this repository
- persistence model for carts, saved baskets, and checkout state
- exact controller routes, request contracts, and authorization filters across `firstmile.web/Api`
- failure handling and retry behavior beyond the HTTP client registration level for Salesforce and KeyCloak integrations
- Stripe payment lifecycle details, including webhook handling if any
- which frontend widgets map to which backend pages and APIs
- operational approval and rollback procedure details for preproduction and production

## Documentation rule

If a later chapter cannot verify one of these areas from code, configuration, or a reviewed runbook, keep the uncertainty visible instead of filling the gap with assumptions.
