# Meet Swift Testing
**WWDC24 · Session 10179** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10179/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia 15, watchOS 11, tvOS 18, visionOS 2

## Overview
Swift Testing is a new open-source framework for writing tests in Swift, built from the ground up to leverage modern Swift language features. It replaces XCTest's class-based, string-named conventions with expressive macros, traits, and value-type test suites. Tests written with Swift Testing produce richer failure messages, support parameterization, and run in parallel by default.

The session introduces the four building blocks of Swift Testing — `@Test` functions, `#expect`/`#require` macros, traits, and `@Suite` types — then demonstrates how they compose to handle conditional tests, tagged test groups, and parameterized tests. It also covers how Swift Testing relates to XCTest and its open-source availability.

## Key Topics

**Building Blocks: @Test Functions**
- `@Test` attribute marks any Swift function (free function or method) as a test
- Test functions can be `async`, `throws`, or both; Swift Testing handles error propagation naturally
- A display name string can be passed: `@Test("Check video metadata")` — shown in Xcode's test navigator and results
- No need to prefix with "test"; any function name works

**Building Blocks: #expect and #require**
- `#expect(condition)` — records a test failure if the condition is false; test continues running
- `#require(condition)` — like `#expect` but stops the current test function on failure (equivalent to a fatal assertion)
- Both macros capture the full expression and show the exact values on failure — far more informative than XCTAssert
- `try #require(optional)` — unwraps an optional or stops the test if nil

**Building Blocks: Traits**
- Traits are arguments to `@Test` or `@Suite` that modify behavior
- `.enabled(if:)` — runtime condition; test is skipped if the condition is false
- `.disabled("reason")` — unconditionally skip; always provide a reason string
- `.bug("url", "description")` — associates a known bug URL with a test
- `@available(macOS X.Y, *)` on a `@Test` function — preferred over `guard #available(...)` inside the function body
- `.tags(Tag...)` — attach semantic labels; filter by tag in Xcode test plans and on the command line

**Building Blocks: @Suite Types**
- Any Swift struct, class, or actor containing `@Test` functions is implicitly a suite
- `@Suite` attribute can be added explicitly to give it a display name or suite-level traits
- Stored properties in a suite type are re-initialized before each test function — safe, isolated state
- Suites can be nested; traits on a parent `@Suite` (like `.tags(...)`) are inherited by all child tests

**Common Workflow: Tests with Conditions**
- Use `.enabled(if: AppFeatures.isCommentingEnabled)` to gate a test on a runtime flag
- Use `@available(macOS 15, *)` (not `#available` inside the body) to scope platform availability

**Common Workflow: Tests with Common Characteristics**
- Group related tests into a nested `@Suite` struct; share setup via stored properties
- Apply `.tags(.formatting)` on a `@Suite` to tag all its tests at once
- Tags cross suite and file boundaries — useful for CI filtering

**Common Workflow: Tests with Different Arguments (Parameterized Tests)**
- `@Test(arguments: collection)` — runs the test once per element in the collection, as separate test cases
- Each parameterized case can pass, fail, or be re-run independently in Xcode
- Replaces `for` loops inside test bodies, which count as a single test case regardless of how many iterations fail

**Swift Testing and XCTest**
- Swift Testing can coexist with XCTest in the same project and target
- XCTest methods (`test*()`) still run alongside `@Test` functions
- Migrate gradually — no need to rewrite all XCTest at once
- XCTestCase subclasses and setUp/tearDown remain supported

**Open Source**
- Swift Testing is open source at github.com/swiftlang/swift-testing
- Runs anywhere Swift runs, including Linux
- Run tests from the Terminal: `swift test`

## APIs & Frameworks

**Swift Testing** **[NEW]**
- `@Test` **[NEW]** — marks a function as a test; accepts an optional display name string and traits
- `@Suite` **[NEW]** — marks a type as a test suite; accepts an optional display name string and traits
- `#expect(_:)` **[NEW]** — records a failure if the expression is false; test continues
- `#require(_:)` **[NEW]** — records a failure and stops the test if the expression is false
- `try #require(optional)` **[NEW]** — unwraps an optional or stops the test
- `.enabled(if:)` **[NEW]** — trait; conditionally enable a test at runtime
- `.disabled(_:)` **[NEW]** — trait; unconditionally disable a test with a reason string
- `.bug(_:_:)` **[NEW]** — trait; associate a bug URL and title with a test
- `.tags(Tag...)` **[NEW]** — trait; apply semantic tags to a test or suite
- `Tag` **[NEW]** — namespace for custom tag declarations via `extension Tag { @Tag static var myTag: Self }`
- `@Test(arguments: collection)` **[NEW]** — parameterized test; runs once per element
- `Issue.record(_:)` — explicitly record a failure (advanced use)
- `swift test` — command-line test runner for Swift packages

## Code Highlights

Minimal `@Test` function with `#expect`:
```swift
import Testing

@Test("Check video metadata")
func videoMetadata() throws {
    let video = Video(fileName: "By the Lake.mov")
    let expectedMetadata = Metadata(duration: .seconds(19))
    #expect(video.metadata == expectedMetadata)
}
```

Stop the test early with `#require`:
```swift
let video = try await #require(videoLibrary.video(named: "By the Lake"))
#expect(video.formattedDuration == "0m 19s")
```

Conditional and disabled traits:
```swift
@Test(.enabled(if: AppFeatures.isCommentingEnabled))
func videoCommenting() { ... }

@Test(.disabled("Due to a known crash"), .bug("example.org/bugs/1234", "Program crashes at <symbol>"))
func example() { ... }
```

Suite with shared stored property and tag inheritance:
```swift
@Suite(.tags(.formatting))
struct MetadataPresentation {
    let video = Video(fileName: "By the Lake.mov")

    @Test func rating() async throws {
        #expect(video.contentRating == "G")
    }
}
```

Parameterized test replacing repetitive test functions:
```swift
@Test("Number of mentioned continents",
      arguments: ["A Beach", "By the Lake", "Camping in the Woods"])
func mentionedContinentCounts(videoName: String) async throws {
    let videoLibrary = try await VideoLibrary()
    let video = try await #require(videoLibrary.video(named: videoName))
    // ...
}
```

## Takeaways
- Replace XCTest methods incrementally — Swift Testing coexists in the same target; no big-bang migration needed.
- Use `#expect` for non-fatal checks and `try #require` to safely unwrap optionals and stop on failure, rather than force-unwrapping.
- Prefer `@available` on `@Test` functions over `guard #available` inside them — Xcode shows these tests as skipped rather than silently passing.
- Parameterize tests with `@Test(arguments:)` instead of `for` loops — each argument becomes an independent, re-runnable test case.

---
_Source: WWDC24 Session 10179 page (abstract, chapter list, code samples, and resource links)._
