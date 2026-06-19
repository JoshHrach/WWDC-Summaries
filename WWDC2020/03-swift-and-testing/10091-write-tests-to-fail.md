# Write Tests to Fail
**WWDC20 · Session 10091** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10091/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, Xcode 12

## Overview
"Write tests to fail" reframes the purpose of automated tests: the goal is not to see green, but to build tests that find bugs and provide enough context in the result bundle to triage failures without additional reproduction steps. The session covers four phases — setup, actions, assertions, and teardown — and shares concrete techniques for each: using `setUpWithError()` and launch arguments for fast, focused setup; modeling the app's UI hierarchy in test helper objects; writing assertion messages that aid triage; using `waitForExistence(timeout:)` instead of `sleep`; unwrapping optionals safely with `XCTUnwrap`; throwing errors from shared code instead of asserting; and using `XCTContext.runActivity` and `XCTAttachment` to enrich result bundles with breadcrumbs and files.

## Key Topics

### Setup: setUpWithError() and Launch Arguments
`XCTestCase.setUpWithError()` (introduced in Xcode 11.4) replaces `setUp()` and supports `throws`, enabling clean error propagation from setup code. Key practices:
- Set `continueAfterFailure = false` to stop at the first error — simplifies finding root cause
- Launch the app fresh for every test in the class to eliminate state leakage between tests
- Use `XCUIApplication.launchArguments` and `launchEnvironment` to communicate with the app and rapidly set state (e.g., bypass a login tab, skip onboarding) — faster than UI-driven setup and avoids triaging failures in unrelated tabs

### Structuring Test Actions with a UI Hierarchy Model
Factoring common test code into helper objects modeled after the app's view hierarchy makes tests readable and resilient to UI changes:
- Build wrapper classes (e.g., `FrutaApp`, `SmoothieList`, `Recipe`) that hold a reference to an `XCUIApplication` and an `XCUIElement`
- Helper methods return the next level's wrapper, creating a navigation chain like `app.smoothieList().selectRecipe(smoothie: .berryBlue)`
- Use `enum` for all string values (cell titles, accessibility IDs); when the UI changes, update one enum case — not dozens of test methods
- Share this helper code as a framework or Swift package across test targets and apps

### Assertions: Messages, Async, and Optionals
Three practices that make assertion failures usable in result bundles:

1. **Assertion messages** — always use the optional message parameter on `XCTAssert*` calls; include the values being compared and the expected meaning; avoid non-deterministic content (timestamps, file paths) in messages so CI systems can aggregate identical failures

2. **Asynchronous events** — use `XCUIElement.waitForExistence(timeout:)` instead of `sleep(_:)`: it polls and returns early when the condition is met, fails deterministically when the timeout expires, and shows the wait time in the result bundle

3. **Optional unwrapping** — never force-unwrap (`!`) in tests; use `if let`, `guard let`, `??`, or `XCTUnwrap(_:message:)`. `XCTUnwrap` is a throwing function that fails the test gracefully (tearDown is still called) instead of crashing (tearDown is skipped)

### Throwing Errors from Shared Code
Shared helper methods should throw errors, not call `XCTFail` directly:
- Callers can choose to propagate the error (normal test failure) or catch it (negative testing where the absence of an element is the expected outcome)
- Define error enums conforming to `Error` and `CustomStringConvertible` so that `error.description` produces a human-readable message in the result bundle

### XCTContext.runActivity and XCTAttachment
`XCTContext.runActivity(named:block:)` wraps a group of test steps under a named label in the result bundle:
- The label appears as a collapsible disclosure group, making the result bundle readable like a narrative
- Pass the `XCTActivity` provided to the block to `XCTAttachment`'s `add(to:)` to attach files, images, strings, or data directly to that activity
- Attach `element.debugDescription` or screenshots alongside a thrown error so the result bundle contains everything needed to reproduce the failure without running the test again

### XCTSkip (for Conditional Execution)
`XCTSkip`, `XCTSkipIf`, and `XCTSkipUnless` produce a third test result (skip) and are the correct way to handle tests that legitimately cannot run in the current environment. See session 10164 for a deep dive.

### tearDownWithError() and Data Collection
Use `tearDownWithError()` (the throwing counterpart of `tearDown`) to:
- Collect additional logging and analyze the state of the app after a failure
- Reset environment changes made during setup (e.g., clear test data, sign out)

## APIs & Frameworks

### XCTest
- `XCTestCase.setUpWithError() throws` **[NEW — Xcode 11.4]** — throwing setUp; replace existing `setUp()` calls
- `XCTestCase.tearDownWithError() throws` **[NEW — Xcode 11.4]** — throwing tearDown
- `XCTUnwrap(_ expression:, _ message:) throws -> T` **[NEW — Xcode 11.4]** — unwraps optional or throws XCTest failure (tearDown is still called)
- `XCUIElement.waitForExistence(timeout: TimeInterval) -> Bool` — polls for element existence; preferred over `sleep`
- `XCTContext.runActivity(named:block:)` — groups steps under a named label in the result bundle
- `XCTAttachment(string:)`, `XCTAttachment(image:)`, `XCTAttachment(data:)` — attach content to an `XCTActivity` or `XCTestCase`
- `XCTSkip` / `XCTSkipIf(_:_:)` / `XCTSkipUnless(_:_:)` **[NEW — Xcode 11.4]** — produce skip result
- `XCUIApplication.launchArguments: [String]` — array of arguments passed to the app at launch; read with `CommandLine.arguments` in the app
- `XCUIApplication.launchEnvironment: [String: String]` — environment variables for the launched process

## Code Highlights

Setup with launch argument for state bypass:
```swift
class RecipesTests: XCTestCase {
    let app = FrutaApp()

    override func setUpWithError() throws {
        continueAfterFailure = false
        app.launchArguments.append("-recipes-tests")
        app.launch()
    }
}
// In app code:
@State private var selection: Tab =
    CommandLine.arguments.contains("-recipes-tests") ? .recipes : .menu
```

waitForExistence instead of sleep:
```swift
public func recipe() throws -> Recipe {
    let element = scrollViews["Ingredients View"]
    if !element.waitForExistence(timeout: 5) {
        throw FrutaError.elementDoesNotExist("Ingredients View scroll view")
    }
    return Recipe(app: self, element: element)
}
```

runActivity with XCTAttachment on failure:
```swift
public func verify(ingredients: [String]) throws {
    try XCTContext.runActivity(named: "Verifying \(ingredients) in Recipe screen") { activity in
        for ingredient in ingredients {
            if !element.switches[ingredient].waitForExistence(timeout: 5) {
                let attachment = XCTAttachment(string: element.debugDescription)
                activity.add(attachment)
                throw RecipeError.ingredientDoesNotExist(ingredient)
            }
        }
    }
}
```

XCTUnwrap to safely unwrap optionals:
```swift
let favs = try XCTUnwrap(favorites, "favorites is nil — nothing to count")
```

## Takeaways

- Set `continueAfterFailure = false` and use `setUpWithError()` for clean, focused test setup; launch the app fresh per test to eliminate state leakage.
- Model the app's UI hierarchy in test helper objects with typed wrapper classes — tests read like prose, string changes require one edit, and code is reusable across test classes.
- Always include contextual messages in `XCTAssert*` calls; use `waitForExistence(timeout:)` for async elements; use `XCTUnwrap` for optionals so tearDown always runs.
- Shared helper methods should throw, not assert — callers decide what a failure means, enabling both positive and negative testing from the same code path.
- `XCTContext.runActivity` + `XCTAttachment` turn the result bundle into a self-contained failure report: the failure, its context, and supporting files all in one place.

---
_Source: WWDC20 Session 10091 page (abstract, transcript, code samples, and resource links)._
