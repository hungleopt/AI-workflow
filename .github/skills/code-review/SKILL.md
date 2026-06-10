---
name: code-review
description: 'Review code changes in the repository with high signal-to-noise analysis. USE FOR: reviewing staged/unstaged changes, branch diffs, specific files or methods, code quality, security vulnerabilities, logic errors, and convention compliance. DO NOT USE FOR: creating PRs, merging PRs, generating implementation tasks, or deploying code.'
argument-hint: 'Describe the scope of review: staged changes, branch name, file path, or paste a code snippet'
---

# Code Review

Perform an adversarial, high-signal code review of local changes or specific files — without requiring a Jira ticket or Azure DevOps PR. Surfaces bugs, security issues, logic errors, and convention violations. Ignores style, formatting, and trivial observations.

## AI Workflow Integration

This skill operates as **ARCHITECT** role within the `.ai/` workflow.

### Before starting:

1. Read `.ai/AGENTS.md` — follow golden rules and pre-coding read order.
2. **Always start from Step 1** (Gather Scope) — ask the user what to review. Never silently assume scope.
3. Load `.ai/standards/code-conventions.md` — all reviews must check against project conventions.
4. Load `.ai/standards/security.md` when the diff touches auth, billing, tenant isolation, or shared contracts.
5. Grep `docs/LESSONS.md` for the affected module/file names — flag if known failure patterns are being repeated.

### Review focus (high signal only):

Only raise findings that fall into these categories. Ignore everything else.

- **Bugs** — logic errors, incorrect conditions, wrong return values, silent swallows.
- **Security** — injection, AuthZ bypass, IDOR, information leakage, unvalidated input.
- **Convention violations** — anonymous C# types (`new {}`), missing ticket-ID comments, wrong test placement.
- **Requirement drift** — code that diverges from the documented intent or task file DONE WHEN conditions.
- **Blast radius** — callers of changed interfaces that were not updated.

Do **not** comment on: formatting, whitespace, naming preferences, redundant code that is not a bug, style choices.

---

## Step 1: Gather Scope

Ask the user for one of the following (choose the most specific applicable option):

1. **Staged/unstaged changes** — review `git diff` and/or `git diff --cached`.
2. **Branch diff** — review changes on a named branch vs. a base (e.g., `feat/FMI-123-slug` vs. `develop`).
3. **Specific files** — review one or more named files.
4. **Code snippet** — review pasted code directly.
5. **Mixed** — combination of the above.

Also ask for any context that is not in the code:
- The ticket ID if one exists (e.g., `FMI-123`).
- The intended behavior or requirement (if not captured in a task file).
- Any constraints or decisions made outside the codebase (verbal, Slack, etc.).

---

## Step 2: Obtain the Code

### 2a. Staged / unstaged changes

```powershell
# Staged changes
git diff --cached

# Unstaged changes
git diff

# Both
git diff HEAD
```

### 2b. Branch diff

```powershell
git fetch origin <baseBranch>:<baseBranch>
git diff <baseBranch>...<featureBranch> --stat
git diff <baseBranch>...<featureBranch>
```

Read full context of changed files when needed:

```powershell
git show <featureBranch>:<filePath>
```

### 2c. Specific files

Read each file in full from the repository:

```powershell
Get-Content <filePath>
```

### 2d. Task file cross-check

If a ticket ID was provided, check `.ai/tasks/` for a matching task file:

```powershell
Get-ChildItem -Recurse .ai/tasks/ | Where-Object { $_.Name -match "<ticketId>" }
```

Load the task file if found. The DONE WHEN conditions are a required part of the review checklist.

---

## Step 3: Adversarial Review

Adopt an **adversarial mindset**: your job is to break the code, not confirm it works. Assume every change contains at least one latent defect and hunt for it.

### Phase 1 — Convention & Ticket-ID Compliance

1. Verify every changed method/class includes a comment with the relevant ticket ID (e.g., `// FMI-123: reason`). Flag any changed code without one.
2. Check for anonymous C# types (`new {}`). Flag every occurrence — named types are required per golden rule 14.
3. Verify test files are placed in the correct `<Project>.Tests` project mirroring the source path. Flag any test placed in the wrong assembly.

### Phase 2 — Logic & Correctness

For every changed method or code path, ask:

- **Null / empty inputs**: What happens if any parameter, collection, or config value is `null`, empty, or whitespace?
- **Boundary values**: Zero, negative, max-int, empty arrays, single-element lists, strings at max length.
- **Concurrency**: Can two requests hit this path simultaneously and corrupt shared state?
- **Ordering**: Does this assume a specific call order that is not enforced?
- **External failures**: What if a service call, DB query, or file read throws? Is the exception silently swallowed?
- **Return values**: Is the return value or out/ref parameter correct for all branches, including error paths?

### Phase 3 — Blast Radius Analysis

1. Identify every changed **public interface** (method signatures, properties, types exported across modules).
2. Grep for callers of changed symbols across the codebase:
   ```powershell
   Select-String -Path "**/*.cs" -Pattern "<changedMethodOrTypeName>" -Recurse
   ```
3. Verify callers were updated or confirm they are unaffected by the change.
4. Check DI registrations, feature flag guards, and route registrations for consistency with the change.

### Phase 4 — Security Adversarial Pass

Apply only when the diff touches controllers, API endpoints, authentication, authorization, data access, or external integrations:

- **Injection**: Can user-controlled input reach SQL, HTML, or command execution without sanitization?
- **AuthZ/AuthN**: Does the change accidentally expose data or actions to unauthorized users?
- **IDOR**: Can a user manipulate IDs to access another tenant's or user's data?
- **Information leakage**: Do error messages, logs, or API responses expose internal stack traces or PII?
- **Salesforce field values**: Typos in status strings or field names cause silent failures — verify against known-good values.

### Phase 5 — DONE WHEN Verification (if task file exists)

- List every DONE WHEN condition from the task file.
- Mark each ✅ proven-met or ❌ not-met/unverifiable with `file:line` evidence.
- Untested logic for a DONE WHEN condition → treat as ❌.

---

## Step 4: Summarize Findings

Present the review as:

1. **Scope Summary**: What was reviewed (files, branches, diff size).
2. **Adversarial Findings**: For each finding, state:
   - The **attack vector** (how you tried to break it).
   - The **evidence** (`file:line` citation and reproduction scenario).
   - The **impact** (what goes wrong if this defect ships).
3. **Issues Table**: Categorized by severity:
   - **Critical / Bug** — will cause runtime errors, data loss, or security breach.
   - **High** — doesn't meet spec, risks data corruption, missing test coverage for critical logic.
   - **Medium** — potential issue under specific conditions; needs clarification or hardening.
   - **Low** — convention violation, missing ticket ID comment, wrong test placement.
4. **DONE WHEN Checklist** (if task file was loaded): Every condition marked ✅ or ❌ with evidence.
5. **Verdict**: Pass / Request changes / Needs clarification.

> **Verdict bias**: Default to "Request changes" unless you can prove correctness for all critical paths. The burden of proof is on the code.

---

## Step 5: Offer Fixes

For each identified issue:

1. Propose a concrete, minimal code fix.
2. Cite the exact `file:line` that needs changing.
3. Where a unit test would have caught the defect, propose the test using the `unit-tests` skill conventions.

Ask the user to confirm which fixes to apply before making any changes.

---

## Step 6: Apply Approved Changes

After the user confirms fixes:

1. Apply the approved changes to the affected source files.
2. If new tests are needed, create them following the `unit-tests` skill.
3. **Do NOT push automatically.** Remind the user to review all changes before pushing.

---

## Notes

- If anything is unclear about the intended behavior, **ask the user** — do not infer. Refer to `.ai/standards/stakeholders.md` for escalation guidance.
- When the diff is large, prioritize findings by severity and blast radius — lead with Critical/High.
- Always verify string literals, magic values, and status codes against the ticket or task file spec.
- Pay special attention to Salesforce API field values — typos in status strings or field names cause silent failures.
- If a lesson pattern from `docs/LESSONS.md` is being repeated, flag it explicitly by name.
