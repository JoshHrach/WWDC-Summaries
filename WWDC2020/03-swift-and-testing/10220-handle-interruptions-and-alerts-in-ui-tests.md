# Handle Interruptions and Alerts in UI Tests
**WWDC20 · Session 10220** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10220/)

_Platforms:_ iOS 13.4+, iPadOS 13.4+, macOS 10.15.4+, tvOS 13.4+

## Overview
This session clarifies the distinction between UI interruptions (unexpected, non-deterministic elements that block test interactions) and expected alerts (deterministic, direct results of test actions), and shows the right API for each scenario.

The first half covers `addUIInterruptionMonitor(withDescription:handler:)` — how the handler stack works, when to return `true` vs `false`, and the implicit interruption handler that XCTest provides for free (including new banner notification handling in Xcode 12). The second half covers the correct way to interact with expected alerts using standard `XCUIElement` queries and `waitForExistence(timeout:)`, and how to use `XCUIApplication.resetAuthorizationStatus(for:)` (introduced in Xcode 11.4) to test protected resource permission flows deterministically — including a new HealthKit resource type in Xcode 12.

## Key Topics

**Defining UI Interruptions**
- A UI interruption is any element that unexpectedly blocks access to an element the test needs to interact with
- Key word: unexpected or non-deterministic — the alert may or may not appear depending on app/system state
- An alert that appears in direct response to a deliberate test action (e.g., tapping a Delete button) is NOT an interruption — handle it with standard queries

**Interruption Handler Stack**
- `addUIInterruptionMonitor(withDescription:handler:)` registers a closure on a stack
- When XCTest needs to interact with an element blocked by another element, it invokes handlers in reverse registration order (LIFO)
- Handler returns `true` if it handled the interruption (stops the chain), `false` to pass to the next handler
- Handlers are automatically removed at end of the test, or manually via the returned token
- Multiple handlers can be registered simultaneously

**XCTest Implicit Interruption Handler**
- XCTest provides a built-in bottom-level handler covering common cases:
  - iOS: alerts with Cancel or Default buttons → taps them
  - iOS Xcode 12 **[NEW]**: banner notifications → dismisses them automatically
  - macOS: user permission dialogs → clicks "Don't Allow"; Bluetooth setup assistant → dismisses it

**Handling Expected Alerts**
- Use standard XCUIElement queries + `waitForExistence(timeout:)` to find and validate expected alerts
- Assert specific content (static text labels, button titles) to validate the alert's correctness
- Tap buttons to dismiss and continue test flow
- Do NOT use interruption monitors for deterministic alerts — they bypass validation

**Protected Resource Authorization Testing**
- `XCUIApplication.resetAuthorizationStatus(for:)` introduced in Xcode 11.4 / iOS 13.4 **[NEW]**
- Resets the stored authorization decision so the permission alert appears again on next access
- Resetting may terminate the app process — always call it before `app.launch()`
- Alerts originate from the system process, not the app; queries must reference `app.alerts` (still works via accessibility)
- Xcode 12 / iOS 14 adds `.health` to the supported resource list **[NEW]**

**Supported Protected Resources (XCUIProtectedResource)**
- Cross-platform: `.contacts`, `.calendar`, `.reminders`, `.photos`, `.microphone`, `.camera`, `.location`
- iOS/tvOS: `.bluetooth`, `.keyboardNetwork`
- iOS 14 / Xcode 12 new: `.health` **[NEW]**
- macOS: `.downloads`, `.desktopFolder`, `.documentsFolder`, `.networkVolumes`, `.removableVolumes`

## APIs & Frameworks

### XCTest — UI Interruption Handling
- `XCTestCase.addUIInterruptionMonitor(withDescription:handler:) -> NSObjectProtocol` — registers interruption handler; returns token for manual removal
  - `handler: (XCUIElement) -> Bool` — receives the interrupting element; return `true` if handled
  - `withDescription: String` — human-readable label for debugging
- `XCTestCase.removeUIInterruptionMonitor(_:)` — manually deregister a handler using its token

### XCTest — Element Queries and Interaction
- `XCUIApplication` — represents the app under test
  - `launch()` — launches the app
  - `resetAuthorizationStatus(for:)` **[NEW in Xcode 11.4]** — resets protected resource authorization
- `XCUIElement.waitForExistence(timeout: TimeInterval) -> Bool` — polls until element exists or timeout elapses
- `XCUIElement.tap()` — taps the element
- `XCUIElementQuery` — chains like `.alerts`, `.cells`, `.staticTexts`, `.buttons`, `.images`, `.navigationBars`
- `XCUIElementQuery.firstMatch` — returns first matching element (faster than subscript)

### XCUIProtectedResource (XCTest)
- `.photos`, `.camera`, `.microphone`, `.location`, `.contacts`, `.calendar`, `.reminders` — cross-platform
- `.bluetooth`, `.keyboardNetwork` — iOS/tvOS
- `.health` **[NEW in Xcode 12 / iOS 14]** — HealthKit authorization
- `.downloads`, `.desktopFolder`, `.documentsFolder` — macOS only

## Code Highlights

Interruption handler that retries on a network failure alert:
```swift
addUIInterruptionMonitor(withDescription: "Handle recipe update failures") { element -> Bool in
    let retryButton = element.buttons["Retry"].firstMatch
    if element.elementType == .alert && retryButton.exists {
        retryButton.tap()
        return true
    }
    return false
}
```

Handling an expected alert with validation:
```swift
func testDeleteRecipe() throws {
    let breadCell = cell(recipeName: "Banana Bread")
    deleteCell(breadCell)

    let alert = app.alerts["Delete Recipe"].firstMatch
    XCTAssert(alert.waitForExistence(timeout: 30), "Expected delete confirmation alert")
    XCTAssert(alert.staticTexts["Are you sure you want to delete this recipe?"].exists)

    alert.buttons["Delete"].tap()
    XCTAssertFalse(breadCell.exists)
}
```

Resetting protected resource authorization before testing first-launch flow:
```swift
func testAddingPhotosFirstTime() throws {
    let app = XCUIApplication()
    app.resetAuthorizationStatus(for: .photos)  // may terminate app process
    app.launch()
    // proceed with test — permission alert will appear as if app never asked before
}
```

Resetting health authorization (Xcode 12 / iOS 14):
```swift
func testHealthOnboarding() throws {
    let app = XCUIApplication()
    app.resetAuthorizationStatus(for: .health)  // NEW in Xcode 12
    app.launch()
    // HealthKit authorization sheet will appear
}
```

## Takeaways
- An interruption is anything that non-deterministically blocks a test interaction — use interruption monitors for these. For alerts that your test directly causes, use standard `XCUIElement` queries and `waitForExistence(timeout:)` instead.
- XCTest's implicit interruption handler covers the most common cases automatically (Cancel/Default buttons on iOS, "Don't Allow" on macOS, and banner notifications in Xcode 12) — you only need a custom handler for app-specific logic like Retry.
- `resetAuthorizationStatus(for:)` is the correct way to test first-launch protected resource flows — call it before `launch()` since it may terminate the app process, and remember to account for it in CI where device state persists between runs.
- Xcode 12 adds `.health` to `XCUIProtectedResource`, enabling deterministic testing of HealthKit authorization flows.

---
_Source: WWDC20 Session 10220 page (abstract, transcript, code samples, and resource links)._
