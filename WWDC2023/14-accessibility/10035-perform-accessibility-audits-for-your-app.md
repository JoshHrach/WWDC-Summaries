# Perform accessibility audits for your app
**WWDC23 · Session 10035** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10035/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, tvOS 17, watchOS 10

## Overview
This session introduces automated accessibility auditing directly inside XCTest UI tests, making it possible to catch accessibility issues with every build. A single call to `XCUIApplication.performAccessibilityAudit()` runs the same checks available in the Xcode Accessibility Inspector — covering element descriptions, contrast, dynamic type, and more — and automatically fails the test if any issues are found.

The session also introduces `automationElements`, a new UIKit API that lets developers expose elements specifically for UI test automation without changing what assistive technologies like VoiceOver see. This resolves a longstanding tension where hiding a decorative element from accessibility would also hide it from UI tests.

Together, these two features allow developers to write high-quality UI tests and deliver high-quality accessibility experiences without compromising either goal.

## Key Topics

### Accessibility Audits in XCTest
`XCUIApplication.performAccessibilityAudit()` is a new throwing method that audits the currently visible view for accessibility issues. If any issues are found, the test fails automatically — no manual assertions required. Issues appear inline in the Xcode source editor and in the Report navigator with element screenshots.

### Issue Filtering with a Closure
`performAccessibilityAudit(for:issueHandler:)` accepts an `XCUIAccessibilityAuditType` option set to restrict which audit categories run (e.g., `.dynamicType`, `.contrast`). A closure receives each `XCUIAccessibilityAuditIssue` and returns a `Bool` indicating whether to ignore it. Use `issue.element`, `issue.auditType`, `issue.element.label`, and `issue.element.identifier` to target specific false positives.

### Accessibility Element vs. Accessibility Identifier
Setting an accessibility label to a non-human-readable string (e.g., `QUOTE_TEXTVIEW`) causes VoiceOver to read that string aloud. The correct approach is to use `accessibilityIdentifier` for UI test targeting and leave the label empty or meaningful. Audits catch this pattern as a "label is not human-readable" failure.

### Controlling Accessible Elements
`UIView.accessibilityElements` controls which elements assistive technologies can reach. Setting it to exclude decorative views (e.g., background images) prevents VoiceOver from landing on them, but previously also hid them from UI tests.

### Automation Elements (New UIKit API)
`UIView.automationElements` **[NEW]** is a new property parallel to `accessibilityElements`. It controls which elements are exposed to XCTest UI automation independently of what is exposed to accessibility. Setting `automationElements` allows a view to be reachable in UI tests while still being excluded from VoiceOver and other assistive technologies.

### Practical Testing Strategies
- Set `continueAfterFailure = true` before calling `performAccessibilityAudit()` so all issues are reported in a single run.
- Add audits for every distinct view/screen in the app; audits only check what is currently on screen.
- Use `tearDown()` overrides with class-level opt-in/opt-out variables to run audits across many tests automatically.
- Use Test Plans to group accessibility audit tests separately.
- Audits supplement but do not replace manual testing with VoiceOver, Dynamic Type, and other assistive technologies.

## APIs & Frameworks

- **XCTest / XCUIAutomation**
  - `XCUIApplication.performAccessibilityAudit()` — run audit on current view, throws on issues **[NEW]**
  - `XCUIApplication.performAccessibilityAudit(for:issueHandler:)` — filtered audit with closure **[NEW]**
  - `XCUIAccessibilityAuditType` — option set of audit categories **[NEW]**
    - `.dynamicType` — checks dynamic type scaling
    - `.contrast` — checks color contrast ratios
    - `.elementDetection` — checks for elements missing descriptions
    - `.hitRegion` — checks touch target sizes
    - `.sufficientElementDescription` — checks label quality
    - (additional categories matching Accessibility Inspector)
  - `XCUIAccessibilityAuditIssue` — represents a single audit finding **[NEW]**
    - `issue.element: XCUIElement?` — the element with the issue
    - `issue.auditType: XCUIAccessibilityAuditType` — category of the issue
    - `issue.description: String` — human-readable description
  - `XCUIElement.label` — element's accessibility label
  - `XCUIElement.identifier` — element's accessibility identifier
- **UIKit**
  - `UIView.accessibilityElements: [Any]?` — controls elements exposed to assistive technologies
  - `UIView.automationElements: [Any]?` — controls elements exposed to XCTest automation, independent of accessibility **[NEW]**
  - `UIAccessibilityElement.accessibilityLabel` — human-readable label for assistive tech
  - `UIAccessibilityElement.accessibilityIdentifier` — machine identifier for UI tests, not read aloud
- **Accessibility Inspector** (Xcode tool) — manual audit tool; `performAccessibilityAudit()` runs the same checks programmatically

## Code Highlights

Minimal accessibility audit in a UI test:
```swift
func testAccessibility() throws {
    let app = XCUIApplication()
    app.launch()
    try app.performAccessibilityAudit()
}
```

Filtered audit that ignores a known false positive:
```swift
try app.performAccessibilityAudit(for: [.dynamicType, .contrast]) { issue in
    var shouldIgnore = false
    if let element = issue.element,
       element.label == "My Label",
       issue.auditType == .contrast {
        shouldIgnore = true
    }
    return shouldIgnore
}
```

Excluding a decorative image from VoiceOver while keeping it accessible to UI tests:
```swift
// Exclude from assistive technology
view.accessibilityElements = [quoteTextView, newQuoteButton]

// Still expose to UI test automation
view.automationElements = [imageView, quoteTextView, newQuoteButton]
```

## Takeaways

- `XCUIApplication.performAccessibilityAudit()` brings automated accessibility checking into every UI test with a single line of code — there is no excuse not to add it.
- Use `XCUIAccessibilityAuditType` categories and the `issueHandler` closure to surgically filter false positives without disabling entire audit categories.
- Prefer `accessibilityIdentifier` over a non-human-readable `accessibilityLabel` so VoiceOver reads meaningful content while UI tests can still find elements by string.
- The new `UIView.automationElements` property resolves the conflict between hiding decorative views from VoiceOver and needing to find them in UI tests.

---
_Source: WWDC23 Session 10035 page (abstract, chapter summaries, code samples, and resource links)._
