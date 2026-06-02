# Testing And Quality

The solution includes a test project, but the current automated coverage is narrow.

## Current test project

`FirstMile.Services.Tests` references:

- xUnit
- NSubstitute
- Microsoft.NET.Test.Sdk
- coverlet collector

## What is currently present

The repository currently contains one concrete test file under the test project:

- `FirstMile.Services.Tests/Email/EmailServiceTests.cs`

That file covers:

- email body builders
- create-account and reset-password email content builders
- recipient routing and send-method logic in `EmailService`

## Practical coverage gap

There are currently no verified test files in this pass for:

- controller behavior
- cart and checkout flows
- recurring-order flows
- Salesforce access checks
- startup and configuration behavior
- frontend integration or widget logic

## Quality implication

The CI pipeline does run `dotnet test`, but today that primarily validates a small subset of the service layer. For most behavior changes, confidence still depends on code review, local validation, and environment testing rather than broad automated regression coverage.

## Recommended next documentation step

As feature chapters deepen, add a small “verified by tests” subsection where tests exist and explicitly call out where behavior is currently only code-inspected.
