# Explore Digital Crown, Trackpad, and iPad Pointer Automation
**WWDC21 · Session 10208** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10208/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, watchOS 8

## Overview
This session introduces three new XCTest UI automation APIs in Xcode 13, one per platform: iPadOS pointer interaction (hover, click, drag, scroll), watchOS Digital Crown rotation, and macOS continuous trackpad-style scrolling. Together they fill significant gaps in UI test coverage for device-specific input methods that were previously untestable.

The iPadOS pointer API enables automated testing of pointer-specific UI behaviors such as hover states and click-and-drag interactions, which are only meaningful on iPads connected to Magic Keyboard or other pointer accessories. The Digital Crown rotation API enables watchOS apps that respond to crown input to be tested programmatically for the first time. The macOS trackpad API adds inertial, continuous swipe-style scrolling to complement the existing discrete pixel-precise scroll method.

## Key Topics
- **iPadOS Pointer Automation (NEW):** Available on devices running iPadOS 15+. New methods on `XCUIElement` and `XCUICoordinate` for hover, left/right/double click, two-finger scroll, click-and-drag, and modifier-key-held interactions. New `XCUIDevice.supportsPointerInteraction` property to skip tests on non-pointer devices. Annotate test methods with `@available(iOS 15.0, *)` and guard with `XCTSkipUnless(XCUIDevice.shared.supportsPointerInteraction)`.
- **watchOS Digital Crown Rotation (NEW):** New `XCUIDevice.rotateDigitalCrown(delta:velocity:)` method. `delta` is number of crown rotations (positive = forward, negative = backward). `velocity` is `XCUIGestureVelocity` (.slow, .default, .fast, or custom decimal in rotations/sec). Builds on watchOS UI testing support introduced in Xcode 12.5.
- **macOS Continuous Trackpad Scrolling (NEW):** New `swipeUp/Down/Left/Right(velocity:)` methods on `XCUIElement` for continuous, inertial scrolling. `velocity` is `XCUIGestureVelocity` in pixels per second. Distinct from the existing discrete `scroll(byDeltaX:deltaY:)` method which simulates mouse-wheel behavior.
- **Discrete vs. Continuous Scrolling:** Discrete scrolling (existing `scroll(byDeltaX:deltaY:)`) provides exact pixel increments with no inertia—simulates scroll wheel. Continuous scrolling (new `swipeUp/Down/Left/Right`) provides fluid, inertial trackpad-style movement that decelerates after input ends.

## APIs & Frameworks

**XCTest**

_iPadOS Pointer APIs_
- `XCUIDevice.supportsPointerInteraction: Bool` **[NEW]** – Whether the device supports pointer input
- `XCUIElement.hover()` **[NEW]** – Moves pointer over element (triggers hover states)
- `XCUIElement.click()` **[NEW]** – Pointer left-click
- `XCUIElement.rightClick()` **[NEW]** – Pointer right-click
- `XCUIElement.doubleClick()` **[NEW]** – Pointer double-click
- `XCUIElement.scroll(byDeltaX:deltaY:)` **[NEW on iPadOS]** – Two-finger pointer scroll
- `XCUIElement.click(forDuration:thenDragToElement:)` **[NEW]** – Click-and-drag to another element
- `XCUIElement.click(forDuration:thenDragToElement:withVelocity:thenHoldForDuration:)` **[NEW]** – Click-and-drag with velocity and hold
- `XCUIElement.perform(withKeyModifiers:block:)` **[NEW]** – Execute interactions with modifier keys held (e.g., `.shift`, `.command`)
- `XCUICoordinate` – All above pointer methods also available for coordinate-precise targeting **[NEW]**

_watchOS Digital Crown APIs_
- `XCUIDevice.rotateDigitalCrown(delta:velocity:)` **[NEW]** – Simulates Digital Crown rotation
  - `delta: CGFloat` – Rotations; positive = forward, negative = backward
  - `velocity: XCUIGestureVelocity` – Rotation speed (optional, default = `.default`)

_macOS Continuous Scrolling APIs_
- `XCUIElement.swipeUp(velocity:)` **[NEW]** – Continuous upward trackpad swipe
- `XCUIElement.swipeDown(velocity:)` **[NEW]** – Continuous downward trackpad swipe
- `XCUIElement.swipeLeft(velocity:)` **[NEW]** – Continuous leftward trackpad swipe
- `XCUIElement.swipeRight(velocity:)` **[NEW]** – Continuous rightward trackpad swipe
- `XCUIGestureVelocity` – Shared velocity type: `.slow`, `.default`, `.fast`, or custom `CGFloat` (pixels/sec for swipe, rotations/sec for crown)
- `XCUIElement.scroll(byDeltaX:deltaY:)` – Existing discrete scrolling API (unchanged)

## Code Highlights
iPadOS pointer test with platform guard:
```swift
@available(iOS 15.0, *)
func testHorizontalScrollRevealsSidebar() throws {
    try XCTSkipUnless(XCUIDevice.shared.supportsPointerInteraction,
                      "Device does not support pointer interaction")
    let app = XCUIApplication()
    app.launch()
    let sidebar = app.tables["Sidebar"]
    XCTAssertFalse(sidebar.exists, "Sidebar should not be present initially")
    app.staticTexts["Select a smoothie"].scroll(byDeltaX: -200, deltaY: 0)
    XCTAssertTrue(sidebar.waitForExistence(timeout: 5),
                  "Sidebar did not appear within 5 second timeout")
}
```

watchOS Digital Crown rotation test:
```swift
func testForecastScrolling() {
    let app = XCUIApplication()
    app.launch()
    let forecastTime = app.staticTexts["forecast-time"]
    XCTAssertEqual(forecastTime.label, "Current Temperature")
    XCUIDevice.shared.rotateDigitalCrown(delta: 1.0)    // 1 rotation forward
    XCTAssertEqual(forecastTime.label, "One hour from now")
    XCUIDevice.shared.rotateDigitalCrown(delta: -2.0)   // 2 rotations backward
    XCTAssertEqual(forecastTime.label, "One hour ago")
}
```

macOS continuous trackpad swipe:
```swift
// Continuous (trackpad-like, inertial)
scrollView.swipeUp(velocity: .fast)

// Discrete (mouse-wheel-like, no inertia)
scrollView.scroll(byDeltaX: 0, deltaY: -100)
```

## Takeaways
- Always check `XCUIDevice.shared.supportsPointerInteraction` and use `XCTSkipUnless` rather than letting tests fail on incompatible hardware—this produces cleaner CI results.
- Mark pointer-interaction test methods with `@available(iOS 15.0, *)` to satisfy the compiler since these APIs are availability-gated.
- For watchOS crown tests, `delta` values map to full rotations, not arbitrary units—a `delta` of `1.0` scrolls exactly one complete crown rotation forward.
- Use the new `swipeUp/Down/Left/Right` methods (not `scroll(byDeltaX:deltaY:)`) when testing UI that responds specifically to inertial trackpad scrolling on macOS, such as scroll views that implement `NSScrollView` momentum scrolling.

---
_Source: WWDC21 Session 10208 page (abstract, transcript, and code samples)._
