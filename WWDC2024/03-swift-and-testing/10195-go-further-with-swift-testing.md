# Go Further with Swift Testing
**WWDC24 · Session 10195** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10195/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia 15, watchOS 11, tvOS 18, visionOS 2

## Overview
This session dives deeper into Swift Testing — Apple's new testing framework introduced at WWDC24 — demonstrating advanced features that help teams write more expressive, thorough, and efficient tests. The session builds on the foundational `@Test` and `#expect` macros introduced in "Meet Swift Testing," showing how to handle errors, organize tests with suites and tags, expand coverage via parameterized testing, and write reliable async tests.

The driving philosophy is that good tests are expressive, focused, and easy to run in parallel. Swift Testing makes each of those goals easier to achieve than with XCTest, with less boilerplate and richer diagnostic output on failure.

## Key Topics

**Expectations and Error Validation**
- `#expect(throws:)` validates that a closure throws a specific error — supports checking a general error type, a specific error type, or a specific error value (e.g., `.oversteeped`)
- Unlike `do-catch` patterns, `#expect(throws:)` does not fail the test if an unexpected error escapes; use `#require(throws:)` for the throwing version that stops execution on failure
- Complex validations: `#expect { ... } throws: { error in ... }` — closure receives the thrown error, return `true` to confirm validation passed
- `#require` on optional values: unwraps and stops the test if `nil`; replacing `guard let` + `Issue.record` with `try #require(optionalValue)`

**Tests with Known Issues**
- `withKnownIssue { }` wraps a code block that is expected to fail — keeps the test enabled without marking the whole function `.disabled`
- Only wrap the failing section; code outside the block still runs and can still fail the test
- If the wrapped block unexpectedly passes, the test is marked as a failure (to alert you the issue was fixed)

**Custom Test Descriptions**
- `CustomTestStringConvertible` protocol — implement `var testDescription: String` on argument types so parameterized test output is human-readable
- Xcode and Swift Testing use `testDescription` in test navigator, results, and failure messages

**Parameterized Testing**
- `@Test(arguments: collection)` — runs the test once per element; each run is a separate test case that can pass, fail, or be re-run independently
- Supports `CaseIterable` enums, arrays, and any `Collection`
- Two-argument form: `@Test(arguments: collectionA, collectionB)` — produces a Cartesian product; use `zip(collectionA, collectionB)` to pair elements one-to-one instead
- Avoids `for` loops inside test bodies, which would count as a single test case

**Organizing Tests with Suites**
- `@Suite` marks a struct, class, or actor as a test suite — groups related `@Test` functions under a named container
- Suites can be nested: `@Suite struct Outer { @Suite struct Inner { ... } }` — hierarchy appears in Xcode's test navigator
- Tests can live in separate `@Suite` types across multiple files; they don't need to be co-located

**Tags**
- `@Tag` — declare custom tags as static vars on an extension of `Tag`; apply with `@Test(.tags(.tagName))`
- Tags cross suite and file boundaries; filter and run tagged tests together in Xcode or `swift test --filter`
- Tags appear in Xcode Cloud test plans, enabling selective CI runs by tag
- Multiple tags can be applied to a single test or suite

**Parallel Testing**
- Swift Testing runs all tests in parallel by default — ensures tests don't accidentally share mutable state
- `@Suite(.serialized)` — runs all tests within that suite serially, in source order; inherited by nested suites but not by nested `.serialized` suites' external children
- Use `.serialized` only when tests have genuine sequential dependencies; prefer isolating state in `init`/`deinit` or actors

**Asynchronous Conditions**
- Tests are `async` by default; `await` is always available — no need for XCTestExpectation
- For completion-handler APIs not yet converted to async, use `withCheckedThrowingContinuation`
- `confirmation(expectedCount:)` — new API for callbacks that must fire a specific number of times; replaces unsafe shared mutable counters; `expectedCount: 0` confirms the callback never fires

## APIs & Frameworks

**Swift Testing**
- `@Test` — marks a function as a test case; supports traits like `.tags()`, `.disabled`, display name string **[NEW]**
- `@Suite` — marks a type as a test suite; supports display name string and traits **[NEW]**
- `#expect(throws:) { }` — validates an expression throws a matching error **[NEW]**
- `#expect { } throws: { error in }` — validates thrown error with a custom closure returning `Bool` **[NEW]**
- `#require(throws:) { }` — like `#expect(throws:)` but stops execution on failure **[NEW]**
- `try #require(optional)` — unwraps optional or stops test with a failure **[NEW]**
- `withKnownIssue { }` — wraps code expected to fail; failure doesn't propagate to the test result **[NEW]**
- `@Test(arguments: collection)` — parameterized test with a single collection **[NEW]**
- `@Test(arguments: collectionA, collectionB)` — parameterized test with Cartesian product **[NEW]**
- `@Test(arguments: zip(a, b))` — parameterized test with paired (zipped) arguments **[NEW]**
- `CustomTestStringConvertible` — protocol; implement `testDescription: String` for readable argument labels **[NEW]**
- `@Tag` — declare a custom tag on `extension Tag` **[NEW]**
- `.tags(Tag...)` — trait for `@Test` or `@Suite` to apply tags **[NEW]**
- `.serialized` — suite trait; runs tests within the suite in serial, source-order **[NEW]**
- `.disabled` — trait to skip a test (prefer `withKnownIssue` for temporarily failing tests)
- `Issue.record(_:)` — explicitly record a test failure
- `confirmation(expectedCount:) { }` — async function that validates a callback fires `expectedCount` times **[NEW]**
- `withCheckedThrowingContinuation` — Swift concurrency bridging for completion-handler APIs

**Xcode**
- Xcode test navigator displays suite hierarchy and parameterized test cases individually
- Tag-based filtering in test plans (Xcode and Xcode Cloud)
- Individual parameterized test case re-run from the test navigator

## Code Highlights

Validate a specific error is thrown:
```swift
#expect(throws: BrewingError.oversteeped) {
    try teaLeaves.brew(forMinutes: 200)
}
```

Required unwrap of an optional (stops test if nil):
```swift
let color = try #require(cupOfTea.color)
```

Wrap known-failing code without disabling the whole test:
```swift
withKnownIssue {
    softServeMachine.makeSoftServe(in: .cone)
}
```

Parameterized test over an enum's allCases:
```swift
@Test(arguments: IceCream.Flavor.allCases)
func doesNotContainNuts(flavor: IceCream.Flavor) throws {
    try #require(!flavor.containsNuts)
}
```

Paired parameterized test with zip:
```swift
@Test(arguments: zip(Ingredient.allCases, Dish.allCases))
func cook(ingredient: Ingredient, into dish: Dish) async throws { ... }
```

Tag declaration and application:
```swift
extension Tag {
    @Tag static var caffeinated: Self
}

@Suite(.tags(.caffeinated))
struct DrinkTests { ... }
```

Async confirmation for a callback firing exactly N times:
```swift
try await confirmation("Ate cookies", expectedCount: cookies.count) { ateCookie in
    try await eat(cookies, with: .milk) { _ in ateCookie() }
}
```

Serial suite for order-dependent tests:
```swift
@Suite("Cupcake tests", .serialized)
struct CupcakeTests {
    var cupcake: Cupcake?
    @Test func mixingIngredients() { /* ... */ }
    @Test func baking() { /* ... */ }
    @Test func decorating() { /* ... */ }
}
```

## Takeaways
- Replace `do-catch` error checks with `#expect(throws:)` for cleaner, more informative failure output; use `withKnownIssue` instead of `.disabled` for temporarily broken tests.
- Use parameterized testing with `@Test(arguments:)` instead of `for` loops to get independent, re-runnable test cases per argument.
- Organize tests into `@Suite` types with meaningful names and tags — tags enable cross-suite filtering in Xcode and Xcode Cloud test plans.
- Rely on Swift Testing's default parallelism; reach for `.serialized` only when tests have genuine sequential state dependencies.

---
_Source: WWDC24 Session 10195 page (abstract, chapter summaries, code samples, and resource links)._
