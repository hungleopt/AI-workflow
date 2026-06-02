# Troubleshooting And Validation

This chapter collects the most immediate validation and failure points that can be verified from the repository.

## Deployment validation checklist

After a backend deployment or frontend package merge, the fastest high-value checks are:

1. confirm the target environment and branch mapping are correct
2. verify the application starts and the expected validation URL responds
3. verify login and account-location navigation still work
4. verify static assets load from `wwwroot/assets`
5. verify one account-portal flow such as order history or documents
6. verify one checkout-related flow if the release touches cart or pricing behavior

## Common failure surfaces

### Authentication issues

Check:

- Azure AD settings
- SAML metadata availability
- home-page login/account links
- redirect filter behavior on the current page

Symptoms likely include redirect loops, login landing on the wrong page, or account users being forced back to login.

### Salesforce-backed API failures

Check:

- current auth token refresh behavior
- distributed-cache token contents
- retry behavior on `401` and `500`
- access-validation failures in `SalesforceAuthorizeAttribute`

Symptoms likely include `401` JSON responses, empty account data, or portal pages loading without expected records.

### Frontend package issues

Check:

- whether the frontend integration pipeline detected any changes
- whether generated output appeared in `firstmile.web/wwwroot/assets`
- whether widget output landed in `firstmile.ui/widgets` during build
- whether the release package branch PR was created for the expected target branch

Symptoms likely include old assets in the environment, broken widget behavior, or missing pattern output.

### Media and image issues

Check:

- whether the environment is treated as Azure or local
- whether local `FirstMileConfig:ResizedImagesCache` is configured
- whether blob-cache settings are valid in the current environment

Symptoms likely include startup failures locally or missing resized images.

## DB export validation

The `get-db.yml` pipeline is a manual utility flow that:

- connects to Optimizely DXP with Epi CLI
- exports a named database from a chosen environment
- downloads the returned package URL
- publishes the result as `DXP_DB_PACKAGE`

If DB export fails, first verify the DXP project credentials and the selected environment/database parameters.

## Log retrieval validation

The `get-logs.yml` pipeline is a manual utility flow that:

- connects to Optimizely DXP with Epi CLI
- downloads logs for a configurable time window
- compresses the log folder to a `.7z` archive
- publishes the result as `DXP_LOG_PACKAGE`

If log retrieval fails, verify the time-window parameters, target environment, and whether the returned log folder exists before packaging.

## Source anchors

- `azure-pipelines.yml`
- `firstmile.ui/azure-pipelines-integration.yml`
- `get-db.yml`
- `get-logs.yml`
- `firstmile.web/Infrastructure/EngineConfigurator.cs`
- `firstmile.web/Authorization/SalesforceAuthorizeAttribute.cs`
