# XCTSkip Your Tests
**WWDC20 · Session 10164** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10164/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, Xcode 12 (API available since Xcode 11.4)

## Overview
`XCTSkip` introduces a third test result alongside pass and fail: a **skip**. Before this API, tests that legitimately could not execute in the current environment had two bad options — pass silently (hiding the fact that the test didn't actually run) or fail (raising a false alarm that consumes triage resources). `XCTSkip` produces a distinct, visible skip result that appears in the Xcode Test Navigator, the test report, and CI result bundles with the exact file, line, and reason for the skip.

## Key Topics

### When to Use XCTSkip
`XCTSkip` is appropriate when the test has a runtime dependency that cannot be mocked:
- **Platform/device type** — a test for iPad-only pointer interactions should not run on iPhone
- **OS version** — tests for APIs that require a minimum deployment target (e.g., `UIPointerInteraction` requires iOS 13.4+)
- **External service availability** — a server required by an integration test is temporarily offline
- **Feature flags / debug modes** — stubs for tests not yet implemented; temporarily unfixable tests that should not be disabled completely

Skipped tests remain visible in the test navigator and result bundle so they are not forgotten.

### Three Ways to Skip

1. **`XCTSkipIf(_:_:)`** — skip when the expression is `true`
   ```swift
   try XCTSkipIf(UIDevice.current.userInterfaceIdiom != .pad,
                 "Pointer interaction tests are for iPad only")
   ```

2. **`XCTSkipUnless(_:_:)`** — skip when the expression is `false` (semantically cleaner for "unless condition is met")
   ```swift
   try XCTSkipUnless(UIDevice.current.userInterfaceIdiom == .pad,
                     "Pointer interaction tests are for iPad only")
   ```

3. **`throw XCTSkip(_:)`** — throw the struct directly, typically combined with `guard #available`:
   ```swift
   guard #available(iOS 13.4, *) else {
       throw XCTSkip("Pointer interaction tests can only run on iOS 13.4+")
   }
   ```

All three require the test method to be declared `throws`. The optional message string appears in the result bundle as the skip reason.

### How Xcode Surfaces Skips
- **Source editor** — grey skip icon and an annotation showing where and why the test was skipped
- **Test Navigator** — skip icon next to the test; filter button to show only skipped tests
- **Test Report** — expands to show file, line number, and skip reason for each skipped test; "jump" button navigates to the source; "assistant" button (new in Xcode 12) opens the source in a secondary editor alongside the report
- **CI result bundles** — per-device results show pass/skip breakdown; expanding shows the skip location and reason for each device

### Keyboard Shortcut Tip
Control-Option-Command-G re-runs the previous test action — useful for quickly re-running after changing destinations.

## APIs & Frameworks

### XCTest
- `XCTSkip` struct **[NEW — Xcode 11.4]** — throwable; produces a skip test result; conforms to `Error`
- `XCTSkipIf(_ expression: Bool, _ message: String?, file: StaticString, line: UInt) throws` **[NEW — Xcode 11.4]** — throws `XCTSkip` when expression is true
- `XCTSkipUnless(_ expression: Bool, _ message: String?, file: StaticString, line: UInt) throws` **[NEW — Xcode 11.4]** — throws `XCTSkip` when expression is false
- Skip result displayed in: Test Navigator, Test Report, result bundle, source editor annotation

## Code Highlights

Skip based on device type:
```swift
func testPointerInteraction() throws {
    try XCTSkipUnless(UIDevice.current.userInterfaceIdiom == .pad,
                      "Pointer interaction tests are for iPad only")
    // Test proceeds only on iPad
    let view = app.views["MainView"]
    // ...assert pointer interaction behavior...
}
```

Skip based on OS version:
```swift
func testPointerInteraction() throws {
    guard #available(iOS 13.4, *) else {
        throw XCTSkip("Pointer interaction tests can only run on iOS 13.4+")
    }
    // Test proceeds only on iOS 13.4+
}
```

Skip as a stub for unimplemented tests:
```swift
let featureIsImplemented = false

func testNewFeature() throws {
    try XCTSkipUnless(featureIsImplemented,
                      "Test not yet implemented — tracks pending work")
}
```

## Takeaways

- `XCTSkip` fills the gap between pass (test ran) and fail (test found a bug) — it communicates "this test is valid but could not execute here" without consuming triage resources.
- Use `XCTSkipUnless` for conditions that must be met for the test to be meaningful (device type, OS version); use `throw XCTSkip` with `guard #available` for availability checks.
- Skipped tests remain visible in all Xcode result surfaces — use `XCTSkip` instead of disabling tests or deleting them, so the gap in coverage is not forgotten.
- The optional message string is the skip reason shown in the result bundle; write it so future developers (and CI systems) understand why the test was skipped without reading the source.

---
_Source: WWDC20 Session 10164 page (abstract, transcript, code samples, and resource links)._
