# DB And Logs Runbook

This chapter migrates the current operational notes from `docs/db-and-logs.md`.

## Export database

Use the `Export DB` pipeline in Azure DevOps.

Current documented expectations:

- branch: `develop`
- select the target DXP environment
- choose retention hours, with a documented default of `168` and maximum of `168`
- download the resulting database backup from pipeline artifacts after completion

## Retrieve logs

Use the `Get logs from DXP` pipeline in Azure DevOps.

Current documented parameters include:

- target environment
- starting window using days and hours
- ending window using days and hours
- timezone for readable output
- thread count used during log conversion

The legacy note documents UTC+7 as the default timezone used by the Hanoi team.

## Operational note

These are currently documented as manual runbooks rather than self-service application features. That means support and QA workflows depend on Azure DevOps access and knowledge of the pipeline parameters.
