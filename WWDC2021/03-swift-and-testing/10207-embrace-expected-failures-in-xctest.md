# Embrace Expected Failures in XCTest
**WWDC21 · Session 10207** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10207/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, watchOS 8, tvOS 15

## Overview
This session introduces `XCTExpectFailure`, a new XCTest API for managing known but unresolved test failures without disabling tests or generating noisy red results. The core idea is to improve the signal-to-noise ratio in test suites: when a failure is already known and tracked, subsequent failures of the same test produce no new actionable information. `XCTExpectFailure` marks those failures as "expected," letting the test suite pass while preserving the test's continued execution and visibility in the test report.

The session contrasts this API with two common but inferior alternatives—disabling tests (reduces visibility, code stops running) and `XCTSkip` (stops test mid-execution, loses information about additional failures)—and explains when each is appropriate. It also covers the strict default behavior (generates a new failure if the expected failure never occurs), non-strict mode for flaky tests, scoped closures for precise failure regions, issue matchers for selective matching, and platform-conditional enabling.

## Key Topics
- **XCTExpectFailure (stateful):** Call at the top of a test to mark all subsequent failures in that test as expected. The test still runs completely; failures are reported as "expected failure" (gray X) rather than red X. The test suite icon shows a green dash (passed with mixed state).
- **XCTExpectFailure (scoped/closure):** Pass the failing code in a trailing closure to limit the expected failure scope. Failures outside the closure are reported normally.
- **Failure Reason String:** Required parameter; documents the bug and can embed a bug-tracker URL. Xcode shows an "issue tracking" button in the test report when a URL is present.
- **Strict Behavior (default):** If `XCTExpectFailure` is called but no failure occurs, XCTest generates an "unmatched expected failure." This prompts removal of stale `XCTExpectFailure` calls when the bug is fixed.
- **Non-Strict Mode:** Set `isStrict = false` (or pass `strict: false`) to handle flaky/nondeterministic tests—the expected failure call becomes a no-op when no failure occurs.
- **Issue Matcher:** `XCTExpectedFailure.Options.issueMatcher` closure receives an `XCTIssue` and returns `Bool`. Only failures accepted by the matcher are treated as expected.
- **Conditional Enabling:** `XCTExpectedFailure.Options.isEnabled = false` disables the expected failure on platforms where it should not apply.
- **Nesting:** `XCTExpectFailure` calls can nest. When a failure occurs, it is matched against the nearest enclosing call first, then propagated outward with stack semantics. Scoped API is preferred in shared/library code.

## APIs & Frameworks

**XCTest**
- `XCTExpectFailure(_:)` **[NEW]** – Stateful; marks all subsequent failures in the test as expected
- `XCTExpectFailure(_:options:)` **[NEW]** – Stateful with options
- `XCTExpectFailure(_:body:)` **[NEW]** – Scoped; expected failures limited to the closure
- `XCTExpectFailure(_:options:body:)` **[NEW]** – Scoped with options
- `XCTExpectFailure(_:strict:body:)` **[NEW]** – Scoped with inline strict parameter (Swift)
- `XCTExpectedFailure.Options` **[NEW]** – Configuration object for expected failure behavior
  - `issueMatcher: (XCTIssue) -> Bool` – Filter which failures are treated as expected
  - `isEnabled: Bool` – Conditionally disable the expected failure (e.g., per platform)
  - `isStrict: Bool` – Default `true`; if `false`, no failure when expected failure doesn't occur
- `XCTIssue` – Provides failure details to the `issueMatcher` closure (type, description, location)
- `XCTSkip` / `XCTSkipUnless(_:_:)` – Existing API; skips test for configuration-based conditions (compared against `XCTExpectFailure`)
- Test report UI: expected failure indicator (gray X), mixed-state suite indicator (green dash)

## Code Highlights
Stateful expected failure with bug tracker URL:
```swift
XCTExpectFailure("https://dev.myco.com/bugs/4923 myValidationFunction is returning false")
XCTAssert(myValidationFunction())
```

Scoped expected failure:
```swift
XCTExpectFailure("https://dev.myco.com/bugs/4923 fix myValidationFunction") {
    XCTAssert(myValidationFunction())
}
```

With issue matcher (only match assertion failures):
```swift
let options = XCTExpectedFailure.Options()
options.issueMatcher = { issue in issue.type == .assertionFailure }
XCTExpectFailure("https://dev.myco.com/bugs/4923", options: options)
```

Platform-conditional enabling:
```swift
let options = XCTExpectedFailure.Options()
#if os(macOS)
options.isEnabled = false
#endif
XCTExpectFailure("https://dev.myco.com/bugs/4923", options: options) {
    XCTAssert(myValidationFunction())
}
```

Non-strict for flaky tests:
```swift
XCTExpectFailure("https://dev.myco.com/bugs/4923", strict: false) {
    XCTAssert(myValidationFunction())
}
```

## Takeaways
- `XCTExpectFailure` is the correct tool for known, tracked bugs that can't be fixed immediately—tests keep running, failures stay visible, and the suite stays green so real new failures stand out.
- The strict default (unmatched expected failure becomes a real failure) acts as a maintenance reminder: when the bug is fixed, remove the call.
- Use the scoped closure form in shared/library code to avoid polluting surrounding test state via nested matching.
- Non-strict mode (`strict: false`) is appropriate for genuinely flaky, nondeterministic tests where the failure may or may not occur on any given run.

---
_Source: WWDC21 Session 10207 page (abstract, chapter summaries, code samples, and resource links)._
