# Migrate to Swift Testing
**WWDC26 · Session 267** · [Watch](https://developer.apple.com/videos/play/wwdc2026/267/)

_Platforms:_ iOS, iPadOS, macOS, tvOS, visionOS, watchOS, Swift toolchain (Linux, cross-platform)

## Overview
This session provides a practical, step-by-step guide for migrating an existing XCTest suite to Swift Testing. The central theme is that migration should be incremental and fearless: you do not have to convert everything at once. New tests can be written in Swift Testing immediately, alongside existing XCTest cases, thanks to the new bidirectional test framework interoperability introduced in this release.

The session walks through the four interoperability modes available in Xcode Test Plans and Swift packages — Limited, Complete, Strict, and None — explaining when each is appropriate during a migration. It then covers the most common translation patterns developers encounter: replacing `XCTSkip`, converting `continueAfterFailure`, refactoring assertion helpers, migrating loop-based tests to parameterized tests, and adding coverage for crash-path tests using exit tests.

Swift Testing is open-source and runs cross-platform, so the patterns shown apply equally to server-side Swift, Linux CI pipelines, and any other environment where the Swift toolchain is available.

## Key Topics

### Swift Testing Basics
A quick reference: `@Test` replaces `func test*()`, the `Testing` module is imported alongside `@testable import`, `#expect` replaces `XCTAssert*`, and `#require` replaces `XCTAssertNotNil` + `guard`. Raw-identifier test names (backtick syntax) allow human-readable test names with spaces and special characters.

### Migration Strategy
The recommended approach is additive: leave all existing XCTests in place and begin writing new tests in Swift Testing right away. There is no need to mass-convert, and the interoperability feature bridges the two frameworks so shared test helpers continue to work regardless of which framework hosts the test.

### Test Framework Interoperability
The new interoperability feature allows calling XCTest APIs (`XCTAssert*`, `XCTFail`) from within a Swift Testing `@Test` function, and calling Swift Testing APIs (`#expect`, `#require`, `Issue.record`) from within an `XCTestCase` method. This is essential for shared helper functions during incremental migration.

### Interoperability Modes
| Mode | Description |
|---|---|
| **Limited** (default) | Basic cross-framework helper calls work; strict isolation otherwise. |
| **Complete** | Full bidirectional interoperability; any XCTest or Swift Testing API can be called from either framework. |
| **Strict** | Enforces that every test uses only its own framework's APIs; useful as a finishing-line check. |
| **None** | No interoperability; fully isolated suites. |

Configure in Xcode via Test Plans, or for Swift packages via the `SWIFT_TESTING_XCTEST_INTEROP_MODE` environment variable (e.g., `strict` mode: `SWIFT_TESTING_XCTEST_INTEROP_MODE=strict swift test`).

### Common Migration Patterns

**Skipping tests:**
- `XCTSkipIf` / `XCTSkipUnless` → `try Test.cancel(_:)` for imperative skipping during migration.
- Prefer the `@Test(.enabled(if:))` trait as the final idiomatic form.

**Halting after failure:**
- `continueAfterFailure = false` behavior is replicated by `try #require(...)`, which throws on failure and stops the test body.

**Assertion helpers:**
- Replace `XCTFail(_:file:line:)` with `Issue.record(_:sourceLocation:)`. The `SourceLocation` default argument uses `#fileID`/`#line` under the hood, but the call site does not need to pass these explicitly.

### Parameterized Tests
Nested `for` loops in `XCTestCase` test methods are a prime migration target. Refactor them into `@Test(arguments:)` parameterized tests for parallel execution, granular failure reporting per argument combination, and cleaner test names.

```swift
// Before
for bird in Aviary.birds { for count in (40...100) { try await bird.flapWings(count: count) } }

// After
@Test(arguments: Aviary.birds, 40...100)
func `Birds flap wings successfully`(bird: Bird, count: Int) async throws {
    try await bird.flapWings(count: count)
}
```

### Exit Tests
Swift Testing's `#expect(processExitsWith:)` runs a closure in a child process and asserts the process exits with the specified exit status. This is the idiomatic way to test code paths that call `preconditionFailure`, `fatalError`, or trigger other expected process terminations — something XCTest had no equivalent for.

```swift
@Test func `Bird with empty name crashes`() async throws {
    await #expect(processExitsWith: .failure) { _ = Bird(name: "") }
}
```

## APIs & Frameworks

**Swift Testing — core**
- `@Test` macro
- `#expect(_:_:)` macro
- `#require(_:_:)` macro (throwing, halts test on failure)
- `Testing` module import

**Swift Testing — new/changed**
- **[NEW]** `Issue.record(_:sourceLocation:)` — replaces `XCTFail` in shared helpers
- **[NEW]** `Test.cancel(_:)` — imperative test skip/cancel
- **[NEW]** `@Test(.enabled(if:, _:))` trait — declarative conditional enablement
- **[NEW]** `#expect(processExitsWith:)` exit test macro
- **[NEW]** `SourceLocation` type (default argument, replaces `file:line:` pairs)
- **[NEW]** Four interoperability modes: Limited, Complete, Strict, None

**XCTest (interoperability surface)**
- `XCTFail(_:file:line:)` — usable from Swift Testing tests in Complete mode
- `XCTAssertEqual`, `XCTAssertNotNil`, etc. — usable from Swift Testing tests
- `XCTSkipIf(_:_:file:line:)` / `XCTSkipUnless` — migration source
- `continueAfterFailure` — migration source (replaced by `#require`)
- `XCTestCase` — Swift Testing `#expect`/`#require` usable within methods

**Tooling**
- Xcode Test Plans — interoperability mode configuration UI
- `SWIFT_TESTING_XCTEST_INTEROP_MODE` environment variable for `swift test`

## Code Highlights

**Raw-identifier test name:**
```swift
@Test func `Default climate: tropical`() async throws {
    #expect(Fruit(name: "Coconut").climate == .tropical)
}
```

**Shared helper with `Issue.record`:**
```swift
import Testing
func assertUnique(_ fruits: [Fruit], sourceLocation: SourceLocation = #_sourceLocation) {
    for name in fruits.map(\.name) {
        if !uniqueNames.insert(name).inserted {
            Issue.record("Duplicate name: \(name)", sourceLocation: sourceLocation)
        }
    }
}
```

**Exit test for `preconditionFailure`:**
```swift
@Test func `Bird with empty name crashes`() async throws {
    await #expect(processExitsWith: .failure) { _ = Bird(name: "") }
}
```

## Takeaways
- Start writing new tests in Swift Testing today — interoperability removes any blocker to starting.
- Migrate shared assertion helpers first by swapping `XCTFail` for `Issue.record`; this unblocks both old and new test code.
- Convert loop-based tests to parameterized `@Test(arguments:)` functions for significantly better failure isolation and parallel execution.
- Use `#expect(processExitsWith:)` to add crash-path test coverage that was previously impossible in XCTest.

---
_Source: WWDC26 Session 267 page (abstract, chapter summaries, code samples, and resource links)._
