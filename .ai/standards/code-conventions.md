# Code Conventions — V1.0

<!-- Fill: extend or replace these rules after repo-scan to match your stack and language -->

## TypeScript (if applicable)

- Strict mode always on. `any` only when unavoidable — must have inline comment explaining why.
- Prefer `unknown` over `any` for external or untyped data.
- Export types from a central shared types module — no scattered inline exports across modules.
- No `Promise.all` for operations with sequential dependencies — use sequential `await`.

<!-- Add or replace with language-specific type rules if not TypeScript -->

## Naming

| Artifact              | Convention                                        |
| --------------------- | ------------------------------------------------- |
| Files                 | kebab-case (`user.service.ts`, `auth.handler.go`) |
| Classes               | PascalCase                                        |
| Functions / variables | camelCase                                         |
| Constants             | UPPER_SNAKE_CASE                                  |
| DB columns            | snake_case                                        |
| URL paths             | kebab-case                                        |

<!-- Fill: adjust naming conventions per language and framework after repo-scan -->

## Module structure

- Each module has a single entry point (`<module>.service.ts`, `index.ts`, or equivalent).
- No circular imports between modules.
- Domain logic in service layer — not in controllers, handlers, or background jobs.
- Background workers and job handlers dispatch to services — no business logic in job files.
- Shared module exports types and utils only — no business logic in shared.

## Error handling

| Context         | Rule                                                                                                                        |
| --------------- | --------------------------------------------------------------------------------------------------------------------------- |
| API handlers    | Return structured error `{ error, message, statusCode }`. Log full stack server-side. Never expose stack trace in response. |
| Background jobs | Log `{ jobId, error, stack }` on every failure. Never swallow errors.                                                       |
| General         | No naked `throw` at module boundary. No empty catch blocks.                                                                 |

<!-- Fill: add project-specific error handling patterns after repo-scan -->

## Logging

- Include a correlation ID (e.g. request ID, job ID, entity ID) on every log in request/job context.
- Structured logs (JSON) — no `console.log` or unstructured print in production code paths.
- Log levels: `info` for state transitions, `error` for failures, `debug` for verbose output.

## Comments

All code changes **must** include detailed comments that explain:

1. **WHY** the change exists — the business reason, constraint, or invariant being enforced.
2. **Ticket ID** — every non-trivial change must reference the relevant ticket (e.g., `// FMI-916: Added currency conversion for EUR support`).
3. **Context for future developers** — what would someone need to know 6 months from now to safely modify this code?

### Comment placement rules

| Change type              | Comment required                              | Example                                                                             |
| ------------------------ | --------------------------------------------- | ----------------------------------------------------------------------------------- |
| New class / service      | Class-level summary + ticket ID               | `/// <summary>Handles EUR currency conversion. See FMI-944.</summary>`              |
| New method               | Method-level summary + ticket ID              | `/// <summary>Validates Stripe webhook signature. FMI-920.</summary>`               |
| Modified method          | Inline comment at change point + ticket ID    | `// FMI-916: Changed from fixed rate to dynamic lookup`                             |
| Bug fix                  | Inline comment explaining the bug + ticket ID | `// FMI-901: Previously returned null when cart was empty, causing NRE in checkout` |
| Config / constant change | Inline comment with reason + ticket ID        | `// FMI-933: Increased timeout from 30s to 60s for slow Salesforce responses`       |
| Complex logic            | Block comment before the logic + ticket ID    | `// FMI-944: EUR orders use a different tax calculation path because...`            |

### What NOT to do

- Do not write comments that merely restate the code (`// increment counter`).
- Do not omit ticket IDs — they are mandatory for traceability.
- Do not use vague comments (`// fix`, `// updated`, `// changed this`).

## Abstraction limits

- Three similar lines is better than a premature abstraction.
- No helper extraction unless used in 3+ places.
- No feature flags or backwards-compat shims — change the code directly.

## Stack-specific rules

### C# — No anonymous types

**Do not use anonymous classes/objects** (`new { }`) in production code. Always create proper named model classes.

| Context               | Rule                                                                |
| --------------------- | ------------------------------------------------------------------- |
| API responses / DTOs  | Create a named class in the appropriate `Models/` or `Poco/` folder |
| LINQ projections      | Use a named record or class — not `select new { }`                  |
| View data / ViewBag   | Create a typed ViewModel class                                      |
| Test data builders    | Anonymous objects in tests are acceptable — production code is not  |
| Serialization targets | Must be a named class with explicit properties                      |

**Why:** Anonymous types cannot be reused, referenced by other code, documented, or unit-tested in isolation. They create maintenance debt and make refactoring fragile.

**When encountering existing anonymous types:** If modifying code that uses anonymous types, refactor them into named models as part of the change. Reference the ticket ID in the new model class comment.

**Placement:** Place new model classes in:
- `FirstMile.Models/Poco/` — for simple data transfer objects
- `FirstMile.Models/` — for domain models
- `{Project}/ViewModels/` or `{Project}/Models/` — for view-specific models
- Same namespace as the consuming service if tightly coupled and not shared

**Example — before (bad):**
```csharp
var result = new { Success = true, Message = "Order placed", OrderId = order.Id };
```

**Example — after (good):**
```csharp
/// <summary>Result of order placement. FMI-XXX.</summary>
public record OrderPlacementResult(bool Success, string Message, int OrderId);

var result = new OrderPlacementResult(true, "Order placed", order.Id);
```

<!-- Fill after repo-scan — add rules specific to this project's framework, ORM, queue system, etc. -->
<!-- Examples: -->
<!-- - No raw SQL string interpolation — parameterized queries or ORM always -->
<!-- - No direct DB writes from worker/job files -->
<!-- - No <framework-specific anti-pattern> -->
