---
name: pr-review
description: 'Review pull requests on Azure DevOps against requirements in Jira tickets. USE FOR: reviewing PRs, checking code changes against acceptance criteria, identifying potential issues, suggesting fixes and unit tests. DO NOT USE FOR: creating PRs, merging PRs, or deploying code.'
argument-hint: 'Provide the Azure DevOps PR URL and relevant Jira ticket URL(s)'
---

# Pull Request Review

Review Azure DevOps pull requests against Jira ticket requirements, identify issues, suggest fixes, and provide unit tests.

## AI Workflow Integration

This skill operates as **ARCHITECT** role within the `.ai/` workflow.

### Before starting:

1. Read `.ai/AGENTS.md` — follow golden rules.
2. **Always start from Step 1** (Gather Input) — ask the user for PR URL, ticket URL(s), and context. Never silently load existing review/task files or resume previous sessions.
3. After gathering input, load `.ai/standards/definition-of-done.md` — verify all hard gates pass.
4. Grep `docs/LESSONS.md` for the affected module/file names — flag if known failure patterns are being repeated.
5. Load `.ai/standards/stakeholders.md` — know who authored/owns what and who to escalate questions to.
6. Check `.ai/tasks/` for task files matching the PR branch — verify DONE WHEN conditions are met.

### Review checklist (in addition to §2.5 below):

- [ ] All DONE WHEN conditions in the task file are satisfied
- [ ] No claim about existing code without `file:line` citation
- [ ] Skill files updated if public interface changed (per `.ai/AGENTS.md` Skills/ maintenance)
- [ ] `.ai/standards/testing-policy.md` — required test types present
- [ ] `.ai/standards/security.md` — gates pass (if auth/billing/tenant touched)
- [ ] QA file (`.qa.md`) verified by executor (if QA mode is `task`)
- [ ] Adversarial pass completed — at least one fault-injection scenario attempted per changed method
- [ ] Every acceptance criterion mapped to a specific `file:line` or marked missing
- [ ] Blast radius analysis — all upstream callers of changed interfaces verified
- [ ] **Code comments with ticket ID** — verify all changed code includes comments explaining WHY + relevant ticket ID (e.g., `// FMI-XXX: reason`). Flag any uncommented changes.
- [ ] **Documentation up-to-date** — verify affected docs (skill files, ARCHITECTURE.md, DECISIONS.md, LESSONS.md) were updated to reflect the PR changes. Flag stale or missing doc updates.

## Prerequisite

Ensure the user has a `.env.local` file at the workspace root with the following values filled:

```dotenv
JIRA_PAT=
JIRA_EMAIL=
AZURE_DEVOPS_READONLY_PAT=
AZURE_DEVOPS_EMAILS=
```

**Before proceeding**:

1. Check if `.env.local` exists at the workspace root.
2. If it does **not** exist, create it with the following content (empty values for the user to fill):
   ```dotenv
   JIRA_PAT=
   JIRA_EMAIL=
   AZURE_DEVOPS_READONLY_PAT=
   AZURE_DEVOPS_EMAILS=
   ```
   Then tell the user to fill in the values after each `=` and re-run the skill.
3. If it **does** exist, read it and verify all four values are present and non-empty. If any value is missing or empty, ask the user to provide it before continuing.

## Step 1: Gather Input

Ask the user for:

1. The Azure DevOps pull request URL (e.g., `https://episerveremea-expertservices.visualstudio.com/First%20Mile/_git/FirstMile/pullrequest/8577`)
2. The relevant Jira ticket URL(s) (e.g., `https://episerver-services.atlassian.net/browse/FMI-934`)
3. Any additional information or context that is not captured in the ticket (e.g., verbal decisions, Slack discussions, architectural constraints, priority notes)

## Step 2: Review the Pull Request

### 2.1 Fetch PR Metadata

Use the `AZURE_DEVOPS_READONLY_PAT` and `AZURE_DEVOPS_EMAILS` to authenticate against the Azure DevOps REST API.

Parse the PR URL to extract:

- Organization (e.g., `episerveremea-expertservices`)
- Project (e.g., `First%20Mile`)
- Repository (e.g., `FirstMile`)
- PR ID (e.g., `8577`)

Fetch the PR details:

```powershell
$pat = "<AZURE_DEVOPS_READONLY_PAT>"
$b64 = [Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes(":$pat"))
$resp = Invoke-WebRequest -Uri "https://dev.azure.com/<org>/<project>/_apis/git/repositories/<repo>/pullrequests/<prId>?api-version=7.0" -Headers @{Authorization="Basic $b64"} -UseBasicParsing
$resp.Content
```

**Verify** the target branch is `develop`. If not, warn the user.

### 2.2 Sync Local Branches

1. Ensure local `develop` is up to date:

   ```powershell
   git fetch origin develop:develop
   ```

2. Fetch the source branch locally:

   ```powershell
   git fetch origin <sourceBranch>:<sourceBranch>
   ```

### 2.3 Get the Diff

Use `git diff` with the three-dot notation to see changes introduced by the source branch:

```powershell
git diff develop...<sourceBranch> --stat
git diff develop...<sourceBranch>
```

Read the full context of changed files on the feature branch when needed:

```powershell
git show <sourceBranch>:<filePath>
```

### 2.4 Fetch Jira Tickets

Use the `JIRA_PAT` and `JIRA_EMAIL` to authenticate against the Jira REST API.

Parse ticket keys from the provided URLs (e.g., `FMI-934` from `https://episerver-services.atlassian.net/browse/FMI-934`).

Fetch each ticket:

```powershell
$jiraPat = "<JIRA_PAT>"
$jiraAuth = [Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes("<JIRA_EMAIL>:$jiraPat"))
Invoke-WebRequest -Uri "https://episerver-services.atlassian.net/rest/api/3/issue/<TICKET_KEY>?fields=summary,description,status,issuetype,comment" -Headers @{Authorization="Basic $jiraAuth"} -UseBasicParsing | Select-Object -ExpandProperty Content
```

**Important**: Fetch and read ALL comments on the ticket. Comments have **higher priority** than the ticket description when determining requirements.

To fetch comments separately if needed:

```powershell
Invoke-WebRequest -Uri "https://episerver-services.atlassian.net/rest/api/3/issue/<TICKET_KEY>/comment" -Headers @{Authorization="Basic $jiraAuth"} -UseBasicParsing | Select-Object -ExpandProperty Content
```

### 2.5 Adversarial Review

Adopt an **adversarial mindset**: your job is to break the code, not confirm it works. Assume every change contains at least one latent defect and hunt for it.

#### Phase 1 — Requirement Mismatch Attack

1. Extract **acceptance criteria**, **test data**, and **expected behavior** from the ticket description and comments.
2. For each criterion, attempt to construct a scenario where the code produces the wrong result or silently does nothing.
3. Look for requirements that are addressed in name only — where the code path exists but the behavior diverges from what the ticket actually specifies.

#### Phase 2 — Fault Injection (Mental Fuzzing)

For every changed method or code path, ask:

- **Null / empty inputs**: What happens if any parameter, collection, or config value is null, empty, or whitespace?
- **Boundary values**: What about zero, negative, max-int, empty arrays, single-element lists, strings at max length?
- **Concurrency**: Can two requests hit this path simultaneously and corrupt shared state?
- **Ordering**: Does this assume a specific call order that isn't enforced?
- **External failures**: What if an API call, DB query, or file read throws? Is there a silent swallow?

#### Phase 3 — Blast Radius Analysis

1. Trace every changed public interface upstream and downstream. Identify callers that were not updated.
2. Look for implicit contracts (e.g., a method that used to return non-null now can return null).
3. Check if removed or renamed symbols leave dead references in views, JSON contracts, or config.
4. Verify feature flags, DI registrations, and route registrations are consistent with the change.

#### Phase 4 — Security Adversarial Pass

- **Injection**: Can user-controlled input reach SQL, HTML, or command execution without sanitization?
- **AuthZ/AuthN**: Does the change accidentally expose data or actions to unauthorized users?
- **Information leakage**: Do error messages, logs, or API responses expose internals?
- **IDOR**: Can a user manipulate IDs to access another tenant's or user's data?

#### Phase 5 — Specification Completeness

- List every acceptance criterion and mark it ✅ proven-covered or ❌ not-covered/partially-covered.
- Flag any behavior that is implemented but has **no corresponding test** — treat untested logic as suspect.
- Identify any ticket requirement that is entirely missing from the diff.

### 2.6 Summarize Findings

Present the review as:

1. **PR Overview**: Title, author, branch info, files changed.
2. **Ticket Context**: Brief summary of what the ticket(s) require.
3. **Changes Summary**: What each file change does.
4. **Adversarial Findings**: For each finding, state:
   - The **attack vector** (how you tried to break it)
   - The **evidence** (file:line citation and reproduction scenario)
   - The **impact** (what goes wrong if this defect ships)
5. **Issues Table**: Categorized by severity:
   - **Critical/Bug**: Will cause runtime errors, data loss, or security breach.
   - **High**: Doesn't meet spec requirements, risks data corruption, or has no test coverage for critical logic.
   - **Medium**: Potential issues under specific conditions that should be clarified or defended.
   - **Low**: Code quality, minor improvements, defensive hardening.
   - **Info**: Observations, compiler warnings, style.
6. **Acceptance Criteria Checklist**: Every criterion marked ✅ or ❌ with file:line proof.
7. **Verdict**: Approve / Request changes / Needs clarification.

> **Verdict bias**: Default to "Request changes" unless you can prove correctness for all critical paths. The burden of proof is on the code, not the reviewer.

### 2.7 Offer Fixes and Unit Tests

For each identified issue:

1. Propose a concrete code fix.
2. Provide relevant unit tests following the project conventions (see the `unit-tests` skill for test conventions).

Unit test file location mirrors the source structure under `<project>.Tests/`:

```
Source:  FirstMile.Services/Helpers/OrderCreationHelper.cs
Test:    FirstMile.Services.Tests/Helpers/OrderCreationHelperTests.cs

Source:  firstmile.web/Api/SavedBasketController.cs
Test:    firstmile.web.Tests/Api/SavedBasketControllerTests.cs
```

Treat wrong test project placement as a review finding. If a test targets one assembly but lives under another assembly's test project, request changes. If the matching test project does not exist, require creating it rather than accepting the wrong destination.

Reference `FirstMile.Services.Tests/Email/EmailServiceTests.cs` for test style examples.

## Step 3: Apply Changes

After presenting the review and proposed fixes:

1. Ask the user to confirm which fixes to apply.
2. Apply the approved changes to the source branch files.
3. **Do NOT push automatically.** Remind the user to review the changes carefully before pushing to the remote.

## Notes

- If anything is unclear about the requirements or the code, **ask the user** — do not make assumptions. Refer to `.ai/standards/stakeholders.md` for escalation guidance (PO for requirements, Rob Paine for Salesforce, Tuyen Pham for architecture).
- When the diff is large, focus on the most impactful changes first.
- Always verify string literals, magic values, and status codes against the Jira ticket spec.
- Pay special attention to Salesforce API field values — typos in status strings or field names will cause silent failures.
