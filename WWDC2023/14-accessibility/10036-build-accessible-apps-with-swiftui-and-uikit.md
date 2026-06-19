# Build Accessible Apps with SwiftUI and UIKit
**WWDC23 · Session 10036** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10036/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, tvOS 17, watchOS 10

## Overview
This session covers a focused set of new accessibility APIs in SwiftUI and UIKit for iOS 17, giving developers better tools to interact with VoiceOver and other assistive technologies. The additions fall into three areas: new traits and actions that improve how assistive technologies interact with custom UI elements, new modifiers for refining the visual representation of accessibility elements in SwiftUI, and a new block-based setter pattern in UIKit for keeping accessibility attributes automatically up to date.

The practical motivating example is a photo editing app with a custom filter toggle, navigation bar buttons with multiple announcements, a zoomable image view, a piano keyboard that requires direct touch, and circular color buttons that need custom accessibility shapes.

## Key Topics

### Toggle Accessibility Trait **[NEW]**
- Custom button-like UI with on/off state needs the `.isToggle` trait so VoiceOver can describe it as "switch button" and give the correct hint ("double-tap to toggle setting").
- SwiftUI: `.accessibilityAddTraits(.isToggle)`.
- UIKit: `view.accessibilityTraits = [.toggleButton]`.

### Accessibility Notifications and Announcement Priority **[NEW]**
- `AccessibilityNotification` — new Swift-native unified API for posting accessibility notifications across SwiftUI, UIKit, and AppKit (replaces `UIAccessibility.post(notification:argument:)`).
- Supports: `.Announcement`, `.LayoutChanged`, `.ScreenChanged`, `.PageScrolled`.
- Announcements can now carry a **priority** — `high`, `default`, or `low`:
  - **High**: interrupts current speech; cannot be interrupted once started.
  - **Default**: interrupts current speech but can be interrupted by new speech.
  - **Low**: queued; only spoken when speech queue is empty, if no newer announcements arrive.
- SwiftUI: set `AttributedString.accessibilitySpeechAnnouncementPriority` on the announcement string, then pass to `AccessibilityNotification.Announcement(_:).post()`.
- UIKit: use `NSAttributedString` with key `UIAccessibilitySpeechAttributeAnnouncementPriority` and `UIAccessibilityPriority` value.

### Accessibility Zoom Action **[NEW]**
- `accessibilityZoomAction(_:)` (SwiftUI) — receive zoom in/out actions from assistive technologies (switch control, Voice Control, VoiceOver) on a view.
- UIKit: add `.supportsZoom` trait; override `accessibilityZoomIn(at:)` and `accessibilityZoomOut(at:)` returning `Bool`.
- Use `AccessibilityNotification.Announcement` inside the action to voice the resulting zoom level.

### Accessibility Direct Touch Options **[NEW]**
- Previously: `.allowsDirectInteraction` trait passed all touch events directly to the app. VoiceOver still spoke labels and activation sounds on initial touch.
- New **direct touch options**:
  - `.silentOnTouch` — VoiceOver stays silent when the element is touched (app provides its own audio feedback, e.g., piano keys).
  - `.requiresActivation` — VoiceOver must activate the element before touch passthrough begins.
- SwiftUI: `.accessibilityDirectTouch(options: .silentOnTouch)`.
- UIKit: set `.allowsDirectInteraction` trait, then `view.accessibilityDirectTouchOptions = .silentOnTouch`.

### Accessibility Content Shape (SwiftUI) **[NEW]**
- Previously, `contentShape(.interaction, shape)` changed both hit testing shape and accessibility shape together.
- New `.accessibility` content shape kind: only updates the accessibility geometry (VoiceOver cursor path), leaving hit testing unaffected.
- Use: `.contentShape(.accessibility, Circle())` — the VoiceOver cursor now follows the circle shape, preventing it from obscuring nearby elements.

### Block-Based Accessibility Attribute Setters (UIKit) **[NEW]**
- Instead of setting a static accessibility attribute value once at setup, provide a closure that is re-evaluated every time the attribute is accessed by an assistive technology.
- Available as `accessibilityValueBlock`, and equivalents for other attributes.
- The closure is called each time VoiceOver (or other AT) moves focus to the element — always reflects current state without manual invalidation.
- Use `[weak self]` to avoid retain cycles.

## APIs & Frameworks

### SwiftUI Accessibility **[NEW]**
- `.accessibilityAddTraits(.isToggle)` — toggle trait for custom on/off controls **[NEW]**
- `AccessibilityNotification.Announcement(_:)` — post an announcement notification **[NEW]**
- `AccessibilityNotification.LayoutChanged` — post layout change notification **[NEW]**
- `AccessibilityNotification.ScreenChanged` — post screen change notification **[NEW]**
- `AccessibilityNotification.PageScrolled` — post page scroll notification **[NEW]**
- `AttributedString.accessibilitySpeechAnnouncementPriority` — `.high`, `.default`, `.low` **[NEW]**
- `.accessibilityZoomAction(_:)` — handle zoom in/out from assistive technologies **[NEW]**
- `.accessibilityDirectTouch(options:)` — direct touch with `.silentOnTouch` or `.requiresActivation` **[NEW]**
- `.contentShape(.accessibility, shape)` — set accessibility-only hit region shape **[NEW]**
- `.accessibilityAddTraits(_:)` — existing API for adding traits
- `.accessibilityLabel(_:)` — existing label modifier
- `.accessibilityValue(_:)` — existing value modifier

### UIKit Accessibility **[NEW]**
- `UIAccessibilityTraits.toggleButton` — toggle trait **[NEW]**
- `UIAccessibilityTraits.supportsZoom` — zoom capability trait **[NEW]**
- `UIAccessibilityTraits.allowsDirectInteraction` — direct touch passthrough trait (existing)
- `UIView.accessibilityDirectTouchOptions` — `.silentOnTouch`, `.requiresActivation` **[NEW]**
- `UIView.accessibilityZoomIn(at:) -> Bool` — override to handle zoom in **[NEW]**
- `UIView.accessibilityZoomOut(at:) -> Bool` — override to handle zoom out **[NEW]**
- `UIView.accessibilityValueBlock: (() -> String?)` — closure-based value attribute **[NEW]**
- `NSAttributedString.Key.UIAccessibilitySpeechAttributeAnnouncementPriority` — priority key for UIKit announcements **[NEW]**
- `UIAccessibilityPriority` — `.high`, `.default`, `.low` **[NEW]**
- `UIAccessibility.post(notification:argument:)` — existing notification posting method

## Code Highlights

SwiftUI toggle trait:
```swift
Button(action: { filter.toggle() }) { Text("Filter") }
    .background(filter ? darkGreen : lightGreen)
    .accessibilityAddTraits(.isToggle)
```

Posting announcements with priority:
```swift
var lowPriorityAnnouncement: AttributedString {
    var str = AttributedString("Camera Loading")
    str.accessibilitySpeechAnnouncementPriority = .low
    return str
}
AccessibilityNotification.Announcement(defaultPriorityAnnouncement).post()
AccessibilityNotification.Announcement(lowPriorityAnnouncement).post()
AccessibilityNotification.Announcement(highPriorityAnnouncement).post()
```

SwiftUI zoom action:
```swift
Image(imageName ?? "")
    .scaleEffect(zoomValue)
    .accessibilityZoomAction { action in
        switch action.direction {
        case .zoomIn: zoomValue += 1.0
        case .zoomOut: zoomValue -= 1.0
        }
        AccessibilityNotification.Announcement("\(Int(zoomValue))x zoom").post()
    }
```

Silent direct touch for piano keys:
```swift
Rectangle()
    .onTapGesture { playSound(sound: soundFile, type: "mp3") }
    .accessibilityDirectTouch(options: .silentOnTouch)
```

UIKit block-based value setter:
```swift
zoomView.accessibilityValueBlock = { [weak self] in
    guard let self else { return nil }
    return isFiltered ? "Filtered" : "Not Filtered"
}
```

## Takeaways
- Add `.isToggle` / `.toggleButton` to any custom two-state control so VoiceOver describes and announces it correctly without custom hints.
- Use `AccessibilityNotification` for platform-agnostic announcement posting; set `.low` priority on informational announcements and `.high` for critical ones to avoid VoiceOver interruption cascades.
- `.accessibilityDirectTouch(options: .silentOnTouch)` is the right choice for any custom UI that provides its own audio feedback (instruments, drawing canvases, games).
- Use `accessibilityValueBlock` in UIKit anywhere the displayed value can change — it eliminates the need to manually call attribute invalidation on every state change.

---
_Source: WWDC23 Session 10036 page (abstract, chapter summaries, code samples, and resource links)._
