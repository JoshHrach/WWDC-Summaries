# Record, Replay, and Review: UI Automation with Xcode
**WWDC25 · Session 344** · [Watch](https://developer.apple.com/videos/play/wwdc2025/344/)

_Platforms:_ iOS 26, iPadOS 26, macOS Tahoe 26, tvOS 26, visionOS 26, watchOS 26

## Overview
This session introduces a new UI automation workflow in Xcode that lets developers record user interactions, replay them as automated test runs, and review results—all without writing XCTest UI testing code by hand. The tooling records taps, scrolls, and other gestures as a replayable script, making it faster to create regression tests and validate UI flows across device configurations.

## Key Topics

### Recording UI Interactions
Xcode's new recording mode captures user interactions directly against a running app on a simulator or device. Each recorded interaction is stored as a structured step (action type, target element, value) rather than raw coordinates, making recordings resilient to layout changes.

### Replaying Recordings
Recorded scripts can be replayed on any simulator or device. Xcode resolves UI elements using accessibility identifiers and semantic element queries, so replays are not tied to a specific screen size. Replay results are captured as a structured test run with pass/fail per step.

### Reviewing Results
Xcode presents replay results in a new UI Automation results viewer. Each step shows the before/after state, a screenshot diff, and whether the element was found and the action succeeded. Failed steps highlight the specific interaction that regressed.

### Integration with XCTest
Recordings can be exported as XCTest `XCUITest` code, allowing teams to add assertions and integrate recordings into CI pipelines. The generated code uses the same `XCUIApplication`, `XCUIElement`, and `XCUIElementQuery` APIs as hand-written tests.

### Accessibility Identifier Best Practices
The session emphasizes that robust recording and replay require meaningful accessibility identifiers and labels. Elements without identifiers fall back to positional matching, which is fragile. Setting `.accessibilityIdentifier` on key interactive elements improves recording reliability.

## APIs & Frameworks

**Xcode UI Automation (new tooling)** **[NEW]**
- Recording mode — captures interactions against simulator/device
- Replay engine — resolves elements by accessibility identifier and semantic query
- Results viewer — step-by-step pass/fail with screenshot diffs

**XCTest**
- `XCUIApplication` — existing; used in generated test code
- `XCUIElement` / `XCUIElementQuery` — existing; recordings export to these APIs
- `XCUIElement.tap()`, `.swipe*()`, `.typeText(_:)` — interaction methods used in exported code

**SwiftUI / UIKit**
- `.accessibilityIdentifier(_:)` (SwiftUI) / `accessibilityIdentifier` property (UIKit) — setting these on interactive elements dramatically improves recording fidelity

## Code Highlights

```swift
// Example of exported XCUITest code from a recording
func testCheckoutFlow() throws {
    let app = XCUIApplication()
    app.launch()

    app.buttons["Add to Cart"].tap()
    app.buttons["Checkout"].tap()

    let totalLabel = app.staticTexts["order-total"]
    XCTAssert(totalLabel.exists)
    XCTAssertEqual(totalLabel.label, "$29.99")
}
```

## Takeaways
- Use Xcode's new recording mode to create UI automation scripts without writing XCUITest code from scratch — record once, replay on any device or simulator.
- Export recordings as XCUITest code and add assertions to build a maintainable regression suite.
- Add `.accessibilityIdentifier` to all key interactive elements to ensure recordings resolve elements correctly across layout changes.
- Integrate replayed UI automation into Xcode Cloud CI workflows using the standard XCTest runner.

---
_Source: WWDC25 Session 344 page (abstract, chapter summaries, and resource links). Note: full transcript was not accessible; summary is based on available preview content and session abstract._
