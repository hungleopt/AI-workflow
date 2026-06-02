# Documentation Tasks

This chapter tracks the documentation work needed to turn the current first-pass handbook into a fuller system reference.

## Completed in this implementation pass

- established mdBook structure under `docs/src`
- created overview, architecture, backend, frontend, development, operations, and reference sections
- migrated the legacy branching, DB/logs, and git-hooks notes into mdBook chapters
- documented runtime bootstrap, configuration, auth model, service registration, frontend integration pipeline, and testing reality
- validated the mdBook build locally

## Remaining implementation tasks

### Backend behavior tasks

1. Document every controller route and action, including request models and response shapes.
2. Trace the checkout flow end to end from page model to controller to service to Stripe.
3. Document account portal flows for invoices, documents, cases, locations, and reporting.
4. Map feature folders to the APIs and services they rely on.

### Architecture tasks

1. Inventory all startup extension methods beyond the files already covered.
2. Document the page and block inheritance hierarchy in `FirstMile.Models`.
3. Document how `IRequestContext` is populated and consumed.
4. Record more detail on Find, NotFoundHandler, and scheduled jobs.

### Frontend tasks

1. Document gulp tasks and Pattern Lab folder conventions in more detail.
2. Map React widget entry points to the backend pages that render them.
3. Document how generated widget and Pattern Lab assets move into `wwwroot` and `firstmile.patterns`.

### Operations tasks

1. Add environment-specific smoke-test and validation steps.
2. Document approval and rollback expectations for `prep` and `prod`.
3. Add a clear secrets-management policy and ownership map.
4. Expand troubleshooting for auth, Salesforce outage, and bad generated asset scenarios.

## Authoring rule for future work

Each new chapter should state:

- which source files were inspected
- what behavior is verified from code or configuration
- what remains inferred or still requires SME confirmation
