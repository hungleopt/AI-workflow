# Unit Test Guidelines — FirstMile Project

> Comprehensive reference for writing unit tests in this repository.
> For the AI skill file that codifies these patterns into generation rules, see [SKILL.md](./SKILL.md).
> If you hit a mock/stub problem while testing code in this repo and discover a working solution, update this guideline with the exact pattern before finishing so the next AI session does not need to rediscover it.

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Framework & Tooling](#framework--tooling)
3. [File & Namespace Conventions](#file--namespace-conventions)
4. [Test Class Structure](#test-class-structure)
5. [Naming Conventions](#naming-conventions)
6. [Arrange / Act / Assert Pattern](#arrange--act--assert-pattern)
7. [NSubstitute Patterns](#nsubstitute-patterns)
8. [Assertion Patterns](#assertion-patterns)
9. [Category-Specific Guidance](#category-specific-guidance)
10. [Optimizely CMS Patterns](#optimizely-cms-patterns)
11. [Common Pitfalls & Lessons Learned](#common-pitfalls--lessons-learned)
12. [Testability Refactoring](#testability-refactoring)
13. [Checklist](#checklist)

---

## Project Overview

The FirstMile backend is a .NET 8.0 application built on Optimizely CMS 12.x with Salesforce integrations. Tests are distributed across multiple test projects, each mirroring its source assembly:

| Source Project          | Test Project                  |
| ----------------------- | ----------------------------- |
| `firstmile.web`         | `firstmile.web.Tests`         |
| `FirstMile.Services`    | `FirstMile.Services.Tests`    |
| `FirstMile.Models`      | `FirstMile.Models.Tests`      |
| `FirstMile.Salesforce`  | `FirstMile.Salesforce.Tests`  |
| `FirstMile.Integration` | `FirstMile.Integration.Tests` |
| `FirstMile.WebUtils`    | `FirstMile.WebUtils.Tests`    |

---

## Framework & Tooling

| Component        | Version / Notes                                                                                       |
| ---------------- | ----------------------------------------------------------------------------------------------------- |
| Target framework | `net8.0`                                                                                              |
| Test framework   | xUnit 2.9.3                                                                                           |
| Mocking library  | NSubstitute **4.3.0** (NOT 5.x — Castle.Core <5.0 constraint)                                         |
| Global usings    | `Xunit` and `NSubstitute` are global — never add `using Xunit;` or `using NSubstitute;` in test files |
| Implicit usings  | Enabled — standard `System.*` and `Microsoft.*` namespaces available without explicit `using`         |

---

## File & Namespace Conventions

### Test Placement Rule (Hard Rule)

Tests MUST be placed in the test project that matches the source assembly. Mirror the source folder path exactly.

```text
Source:  FirstMile.Salesforce/Services/AccountService.cs
Test:    FirstMile.Salesforce.Tests/Services/AccountServiceTests.cs

Source:  firstmile.web/Features/CartPage/CartPageController.cs
Test:    firstmile.web.Tests/Features/CartPage/CartPageControllerTests.cs

Source:  FirstMile.Models/Poco/LocationHomeReorderModel.cs
Test:    FirstMile.Models.Tests/Poco/LocationHomeReorderModelTests.cs
```

If the matching `<project>.Tests` project does not exist, create it. Never cross-place tests (e.g., do NOT put `FirstMile.Services` tests in `firstmile.web.Tests`).

### File Naming

- Test file: `[SourceClassName]Tests.cs`
- Test helper: `[Feature]TestHelper.cs` (static class)
- Namespace: file-scoped, mirrors folder path exactly

```csharp
namespace FirstMile.Services.Tests.Commerce;
```

---

## Test Class Structure

### Standard Template

```csharp
using Relevant.Namespaces.Here;

namespace Project.Tests.Path;

public class MyServiceTests
{
    // 1. Private readonly substitutes (inline initialization)
    private readonly IServiceA _serviceA = Substitute.For<IServiceA>();
    private readonly IServiceB _serviceB = Substitute.For<IServiceB>();

    // 2. System under test (SUT)
    private readonly MyService _service;

    // 3. Constructor — create SUT with substitute references
    public MyServiceTests()
    {
        _service = new MyService(_serviceA, _serviceB);
    }

    // 4. Test methods
    [Fact]
    public void MethodName_Scenario_ExpectedBehavior()
    {
        // Arrange
        // Act
        // Assert
    }
}
```

### Key Structural Rules

- **All substitutes**: `private readonly T _name = Substitute.For<T>();`
- **Field naming**: `_` prefix, descriptive (e.g., `_contentLoader`, `_userService`)
- **SUT naming**: `_service`, `_controller`, `_validator`, `_component`
- **Constructor**: Initialize SUT. Place ServiceLocator setup here if needed
- **No IDisposable** on test classes
- **No base classes or fixtures**: No `IClassFixture`, `ICollectionFixture`, or abstract base test classes

---

## Naming Conventions

Follow Microsoft's unit test naming standard with three parts:

```text
[MethodBeingTested]_[Scenario]_[ExpectedBehavior]
```

### Examples from the Codebase

```csharp
CreateViewModel_BlockHasNoContent_ReturnsEmptyInsights()
Validate_RequiredFieldsMissing_ReturnsError()
GetSelections_NullContext_ReturnsExpectedThemes()
IndexAsync_CurrentUserAccountMissing_RedirectsToRoot()
GetInvoices_RecordsExist_MapsInvoicesAndPoNumbers()
OrderCompleteWorkflow_ProspectUser_SetsContactIdOnNewOpportunityRequest()
Constructor_DefaultValues_InitializesViewModel()
Login_SignedInOnMainHost_ReturnsSuccessResponseAndUpdatesContact()
ProcessAdditionalChargeAsync_EmptyJsonArray_ReturnsNull()
```

---

## Arrange / Act / Assert Pattern

Always use `// Arrange`, `// Act`, `// Assert` comments with a blank line before each:

```csharp
[Fact]
public async Task GetDocuments_ValidRequest_ReturnsMappedPayload()
{
    // Arrange
    var account = CreateAccount();
    _userService.GetCurrentUserAccountAsync().Returns(account);

    // Act
    var result = await _controller.GetDocuments();

    // Assert
    Assert.NotNull(result);
    Assert.Equal("expected", result.Property);
}
```

If Arrange is trivial (one line), you may omit its comment but ALWAYS keep `// Act` and `// Assert`.

---

## NSubstitute Patterns

### Basic Returns

```csharp
_service.GetData(Arg.Any<string>()).Returns(data);
_service.GetDataAsync(Arg.Any<int>()).Returns(Task.FromResult(data));
_service.IsCurrentUserSalesforceUserAsync().Returns(true);
```

### Returns with Specific Arguments

```csharp
_urlResolver.GetUrl(
    Arg.Is<ContentReference>(cref => cref.ID == 123),
    Arg.Any<string>(),
    null)
.Returns("/page-url");
```

### TryGet Pattern (Optimizely)

```csharp
_contentLoader.TryGet(Arg.Any<ContentReference>(), out Arg.Any<HomePage>())
    .Returns(callInfo =>
    {
        callInfo[1] = homePage;
        return true;
    });
```

### Arg.Do — Capture Arguments

```csharp
CreateCaseModel? capturedCase = null;
_caseService.CreateCaseAsync(Arg.Do<CreateCaseModel>(model => capturedCase = model))
    .Returns("case-123");

// Later in Assert:
Assert.NotNull(capturedCase);
Assert.Equal("account-1", capturedCase.AccountId);
```

### Async Throws — Use Task.FromException

```csharp
// CORRECT — avoids CS0121 ambiguous overload
_service.GetDataAsync(Arg.Any<string>())
    .Returns(Task.FromException<MyResult>(new Exception("fail")));

// WRONG — ambiguous in NSubstitute 4.x
_service.GetDataAsync(Arg.Any<string>())
    .Returns(_ => throw new Exception("fail"));
```

### Synchronous Throws

```csharp
_service.GetData(Arg.Any<string>())
    .Returns(_ => throw new Exception("fail"));
```

### Substitute ContentArea

```csharp
var contentArea = Substitute.For<ContentArea>();
var items = Enumerable.Range(1, 5)
    .Select(i => new ContentAreaItem { ContentLink = new ContentReference(i) })
    .ToList();
contentArea.Items.Returns(items);
contentArea.FilteredItems.Returns(items);
```

### Multi-Type Substitute (Interface Casting)

```csharp
var block = Substitute.For<MyBlock, IContent>();
((IContent)block).ContentLink.Returns(new ContentReference(1));
```

### Verification

```csharp
_service.Received(1).DoSomething(Arg.Any<string>());
_service.DidNotReceive().DoSomething(Arg.Any<string>());
await _service.Received(1).DoAsync(Arg.Any<string>());
```

### Nullable Parameters with Explicit Matchers

When stubbing methods with nullable parameters (especially `string?`), use explicit matchers to avoid `AmbiguousArgumentsException`:

```csharp
_uiUserProvider.CreateUserAsync(
    Arg.Any<string>(),
    Arg.Any<string>(),
    Arg.Any<string>(),
    Arg.Is<string?>(value => value == null)!,
    Arg.Is<string?>(value => value == null)!,
    true)
.Returns(Task.FromResult(createResult));
```

### UIUserProvider Stubbing Pattern

`UIUserProvider` is substitute-friendly in this repo, but its methods split into two patterns:

- `CreateUserAsync(...)` and `UpdateUserAsync(...)` return `Task`-based results, so stub and verify them with `await`
- `FindUsersByEmailAsync(...)`, `FindUsersByNameAsync(...)`, and `GetAllUsersAsync(...)` return `IAsyncEnumerable<IUIUser>`, so stub them with `.ToAsyncEnumerable()` and verify `Received()` / `DidNotReceive()` without `await`

```csharp
var user = Substitute.For<IUIUser>();
user.Username.Returns("user@example.com");

_uiUserProvider.FindUsersByEmailAsync("user@example.com", 0, 1)
    .Returns(new[] { user }.ToAsyncEnumerable());

_uiUserProvider.FindUsersByNameAsync("smith", 0, 1000)
    .Returns(batch.ToAsyncEnumerable());

_uiUserProvider.GetAllUsersAsync(0, 1000)
    .Returns(users.ToAsyncEnumerable());

_uiUserProvider.Received(1).FindUsersByNameAsync("smith", 0, 1000);
_uiUserProvider.DidNotReceive().GetAllUsersAsync(Arg.Any<int>(), Arg.Any<int>());
```

Do not write `await _uiUserProvider.Received(...).FindUsersByNameAsync(...)` because the search/list methods return `IAsyncEnumerable<IUIUser>`, not `Task`.

---

## Assertion Patterns

### Common Assertions

```csharp
Assert.Equal(expected, actual);
Assert.NotNull(result);
Assert.Null(result.Property);
Assert.True(result.IsValid);
Assert.False(result.HasErrors);
Assert.Empty(errors);
Assert.NotEmpty(viewModel.Items);
Assert.Single(errors);
Assert.Contains("keyword", result.Title);
Assert.IsType<SuccessApiResult<Data>>(result);
Assert.IsAssignableFrom<IBlockViewModel<BlockData>>(model);
```

### Type Assert with Value Extraction

```csharp
var typedResult = Assert.IsType<ErrorApiResult>(result);
Assert.Contains("fail", typedResult.Errors.Values.First()[0]);

var redirectResult = Assert.IsType<RedirectResult>(result);
Assert.Equal("/", redirectResult.Url);

var viewResult = Assert.IsType<ViewResult>(result);
var model = Assert.IsType<MyViewModel>(viewResult.Model);
```

### Assert.Single with Extraction

```csharp
var insight = Assert.Single(result.Insights);
Assert.Equal("Title", insight.Title);
```

### Collection Assertions

```csharp
Assert.Equal(2, result.Count);
Assert.All(result, location => Assert.Equal("account-1", location.ParentId));
Assert.Contains(errors, e => e.ErrorMessage.Contains("required"));
```

### JSON Document Assertions (Common Pattern for Controller Tests)

```csharp
using var json = JsonDocument.Parse(model.JsonData);
Assert.Equal("loc-1", json.RootElement.GetProperty("locationId").GetString());
Assert.True(json.RootElement.GetProperty("isEnabled").GetBoolean());
Assert.Equal(4, json.RootElement.GetProperty("items").GetArrayLength());
```

### Anonymous Object Assertions (API Controllers)

```csharp
var payload = GetOkPayload(result);  // helper: Assert.IsType<OkObjectResult>(result).Value
var items = GetEnumerable(GetPropertyValue(payload, "items"));
Assert.Equal("ORD-1", GetProperty<string>(items[0], "orderNumber"));
```

---

## Category-Specific Guidance

### Model / POCO Tests

Test default values, constructors, and computed properties. These are the simplest tests — no mocking needed.

```csharp
[Fact]
public void Constructor_InitializesExpectedDefaults()
{
    var model = new AuthorizationConfig();

    Assert.Equal("Authorization", AuthorizationConfig.SectionName);
    Assert.Equal(TimeSpan.FromMinutes(5), config.UserCacheTime);
}
```

### Page/Block `SetDefaultValues` Tests

Test the Optimizely `SetDefaultValues` method that initializes CMS content with default property values.

```csharp
[Fact]
public void PostCodeHeroBlock_SetDefaultValues_SetsExpectedDefaults()
{
    var block = new PostCodeHeroBlock();

    block.SetDefaultValues(new ContentType());

    Assert.Equal("Continue", block.ContinueButtonText);
    Assert.Equal("Edit Selection", block.EditSelectionText);
}
```

### Selection Factory Tests

- Always pass `null` to `GetSelections()`
- Assert `.Count` first, then each item's `.Text` and `.Value` in order

```csharp
[Fact]
public void GetSelections_NullContext_ReturnsExpectedThemes()
{
    var factory = new MyThemeSelectionFactory();

    var selections = factory.GetSelections(null).ToList();

    Assert.Equal(4, selections.Count);
    Assert.Equal("Grey", selections[0].Text);
    Assert.Equal(Globals.Theme.Grey, selections[0].Value);
}
```

### Validator Tests

- Instantiate directly (no ServiceLocator needed)
- Always call `.Validate(block).ToList()`
- Test both error AND no-error paths

```csharp
[Fact]
public void Validate_RequiredFieldsMissing_ReturnsError()
{
    var block = new MyBlock { RequiredField = null };

    var errors = _validator.Validate(block).ToList();

    Assert.Single(errors);
    Assert.Equal("Expected error message.", errors[0].ErrorMessage);
}
```

### Service Tests

Standard constructor-injection pattern. Register services in ServiceLocator only when source uses Optimizely APIs.

```csharp
public class UrlServiceTests
{
    private readonly IContentRepository _contentRepository = Substitute.For<IContentRepository>();
    private readonly IUrlResolver _urlResolver = Substitute.For<IUrlResolver>();
    // ... other dependencies

    [Fact]
    public void Url_IContentWithTemplate_ReturnsResolverUrl()
    {
        var content = Substitute.For<IContent>();
        _urlResolver.GetUrl(content).Returns("/content/with-template");
        _templateResolver.HasTemplate(content, ...).Returns(true);

        var service = CreateService();  // private helper to construct SUT

        var result = service.Url(content);

        Assert.Equal("/content/with-template", result);
    }
}
```

### Controller Tests (Page Controllers)

Controllers typically require:

- `ControllerContext` with `DefaultHttpContext`
- A substituted "current page" passed to the action method
- `ServiceLocator.SetScopedServiceProvider(...)` for CMS services
- `TempData` if the controller uses it

```csharp
public class MyPageControllerTests
{
    private readonly MyPageController _controller;

    public MyPageControllerTests()
    {
        _controller = new MyPageController(_dep1, _dep2);
        _controller.ControllerContext = new ControllerContext
        {
            HttpContext = new DefaultHttpContext()
        };
    }

    [Fact]
    public async Task IndexAsync_ValidAccount_ReturnsView()
    {
        var currentPage = Substitute.For<MyPage>();
        currentPage.Title.Returns("Page title");

        var result = await _controller.IndexAsync(currentPage);

        var viewResult = Assert.IsType<ViewResult>(result);
        Assert.Equal("~/Features/MyPage/Index.cshtml", viewResult.ViewName);
    }
}
```

### API Controller Tests

Similar to page controllers. Assert on `OkObjectResult`, `NotFoundResult`, `BadRequestObjectResult`, etc.

```csharp
[Fact]
public async Task GetCases_CurrentUserAccountMissing_ReturnsNotFound()
{
    _userService.GetCurrentUserAccountAsync().Returns((SimpleAccountModel?)null);

    var result = await _controller.GetCases(new RequestParams { LocationId = "loc-1" });

    Assert.IsType<NotFoundResult>(result);
    await _userService.DidNotReceive().HasAccessToLocationAsync(Arg.Any<string>());
}
```

### View Component Tests (Async Block Components)

```csharp
[Fact]
public async Task InvokeAsync_ValidBlock_ReturnsCorrectViewWithModel()
{
    var block = new MyBlock();
    var viewComponent = new MyBlockAsyncComponent(_service)
    {
        ViewComponentContext = new ViewComponentContext
        {
            ViewContext = new ViewContext { HttpContext = new DefaultHttpContext() }
        }
    };

    var result = await viewComponent.InvokeAsync(block) as ViewViewComponentResult;

    Assert.NotNull(result);
    Assert.Equal("~/Features/Blocks/MyBlock/MyBlock.cshtml", result.ViewName);
}
```

### Email / Side-Effect Services

Use a two-layer approach:

1. **Builder tests** — Extract `internal static` builder methods, test directly with zero mocks
2. **Dispatch tests** — Make dispatch method `protected virtual`, subclass in test with capture logic

```csharp
// Builder tests — no mocks
public class EmailServiceBuilderTests
{
    [Fact]
    public void BuildCreateAccountEmailContent_NullTemplate_UsesDefaultSubject()
    {
        var (subject, body) = EmailService.BuildCreateAccountEmailContent(
            link: "https://example.com/set-password",
            emailSubject: null,
            emailBodyTemplate: null);

        Assert.Equal("First Mile Create new Password", subject);
        Assert.Contains("<a href=\"https://example.com/set-password\">this link</a>", body);
    }
}
```

---

## Optimizely CMS Patterns

### ServiceLocator Setup

When source code depends on `ServiceLocator`, register services in the test constructor:

```csharp
public MyTests()
{
    var serviceCollection = new ServiceCollection();
    serviceCollection.AddSingleton(_contentLoader);
    serviceCollection.AddSingleton(_publishedStateAssessor);
    ServiceLocator.SetScopedServiceProvider(serviceCollection.BuildServiceProvider());
}
```

If multiple test classes mutate the static `ServiceLocator`, put them in a shared xUnit collection with parallelization disabled to avoid cross-test races:

```csharp
[CollectionDefinition("ServiceLocator tests", DisableParallelization = true)]
public class ServiceLocatorTestCollection
{
}

[Collection("ServiceLocator tests")]
public class MyComponentTests
{
}
```

### ContentArea in Tests

`ContentArea.Items.Add(...)` can trigger Optimizely fragment services. When only branch selection matters, seed via reflection or use `Substitute.For<ContentArea>()`:

```csharp
var contentArea = Substitute.For<ContentArea>();
contentArea.Items.Returns(items);
contentArea.FilteredItems.Returns(items);
```

For tests that must stay stable under `Release` plus `--collect:"XPlat Code Coverage"`, avoid relying on `ContentArea.LoadContent<T>()` internals if a narrower assertion is possible. Coverage instrumentation can change behavior enough that reflection-seeded content-area tests pass locally but fail in coverage runs. Prefer one of these patterns when they still verify the intended behavior:

- invoke a private mapping/helper method via reflection and assert its output directly
- cover the null or empty `ContentArea` path with `InvokeAsync(...)`
- assert logic that does not depend on Optimizely content loading internals

### Fragment Services Registration

If ContentArea operations fail, register fragment services:

```csharp
var securedFragmentMarkupGenerator = Substitute.For<ISecuredFragmentMarkupGenerator>();
securedFragmentMarkupGenerator.GenerateGroupDisplayInformation().Returns(string.Empty);
securedFragmentMarkupGenerator.GenerateCompressedGroupDisplayInformation().Returns(string.Empty);
securedFragmentMarkupGenerator.GenerateGroupStorageInformation().Returns(string.Empty);

var factory = Substitute.For<ISecuredFragmentMarkupGeneratorFactory>();
factory.CreateSecuredFragmentMarkupGenerator().Returns(securedFragmentMarkupGenerator);
serviceCollection.AddSingleton(factory);
```

### IContentLoader.TryGet Pattern

```csharp
_contentLoader.TryGet(Arg.Any<ContentReference>(), out Arg.Any<PageData>())
    .Returns(callInfo =>
    {
        callInfo[1] = pageContent;
        return true;
    });
```

### GetFriendlyUrl / IUrlService

Register `IUrlService` in ServiceLocator when source calls `GetFriendlyUrl()`:

```csharp
serviceCollection.AddSingleton(_urlService);
_urlService.Url(Arg.Any<Url?>())
    .Returns(callInfo => callInfo.Arg<Url?>()?.OriginalString ?? string.Empty);
```

---

## Common Pitfalls & Lessons Learned

### NSubstitute 4.x Constraints

- **Never use NSubstitute 5.x** — EPiServer constrains `Castle.Core < 5.0.0`
- **Async throws**: Use `Task.FromException<T>(...)` instead of `.Returns(_ => throw ...)`
- **Nullable parameters**: Use `Arg.Is<string?>(value => value == null)!` instead of bare `null`

### Optimizely-Specific

- `ContentArea.Items.Add(...)` triggers fragment services — use reflection or substitutes
- `ContentReference.StartPage` requires CMS app context — avoid in plain unit tests
- `AsyncBlockComponent<T>` inherits `AsyncPartialContentComponent<T>` — call `InvokeAsync(currentContent)` directly
- `IContentLoader` must be registered in ServiceLocator even for empty `ContentArea.LoadContent<T>()` paths
- Static `ServiceLocator` state is shared across tests — serialize all tests that call `ServiceLocator.SetScopedServiceProvider(...)` with one xUnit collection
- `Release` + `XPlat Code Coverage` is stricter than ad hoc Debug runs for some Optimizely content-area flows — if a content-loading assertion is flaky only under coverage, narrow the test to a stable helper or null-area branch instead of overfitting CMS internals
- `UIUserProvider` search/list methods return `IAsyncEnumerable<IUIUser>` — stub them with `.ToAsyncEnumerable()` and verify calls without `await`; only `Task`-returning methods like `CreateUserAsync(...)` should be awaited in assertions

### Build Output And Runtime Assets

- Some `firstmile.web` tests resolve files relative to the built web output, not the source tree. If you redirect `BaseOutputPath` for isolated test runs or coverage scripts, runtime file lookups may move under `.artifacts/.../firstmile.web/...`.
- If a test or controller reads assets from `wwwroot` such as PDF templates, copy `firstmile.web/wwwroot` into the redirected `firstmile.web` artifact root before running `firstmile.web.Tests`.
- Keep this in mind when updating `test.ps1`, Azure Pipeline coverage steps, or any local script that changes build output paths.

### Namespace And Type Ambiguity

- New test namespaces can shadow production type names. If a test namespace collides with a model or page class name such as `FeaturePostPage`, add an explicit type alias instead of renaming production code:

```csharp
using FeaturePostPageModel = FirstMile.Models.Pages.FeaturePostPage;
```

### Serialization

- `MoneyHelper.FormatPrice(...)` trims trailing zeroes — expect `£11` not `£11.00`, `£4.5` not `£4.50`
- `PostCodeHeroAsyncBlockComponent` uses `JsonConvert.SerializeObject(...)` (Newtonsoft) — nullable fields appear as explicit `null`
- Controller `Ok(...)` anonymous payloads — assert as `Array` and use `.Cast<object>().ToArray()`

### Service Behaviors

- `IMissedCollectionService.CreateMissedCollectionAsync(...)` returns `Task<string>` — stub must return string, not `Task.CompletedTask`
- `firstmile.web/Infrastructure` contains internal helpers — may need `[assembly: InternalsVisibleTo("firstmile.web.Tests")]`
- `IDistributedCache.SetStringAsync(...)` and `GetStringAsync(...)` are extension methods over `SetAsync(...)` and `GetAsync(...)`; when unit testing services like `AuthService`, configure and assert the underlying byte-array calls on `IDistributedCache`, typically with `Encoding.UTF8.GetBytes(...)` and `Encoding.UTF8.GetString(...)`

### Test Session Pattern

Many tests that touch session state use a `TestSession` inner class implementing `ISession`. This is a common test double in this codebase.

### Knowledge Capture Rule

If a test only became possible after solving a repo-specific mocking or stubbing problem, add that solution to this guideline in the closest relevant section before ending the session. Prefer concrete instructions tied to the actual API shape that caused the issue, including:

- the dependency or framework type involved
- the correct stub/setup pattern
- any verification caveat such as when not to `await`, when to use `Arg.Is(...)`, or when ServiceLocator registration is required

Treat this as part of finishing the test work, not optional cleanup.

---

## Testability Refactoring

Small, behavior-preserving refactors in source code are acceptable to enable testing:

- Splitting large methods into smaller testable units
- Extracting `internal static` builder methods from side-effect methods
- Making dispatch methods `protected virtual` for subclass interception
- Adding `[assembly: InternalsVisibleTo("...")]` for internal member access

Keep public behavior unchanged. Document the refactoring in the test file.

---

## Checklist

Before finalizing tests, verify:

- [ ] File is in the correct mirror path under `<project>.Tests`
- [ ] Namespace matches folder structure
- [ ] Class name ends with `Tests`
- [ ] All substitute fields use `private readonly T _name = Substitute.For<T>();`
- [ ] SUT is created in constructor with substitute references
- [ ] `// Arrange` / `// Act` / `// Assert` comments present
- [ ] Method names follow `MethodBeingTested_Scenario_ExpectedBehavior`
- [ ] `[Fact]` for single cases, `[Theory]` + `[InlineData]` for parameterized
- [ ] No `using Xunit;` or `using NSubstitute;` in the file
- [ ] ServiceLocator set up if source uses CMS services
- [ ] Any repo-specific mock/stub issue solved during the work has been added to this guideline so future AI sessions can reuse it
- [ ] Both happy path AND error/edge cases covered
- [ ] Async throws use `Task.FromException<T>(...)`
- [ ] NSubstitute version is 4.3.0 in `.csproj`
- [ ] `[MemberData]` / `[ClassData]` NOT used — only `[InlineData]`
- [ ] No `IDisposable`, no base classes, no fixtures
