# Triage Test Failures with XCTIssue
**WWDC20 · Session 10687** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10687/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, tvOS 14, watchOS 7

## Overview
Xcode 12 introduces `XCTIssue`, a rich value type that replaces the four discrete parameters (`message`, `file`, `line`, `expected`) previously passed to `XCTestCase.recordFailure(withDescription:inFile:atLine:expected:)`. `XCTIssue` adds an explicit type enumeration, a detailed description field, an optional underlying error, and—most impactfully—a captured, symbolicated call stack. The call stack transforms failure triage in tests that use shared helper functions: instead of a single annotation on the test method with no trace to the actual failure site, Xcode now shows a gray annotation at the call site and a red annotation at the actual assertion line, with the full call stack visible in both the Issue Navigator and the Test Report.

Two new throwing variants of the test lifecycle methods, `setUpWithError()` and `tearDownWithError()`, allow the same idiomatic Swift error throwing that test functions already supported. Swift runtime improvements (iOS/tvOS 13.4, macOS 10.15.4) enable XCTest to capture source locations for thrown errors—removing the need for `do/catch` boilerplate inside tests.

`XCTAttachment` can now be associated directly with an `XCTIssue` (in addition to being added to a test or `XCTContext` activity). This lets developers embed custom diagnostic data—screenshots, raw data, log strings—precisely at the failure that generated them, making the "how and why" questions easier to answer from the Xcode Test Report or a result bundle from CI.

## Key Topics
- **`XCTIssue` type** — encapsulates message, type, detailed description, source code context, call stack, underlying error, and attachments **[NEW]**
- **`XCTIssue.type`** — `XCTIssue.IssueType` enumeration: `.assertionFailure`, `.thrownError`, `.uncaughtException`, `.performanceRegression`, `.system`
- **`XCTIssue.sourceCodeContext`** — `XCTSourceCodeContext` containing `XCTSourceCodeLocation` (file + line) and call stack (`XCTSourceCodeCallStack`)
- **Symbolicated call stacks in Issue Navigator and Test Report** — gray annotation at test call site, red at failure point; clickable frames; Assistant button opens secondary editor
- **`XCTestCase.record(_:)` / `record(_ issue: XCTIssue)`** — new API replacing deprecated `recordFailure(withDescription:inFile:atLine:expected:)` **[NEW]**
- **`setUpWithError()` / `tearDownWithError()`** — throwing lifecycle variants; `setUpWithError` runs before `setUp`, `tearDownWithError` runs after `tearDown` **[NEW]**
- **Swift errors in tests** — test functions throwing an error now report the source code location of the throw (requires iOS/tvOS 13.4+, macOS 10.15.4+)
- **`XCTAttachment` on `XCTIssue`** — attach arbitrary data (images, raw data, strings, etc.) to a specific failure **[NEW capability]**
- **Overriding `record(_ issue:)`** — funnel point for all failures; override to observe, suppress, or modify (add attachments); always call `super` unless suppressing
- **Custom assertions** — create `XCTIssue` directly, set `sourceCodeContext` to call-site location via `#filePath` / `#line`, call `record(_:)`

## APIs & Frameworks

**XCTest**
- `XCTIssue` **[NEW]** — value type encapsulating all failure data
  - `init(type:compactDescription:)` — primary initializer; also longer form with all fields
  - `type: XCTIssue.IssueType` — `.assertionFailure`, `.thrownError`, `.uncaughtException`, `.performanceRegression`, `.system`
  - `compactDescription: String` — short failure message
  - `detailedDescription: String?` — extended description
  - `sourceCodeContext: XCTSourceCodeContext` — file + line + call stack
  - `associatedError: Error?` — underlying Swift error if any
  - `attachments: [XCTAttachment]` — diagnostic data attached to this failure
  - `add(_ attachment: XCTAttachment)` — attach data to the issue
- `XCTSourceCodeContext` **[NEW]** — groups location and call stack
  - `init(location:)` — create context from a single location
  - `location: XCTSourceCodeLocation`
  - `callStack: XCTSourceCodeCallStack`
- `XCTSourceCodeLocation` **[NEW]** — file path + line number
  - `init(filePath:lineNumber:)` — use `#filePath` and `#line` for call-site capture
- `XCTAttachment` — attach arbitrary data to test runs or issues
  - `init(data:)`, `init(string:)`, `init(image:)`, `init(contentsOfFile:)`, etc.
- `XCTestCase.record(_ issue: XCTIssue)` **[NEW]** — record a failure; overridable
- `XCTestCase.recordFailure(withDescription:inFile:atLine:expected:)` **[DEPRECATED]**
- `XCTestCase.setUpWithError() throws` **[NEW]** — throwing setup; runs before `setUp()`
- `XCTestCase.tearDownWithError() throws` **[NEW]** — throwing teardown; runs after `tearDown()`
- `XCTestCase.setUp()` / `XCTestCase.tearDown()` — original non-throwing variants (still valid; use with new methods if needed for inheritance)

**Xcode 12 UI**
- Issue Navigator — shows failure message + full call stack; click frame to jump to source
- Test Report — failure + call stack; Jump button (navigate to source) and new Assistant button (open secondary editor alongside report)
- Gray annotation — failure in a called function (not the annotated line itself)
- Red annotation — actual failure site

## Code Highlights

Custom assertion with attachment and call-site location:
```swift
func assertSomething(about data: Data,
                     file: StaticString = #filePath,
                     line: UInt = #line) {
    if !isValid(data) {
        var issue = XCTIssue(type: .assertionFailure, compactDescription: "Invalid data")
        issue.add(XCTAttachment(data: data))
        let location = XCTSourceCodeLocation(filePath: file, lineNumber: line)
        issue.sourceCodeContext = XCTSourceCodeContext(location: location)
        self.record(issue)
    }
}
```

Override `record` to add a diagnostic attachment to every failure:
```swift
override func record(_ issue: XCTIssue) {
    var issue = issue  // redeclare as var to mutate
    issue.add(XCTAttachment(string: "diagnostic info here"))
    super.record(issue)
}
```

Override `record` to suppress specific failures:
```swift
override func record(_ issue: XCTIssue) {
    if shouldSuppress(issue) { return }
    super.record(issue)
}
```

Throwing test lifecycle:
```swift
override func setUpWithError() throws {
    try super.setUpWithError()
    // Any thrown error is captured with source location
    try prepareDatabaseFixture()
}
```

## Takeaways
- `XCTIssue` and its call stack make failures in shared test helper functions immediately locatable without passing `file:/line:` parameters everywhere; gray/red annotation pairs in Xcode 12 show both the test call site and the actual failure line simultaneously.
- Attach diagnostic data to a specific failure via `XCTIssue.add(_:)`; the attachment appears in the Test Report alongside that exact failure, making "how did it fail?" answerable from CI result bundles.
- Migrate from `recordFailure(withDescription:inFile:atLine:expected:)` (deprecated) to `record(_ issue:)`; override `record` rather than the old method for observing, suppressing, or enriching failures.
- Adopt `setUpWithError()` and `tearDownWithError()` in new test files; thrown errors now include source location on iOS/tvOS 13.4+ and macOS 10.15.4+, eliminating `do/catch` boilerplate in setUp/tearDown.

---
_Source: WWDC20 Session 10687 page (abstract, chapter summaries, code samples, and resource links)._
