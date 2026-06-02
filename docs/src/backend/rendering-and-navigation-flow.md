# Rendering And Navigation Flow

The rendering model combines Optimizely content routing, feature-folder views, and runtime redirect filters.

## Page controller baseline

`PageControllerBase<T>` inherits from Optimizely `PageController<T>` and applies `RedirectSalesforceUserToLocationHomeAttribute` to all derived site-page controllers.

That means normal page rendering is wrapped in a redirect policy that can change navigation based on:

- whether the current user is a Salesforce-backed portal user
- whether the current page sets `RedirectSalesforceUsersFromThisPageToLocationHome`
- whether the current page sets `RedirectToLoginPage`
- whether the request is in edit mode

## Redirect behavior

`RedirectSalesforceUserToLocationHomeAttribute` implements two key flows:

### Logged-in Salesforce user

If the current page is not `LoginPage` or `AccountLocationPage` and the page flag `RedirectSalesforceUsersFromThisPageToLocationHome` is enabled, the filter redirects to the configured account-location page.

### Anonymous or non-Salesforce user

If the current page is not `LoginPage` and the page flag `RedirectToLoginPage` is enabled, the filter redirects to the configured login page and appends the current request path as `returnUrl`.

## Razor model baseline

`RazorPageModelBase<T>` standardizes page models around `SitePageData` and adds a `Layout` object containing shared layout values such as:

- head script
- logo URL
- footer copyright
- social links
- search page URL
- social share image URL

## View-location rule

`EngineConfigurator.ConfigureRazor` adds `/Features/{0}.cshtml` to the Razor view-location formats. That confirms the feature-folder layout is a first-class rendering convention, not an ad hoc folder structure.

## Practical implication

Rendering behavior is influenced by both CMS content and request context. To explain why a page redirects or renders a certain layout, inspect together:

- the page type and its flags
- the feature controller or page model
- the redirect filter
- settings resolved from the home page

## Source anchors

- `firstmile.web/Features/PageBaseController.cs`
- `firstmile.web/Features/RazorPageModelBase.cs`
- `firstmile.web/Authorization/RedirectSalesforceUserToLocationHomeAttribute.cs`
- `firstmile.web/Infrastructure/EngineConfigurator.cs`
