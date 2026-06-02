---
name: unit-tests
description: 'Generate C# xUnit + NSubstitute unit tests for the KFWT.WebPlatform project. USE FOR: creating new test files, adding tests to existing files, generating tests for blocks, services, controllers, validators, selection factories, API endpoints, view components, and infrastructure code. DO NOT USE FOR: frontend/JS tests, integration tests, or test execution.'
argument-hint: 'Describe what class/method to test, or paste the source code'
---

# Unit Test Generation

Generate C# unit tests following the exact conventions of the `KFWT.WebPlatform.Test` project.

> **Companion document:** [`guideline.md`](./guideline.md) — comprehensive human-readable guide with full examples, lessons learned, and category-specific patterns extracted from the codebase. Read it for onboarding context; use THIS file for generation rules.

## AI Workflow Integration

This skill operates as **EXECUTOR** role within the `.ai/` workflow.

### Before generating tests:

1. Load `.ai/standards/testing-policy.md` — verify required test types for the change type.
2. If a task file exists in `.ai/tasks/`, check §3.7 (Unit Tests) for the planned test list.
3. Follow `exec-context.md` golden rules: grep before edit, cite `file:line`, stop on errors.
4. For comprehensive patterns and pitfalls, reference [`.github/skills/unit-tests/guideline.md`](./guideline.md).

### After generating tests:

1. Verify tests compile: `dotnet build` the test project.
2. Verify tests pass: `dotnet test` the test project.
3. If task file has DONE WHEN checkbox for tests — confirm it's satisfiable.
4. If the source module's public interface changed, flag `.ai/skills/{module}.md` for update.
5. **Include ticket ID in test class comments** — add a class-level comment referencing the ticket (e.g., `/// <summary>Tests for CartService EUR support. See FMI-944.</summary>`).
6. **Update relevant documentation** — if new test patterns or testing approaches were established, update `.github/skills/unit-tests/guideline.md`.

## Before You Start

1. **Read the source file** being tested to understand all public methods, constructors, and dependencies.
2. **Check for existing tests** in the mirror path under `src/KFWT.WebPlatform.Test/`.
3. **Identify the category** of code under test (service, block, controller, validator, selection factory, API, infrastructure) — each has specific patterns below.

## Testability Refactoring Is Allowed

- It is acceptable to make small, behavior-preserving refactors in the source code to enable unit testing.
- If a method is too large or mixes multiple concerns, splitting it into smaller methods/functions is allowed so tests can target clear units of behavior.
- Keep public behavior unchanged; if behavior must change, update tests to document and verify the intended new behavior.
- Prefer minimal, focused refactors that improve testability without broad architectural churn.

## Project Setup

- **Framework**: .NET 8.0 (`net8.0`)
- **Test framework**: xUnit 2.9.3
- **Mocking**: NSubstitute **4.3.0** — do NOT use 5.x; EPiServer constrains `Castle.Core < 5.0.0` which conflicts with NSubstitute ≥ 4.4.0.
- **CMS**: Optimizely (EPiServer) 12.x
- **Global usings** (`Usings.cs`): `global using Xunit;` and `global using NSubstitute;` — do NOT add these in test files.
- **Implicit usings**: Enabled — standard `System.*` and `Microsoft.*` namespaces are available without explicit `using` statements.

## File & Namespace Conventions

### File Naming

- Test file: `[SourceClassName]Tests.cs`
- Test helper file: `[Feature]TestHelper.cs` (static class with test data)

### File Location

Mirror the source structure exactly under `<project_to_be_tested>.Tests/`:

```
Source:  FirstMile.Salesforce/Services/AccountService.cs
Test:    FirstMile.Salesforce.Tests/Services/AccountServiceTests.cs

Source:  firstmile.web/Features/CartPage/CartPageController.cs
Test:    firstmile.web.Tests/Features/CartPage/CartPageControllerTests.cs
```

This is a hard rule:

- Use the test project that matches the source assembly.
- Mirror the source folder path exactly.
- If the matching `<project>.Tests` project does not exist yet, create it instead of placing the test in a different test project.
- Do not place `FirstMile.Models`, `FirstMile.Integration`, `FirstMile.WebUtils`, `FirstMile.Services`, or `FirstMile.Salesforce` tests under `firstmile.web.Tests`.

### Namespace

Use file-scoped namespace matching the folder path:

```csharp
namespace FirstMile.Salesforce.Tests.Services;
```

## Class Structure

### Standard Template

```csharp
using Relevant.Namespaces.Here;

namespace FirstMile.Salesforce.Tests.[MirrorPath];

public class [SourceClass]Tests
{
    // 1. Private readonly substitutes (declared with inline initialization)
    private readonly IServiceA _serviceA = Substitute.For<IServiceA>();
    private readonly IServiceB _serviceB = Substitute.For<IServiceB>();

    // 2. System under test
    private readonly MyService _service;

    // 3. Constructor — initialize substitutes, configure ServiceLocator if needed, create SUT
    public [SourceClass]Tests()
    {
        // ServiceLocator setup (only if source uses Optimizely CMS services)
        var serviceCollection = new ServiceCollection();
        serviceCollection.AddSingleton(_serviceA);
        ServiceLocator.SetScopedServiceProvider(serviceCollection.BuildServiceProvider());

        // Create system under test
        _service = new MyService(
            _serviceA,
            _serviceB);
    }

    // 4. Test methods
    [Fact]
    public void MethodName_Scenario_ExpectedBehavior()
    {
        // Arrange
        var input = new InputType { Property = "value" };

        // Act
        var result = _service.Method(input);

        // Assert
        Assert.NotNull(result);
        Assert.Equal("expected", result.Property);
    }
}
```

### Key Rules

- **All substitute fields**: `private readonly T _name = Substitute.For<T>();`
- **Field naming**: descriptive dependency names with `_` prefix (e.g., `_contentLoader`, `_imageService`), `_` prefix for SUT (e.g., `_service`, `_validator`)
- **Constructor**: Initialize all substitutes and the SUT. Keep shared `Returns()`/`When..Do()` calls in the constructor only for reused configurations.
- **No IDisposable**: Test classes do NOT implement `IDisposable`.
- **No base classes or fixtures**: No `IClassFixture`, `ICollectionFixture`, or base test classes.

## Test Method Conventions

Unit test cases must be comprehensive and cover both normal flows and edge cases for each public behavior under test.

### Naming Pattern

Follow [Microsoft's unit test naming standard](https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices#naming-your-tests). Each test name has three parts:

```
[MethodBeingTested]_[Scenario]_[ExpectedBehavior]
```

- **MethodBeingTested**: The name of the method or member under test.
- **Scenario**: The condition or input under which the method is invoked.
- **ExpectedBehavior**: The expected result or outcome.

Examples:

```csharp
CreateViewModel_BlockHasNoContent_ReturnsEmptyInsights()
Validate_RequiredFieldsMissing_ReturnsError()
GetSelections_NullContext_ReturnsExpectedThemes()
CalculateStampDuty_ValidPropertyValue_ReturnsCorrectResults()
InvokeAsync_ValidBlock_ReturnsCorrectViewWithModel()
Constructor_DefaultValues_InitializesViewModel()
```

### Arrange / Act / Assert

Always use `// Arrange`, `// Act`, `// Assert` comment separators with a blank line before each section:

```csharp
[Fact]
public void Process_BlockWithTitle_ReturnsMappedTitle()
{
    // Arrange
    var block = new MyBlock { Title = "Test" };

    _serviceDependency.GetData(Arg.Any<string>())
        .Returns(new Data());

    // Act
    var result = _service.Process(block);

    // Assert
    Assert.NotNull(result);
    Assert.Equal("Test", result.Title);
}
```

If Arrange is trivial (one line), you may omit the comment but keep Act/Assert:

```csharp
[Fact]
public void GetSelections_NullContext_ReturnsExpectedThemes()
{
    // Arrange
    var factory = new ThemeSelectionFactory();

    // Act
    var selections = factory.GetSelections(null).ToList();

    // Assert
    Assert.Equal(4, selections.Count);
}
```

### Test Attributes

| Attribute                   | When to Use                                        |
| --------------------------- | -------------------------------------------------- |
| `[Fact]`                    | Single scenario, no data variations                |
| `[Theory]` + `[InlineData]` | Same logic with multiple input/output combinations |

**[Theory] Example:**

```csharp
[Theory]
[InlineData(30000, "£0.00", "0.0%")]
[InlineData(500000, "£14,500.00", "2.9%")]
[InlineData(1000000, "£39,500.00", "4.0%")]
public void CalculateTax_VariousPropertyValues_ReturnsCorrectTaxAndRate(
    int propertyValue,
    string expectedTax,
    string expectedRate)
{
    // Arrange
    SetupMockData();

    // Act
    var result = _service.CalculateTax(propertyValue);

    // Assert
    Assert.Equal(expectedTax, result.Tax);
    Assert.Equal(expectedRate, result.Rate);
}
```

Do NOT use `[MemberData]` or `[ClassData]` — this project only uses `[InlineData]`.

## NSubstitute Patterns

### Returns

```csharp
_service.GetData(Arg.Any<string>()).Returns(data);
_service.GetDataAsync(Arg.Any<int>()).Returns(Task.FromResult(data));
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
var block = new MyBlock();
_contentLoader.TryGet(Arg.Any<Guid>(), out Arg.Any<MyBlock>())
    .Returns(callInfo =>
    {
        callInfo[1] = block;
        return true;
    });
// or
var pageContent = Substitute.For<PageData>();
_contentLoader.TryGet(Arg.Any<ContentReference>(), out Arg.Any<PageData>())
    .Returns(callInfo =>
    {
        callInfo[1] = pageContent;
        return true;
    });
```

### Throws Pattern

```csharp
// Synchronous method
_service.GetData(Arg.Any<string>())
    .Returns(_ => throw new Exception("fail"));
```

### Async Throws Pattern

For `Task`-returning methods, the lambda form is **ambiguous** in NSubstitute 4.x — use `Task.FromException` instead:

```csharp
// WRONG — CS0121 ambiguous overload for async methods
_service.GetDataAsync(Arg.Any<string>())
    .Returns(_ => throw new Exception("fail"));

// CORRECT
_service.GetDataAsync(Arg.Any<string>())
    .Returns(Task.FromException<MyResult>(new Exception("fail")));

// For Task (non-generic)
_service.DoAsync(Arg.Any<string>())
    .Returns(Task.FromException(new Exception("fail")));
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

### Multi-Type Substitute for Interface Casting

```csharp
var block = Substitute.For<MyBlock, IContent>();
((IContent)block).ContentLink.Returns(new ContentReference(1));
```

### Verification

```csharp
_cache.Received(1).Insert(
    Arg.Any<string>(),
    Arg.Any<object>(),
    Arg.Any<CacheEvictionPolicy>());
```

## Assertion Patterns

### Common Assertions

```csharp
// Equality
Assert.Equal(expected, actual);

// Null checks
Assert.NotNull(result);
Assert.Null(result.Property);

// Boolean
Assert.True(result.IsValid);
Assert.False(result.HasErrors);

// Collections
Assert.Empty(errors);
Assert.NotEmpty(viewModel.Items);
Assert.Single(errors);                          // exactly one item
Assert.Equal(5, result.Items.Count);
Assert.Contains("keyword", result.Title);
Assert.Contains(errors, e => e.ErrorMessage.Contains("required"));

// Type checking
Assert.IsType<SuccessApiResult<Data>>(result);
Assert.IsAssignableFrom<IBlockViewModel<BlockData>>(model);
var typedResult = Assert.IsType<ErrorApiResult>(result);  // returns typed value

// Exceptions
Assert.Throws<ArgumentException>(() => MyMethod(null));
var ex = Assert.Throws<ArgumentException>(() => MyMethod(null));
Assert.Equal("Expected message", ex.Message);

// String matching
Assert.StartsWith("<script", json);
Assert.EndsWith("#arrow-down", result);
```

### Assert.Single with Extraction

```csharp
var insight = Assert.Single(result.Insights);
Assert.Equal("Title", insight.Title);
```

## Category-Specific Patterns

### Selection Factory Tests

```csharp
public class MyThemeSelectionFactoryTests
{
    [Fact]
    public void GetSelections_NullContext_ReturnsExpectedThemes()
    {
        // Arrange
        var factory = new MyThemeSelectionFactory();

        // Act
        var selections = factory.GetSelections(null).ToList();

        // Assert
        Assert.Equal(4, selections.Count);

        Assert.Equal("Grey ", selections[0].Text);
        Assert.Equal(Globals.Theme.Grey, selections[0].Value);

        Assert.Equal("Pink", selections[1].Text);
        Assert.Equal(Globals.Theme.Pink, selections[1].Value);
        // ... assert every item's Text and Value
    }
}
```

- Always pass `null` to `GetSelections()`.
- Assert `.Count` first, then each item's `.Text` and `.Value` in order.
- If a static `ThemeItems()` method exists, test it separately.

### Validator Tests

```csharp
public class MyBlockValidationTests
{
    private readonly MyBlockValidation _validator = new();

    [Fact]
    public void Validate_RequiredFieldsMissing_ReturnsError()
    {
        // Arrange
        var block = new MyBlock { RequiredField = null };

        // Act
        var errors = _validator.Validate(block).ToList();

        // Assert
        Assert.Single(errors);
        Assert.Equal("Expected error message.", errors[0].ErrorMessage);
    }

    [Fact]
    public void Validate_AllFieldsValid_ReturnsNoErrors()
    {
        // Arrange
        var block = new MyBlock { RequiredField = "value" };

        // Act
        var errors = _validator.Validate(block).ToList();

        // Assert
        Assert.Empty(errors);
    }
}
```

- Instantiate validator directly: `private readonly MyValidation _validator = new();`
- If the validator has constructor dependencies, substitute them.
- Always call `.Validate(block).ToList()`.
- Test both error and no-error paths.
- Assert on `errors[0].ErrorMessage` and optionally `errors[0].PropertyName`.

### Block Service Tests

```csharp
public class MyBlockServiceTests
{
    private readonly IImageService _imageService = Substitute.For<IImageService>();
    private readonly IPageHelperService _pageHelperService = Substitute.For<IPageHelperService>();
    private readonly MyBlockService _service;

    public MyBlockServiceTests()
    {
        // ServiceLocator if needed
        _service = new MyBlockService(
            _imageService,
            _pageHelperService);
    }

    [Fact]
    public void CreateViewModel_BlockHasData_MapsProperties()
    {
        // Arrange
        var block = new MyBlock
        {
            Title = "Test",
            Image = new ContentReference(123)
        };

        _imageService.GetImageInfo(
            Arg.Any<ContentReference>(),
            Arg.Any<string>(),
            Arg.Any<string>()
        ).Returns(new ImageInfo(false) { Url = "/img.jpg", AltText = "Alt" });

        // Act
        var viewModel = _service.CreateViewModel(block);

        // Assert
        Assert.Equal("Test", viewModel.Title);
        Assert.Equal("/img.jpg", viewModel.Image.Src);
    }
}
```

### Async View Component Tests

```csharp
public class MyBlockAsyncComponentTests
{
    [Fact]
    public async Task InvokeAsync_ValidBlock_ReturnsCorrectViewWithModel()
    {
        // Arrange
        var block = new MyBlock();
        var httpContext = new DefaultHttpContext();
        var viewContext = new ViewContext { HttpContext = httpContext };
        var viewComponentContext = new ViewComponentContext { ViewContext = viewContext };

        var service = Substitute.For<IMyBlockService>();
        var viewComponent = new MyBlockAsyncComponent(service)
        {
            ViewComponentContext = viewComponentContext
        };

        // Act
        var result = await viewComponent.InvokeAsync(block) as ViewViewComponentResult;

        // Assert
        Assert.NotNull(result);
        Assert.Equal(
            $"~/Features/Blocks/{block.GetOriginalType().Name}/{block.GetOriginalType().Name}.cshtml",
            result.ViewName);
        Assert.NotNull(result.ViewData);
    }
}
```

### API Controller Tests

```csharp
public class MyApiControllerTests
{
    private readonly IMyService _service = Substitute.For<IMyService>();
    private readonly ILogger<MyApiController> _logger = Substitute.For<ILogger<MyApiController>>();
    private readonly MyApiController _controller;

    public MyApiControllerTests()
    {
        // ServiceLocator if needed for CMS dependencies

        var httpContext = Substitute.For<HttpContext>();
        httpContext.Items.Returns(new Dictionary<object, object>());

        _controller = new MyApiController(
            _service,
            _logger);
        _controller.ControllerContext = new ControllerContext
        {
            HttpContext = httpContext
        };
    }

    [Fact]
    public void GetData_ValidRequest_ReturnsSuccess()
    {
        // Arrange
        var request = new MyRequest { Query = "test" };
        var response = new MyResponse();
        _service.GetData(Arg.Any<MyRequest>())
            .Returns(response);

        // Act
        var result = _controller.GetData(request);

        // Assert
        var success = Assert.IsType<SuccessApiResult<MyResponse>>(result);
        Assert.Equal(response, success.Data);
    }

    [Fact]
    public void GetData_ExceptionThrown_ReturnsError()
    {
        // Arrange
        _service.GetData(Arg.Any<MyRequest>())
            .Returns(_ => throw new Exception("fail"));

        // Act
        var result = _controller.GetData(new MyRequest());

        // Assert
        var error = Assert.IsType<ErrorApiResult>(result);
        Assert.Contains("fail", error.Errors.Values.First()[0]);
    }
}
```

### DataAnnotations Validation Tests

```csharp
[Fact]
public void TryValidateObject_RequiredFieldsMissing_FailsValidation()
{
    // Arrange
    var item = new MyModel();
    var validationResults = new List<ValidationResult>();

    // Act
    var isValid = Validator.TryValidateObject(
        item,
        new ValidationContext(item),
        validationResults,
        true);

    // Assert
    Assert.False(isValid);
    Assert.Contains(validationResults,
        r => r.MemberNames.Contains(nameof(MyModel.RequiredProperty)));
}
```

### Services That Send Side-Effects (Email, HTTP, etc.)

When a service method builds content (e.g. an email body) AND dispatches it (e.g. via SMTP), split testing into two layers:

#### Layer 1 — Extract `internal static` builder methods and test them directly

Refactor the source class to extract pure formatting/building logic into `internal static` methods, then expose them to the test project via `InternalsVisibleTo`.

**In the source project** (`AssemblyInfo.cs` or top of the source file):

```csharp
using System.Runtime.CompilerServices;

[assembly: InternalsVisibleTo("MyProject.Tests")]
```

**In the source class** — extract the logic:

```csharp
internal static (string Subject, string Body) BuildWelcomeEmailContent(
    string link, string? subjectOverride, string? bodyTemplate)
{
    var subject = string.IsNullOrWhiteSpace(subjectOverride)
        ? "Default Subject"
        : subjectOverride;

    var body = string.IsNullOrWhiteSpace(bodyTemplate)
        ? $"Click <a href=\"{link}\">here</a>."
        : bodyTemplate.Replace("{{link}}", link).Replace("\n", "<br />");

    return (subject, body);
}
```

**In the test** — call the builder directly with zero mocks:

```csharp
public class MyServiceBuilderTests
{
    [Fact]
    public void BuildWelcomeEmailContent_NullTemplate_UsesDefaultSubjectAndLink()
    {
        // Act
        var (subject, body) = MyService.BuildWelcomeEmailContent(
            link: "https://example.com/verify",
            subjectOverride: null,
            bodyTemplate: null);

        // Assert
        Assert.Equal("Default Subject", subject);
        Assert.Contains("<a href=\"https://example.com/verify\">here</a>", body);
    }

    [Fact]
    public void BuildWelcomeEmailContent_CustomTemplate_ReplacesTokenAndNewlines()
    {
        // Act
        var (subject, body) = MyService.BuildWelcomeEmailContent(
            link: "https://example.com/verify",
            subjectOverride: "Welcome!",
            bodyTemplate: "Line one\nClick {{link}}");

        // Assert
        Assert.Equal("Welcome!", subject);
        Assert.Contains("Line one<br />Click https://example.com/verify", body);
    }
}
```

#### Layer 2 — Intercept the dispatch method via `protected virtual` override

Make the dispatch method `protected virtual` in the source, then subclass it in a private `TestXxx` inner class to capture what was dispatched without actually sending anything.

**In the source class**:

```csharp
protected virtual async Task SendMail(MailMessage message)
{
    // ... real SMTP logic ...
}
```

**In the test file** — capture pattern with `sealed record`:

```csharp
public class MyServiceTests
{
    private readonly List<CapturedMailMessage> _capturedMessages = [];
    private readonly TestMyService _service;

    public MyServiceTests()
    {
        _service = new TestMyService(
            /* dependencies */,
            capturedMail => _capturedMessages.Add(capturedMail));
    }

    [Fact]
    public async Task SendWelcomeEmail_ValidRecipient_SendsWithCorrectSubjectAndTo()
    {
        // Arrange
        // ... set up substitutes ...

        // Act
        await _service.SendWelcomeEmail("user@example.com", "https://example.com/link");

        // Assert
        var message = Assert.Single(_capturedMessages);
        Assert.Equal("user@example.com", Assert.Single(message.To));
        Assert.Equal("Default Subject", message.Subject);
        Assert.True(message.IsBodyHtml);
    }

    private sealed class TestMyService(/* dependencies */, Action<CapturedMailMessage> capture)
        : MyService(/* dependencies */)
    {
        protected override Task SendMail(MailMessage message)
        {
            capture(new CapturedMailMessage(
                message.From?.Address ?? string.Empty,
                message.To.Select(m => m.Address).ToList(),
                message.Subject,
                message.Body,
                message.IsBodyHtml));
            message.Dispose();
            return Task.CompletedTask;
        }
    }

    private sealed record CapturedMailMessage(
        string From,
        List<string> To,
        string Subject,
        string Body,
        bool IsBodyHtml);
}
```

#### Two-class split convention

When both layers apply, put them in **two classes in the same file** with a clear comment separator:

```csharp
// Builder tests — no mocks, call static methods directly
public class MyServiceBuilderTests { ... }

// Send-method tests — validate routing/data-gathering logic
public class MyServiceTests { ... }
```

### JSON Serialization Tests

```csharp
[Fact]
public void Serialize_ValidModel_ProducesCorrectJson()
{
    // Arrange
    var model = new MyModel { Name = "Test", Type = "Example" };

    // Act
    var json = JsonSerializer.Serialize(model);

    // Assert
    Assert.Contains("\"name\":\"Test\"", json);
    Assert.Contains("\"@type\":\"Example\"", json);
}

[Fact]
public void Deserialize_ValidJson_ProducesCorrectModel()
{
    // Arrange
    var json = "{\"name\":\"Test\",\"@type\":\"Example\"}";

    // Act
    var model = JsonSerializer.Deserialize<MyModel>(json);

    // Assert
    Assert.NotNull(model);
    Assert.Equal("Test", model.Name);
}
```

## Optimizely CMS Setup Patterns

When the source code depends on Optimizely's `ServiceLocator`, register needed services in the test constructor:

### ServiceCollection Pattern (preferred)

```csharp
public MyTests()
{
    var serviceCollection = new ServiceCollection();
    serviceCollection.AddSingleton(_contentLanguageAccessor);
    serviceCollection.AddSingleton(_contentLoader);
    ServiceLocator.SetScopedServiceProvider(serviceCollection.BuildServiceProvider());
}
```

### IServiceProvider Substitute Pattern

```csharp
public MyTests()
{
    var serviceProvider = Substitute.For<IServiceProvider>();
    serviceProvider.GetService(typeof(IOptions<MyOptions>))
        .Returns(Options.Create(new MyOptions { Setting = "value" }));
    ServiceLocator.SetScopedServiceProvider(serviceProvider);
}
```

### SiteDefinition Substitute

```csharp
var siteDef = Substitute.For<SiteDefinition>();
siteDef.RootPage.Returns(new ContentReference(1));
siteDef.StartPage.Returns(new ContentReference(9));
siteDef.SiteAssetsRoot.Returns(new ContentReference(9));
siteDef.GlobalAssetsRoot.Returns(new ContentReference(3));
SiteDefinition.Current = siteDef;
```

### ContentLanguageAccessor

```csharp
var contentLanguageAccessor = Substitute.For<IContentLanguageAccessor>();
contentLanguageAccessor.Language.Returns(new CultureInfo("en-GB"));
```

## Test Helper Classes

For blocks with complex test data, create a static helper class in the same folder:

```csharp
// File: [Feature]TestHelper.cs
public static class MyBlockTestHelper
{
    public static MyBlock GetConfiguredBlock
    {
        get
        {
            return new MyBlock
            {
                Brackets = new List<BracketItem>
                {
                    new() { MinAmount = 0, MaxAmount = 150000, Rate = 0 },
                    new() { MinAmount = 150001, MaxAmount = 250000, Rate = 2 }
                }
            };
        }
    }
}
```

Use static properties (not methods) for simple pre-configured objects.

## Private Helper Methods

For complex substitute setups reused across tests, create private helper methods in the test class:

```csharp
private void SetupMockData()
{
    var block = TestHelper.GetConfiguredBlock;
    _contentLoader.TryGet(Arg.Any<Guid>(), out Arg.Any<MyBlock>())
        .Returns(callInfo =>
        {
            callInfo[1] = block;
            return true;
        });
}

private static MyBlock MockBlock(ContentReference blockRef = null)
{
    var block = Substitute.For<MyBlock, IContent>();
    block.Property.Returns("value");
    ((IContent)block).ContentLink
        .Returns(blockRef ?? ContentReference.EmptyReference);
    return block;
}
```

## Private Inner Test Classes

For test-only data models (e.g., configuration binding), define them as `private class` inside the test class:

```csharp
public class ConfigServiceTests
{
    [Fact]
    public void GetSection_ValidConfig_BindsCorrectly()
    {
        // ... test logic using TestConfig
    }

    private class TestConfig
    {
        public string Property1 { get; set; }
    }
}
```

## Checklist

Before finalizing generated tests, verify:

- [ ] File is in the correct mirror path under `<project_to_be_tested>.Tests`
- [ ] File is in the test project that matches the source assembly, or a new matching test project was created
- [ ] Namespace matches folder structure: `<project_to_be_tested>.Tests.[Path]`
- [ ] Class name ends with `Tests`
- [ ] All substitute fields use `private readonly T _name = Substitute.For<T>();`
- [ ] SUT is created in constructor with substitute references
- [ ] `// Arrange` / `// Act` / `// Assert` comments are present
- [ ] Method names follow `MethodBeingTested_Scenario_ExpectedBehavior` pattern
- [ ] `[Fact]` for single cases, `[Theory]` + `[InlineData]` for parameterized
- [ ] No `using Xunit;` or `using NSubstitute;` (they are global)
- [ ] ServiceLocator is set up if source uses CMS services
- [ ] Both happy path and error/edge cases are covered
- [ ] Validator tests cover both error and no-error paths
- [ ] Selection factory tests assert count then each Text/Value pair
- [ ] NSubstitute version is **4.3.0** (not 5.x) in the `.csproj`
- [ ] Async throws use `Task.FromException<T>(...)`, not `.Returns(_ => throw ...)`
- [ ] Services with side-effects: builder methods extracted as `internal static` and tested directly; dispatch method made `protected virtual` and intercepted via inner subclass
- [ ] `[assembly: InternalsVisibleTo("...")]` added to source project when `internal` members are tested
