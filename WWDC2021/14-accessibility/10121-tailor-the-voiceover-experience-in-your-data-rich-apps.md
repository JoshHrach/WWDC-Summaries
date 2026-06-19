# Tailor the VoiceOver Experience in Your Data-Rich Apps
**WWDC21 · Session 10121** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10121/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, watchOS 8, tvOS 15

## Overview
When an app presents a lot of data per cell or view — names, statistics, descriptions, ratings — VoiceOver reads everything by default, creating a wall of speech that is cognitively overwhelming. Conversely, silencing too much leaves users with an incomplete picture. This session introduces the Accessibility Custom Content API (`AXCustomContent` / `AXCustomContentProvider`) as the solution: a way to tier information so VoiceOver speaks the most important details immediately and exposes secondary details on demand via the "More Content" rotor.

The API is available in the `Accessibility` framework across all Apple platforms and is now supported in SwiftUI as well as UIKit and AppKit.

## Key Topics

### The Problem: Information Overload vs. Information Drought
Without any customization, VoiceOver reads every label and value in a cell from top-left to bottom-right. A kennel-app cell with name, breed, description, popularity, age, weight, and height produces a lengthy announcement on every swipe. Shortening the `accessibilityLabel` helps but discards data the user might want.

### AXCustomContent and AXCustomContentProvider (UIKit / AppKit)
Conforming a `UITableViewCell` (or any `UIAccessibilityElement`) to `AXCustomContentProvider` requires implementing one property: `accessibilityCustomContent`, which returns an ordered array of `AXCustomContent` objects. Each `AXCustomContent` is a key-value pair of localized strings. VoiceOver announces only the main `accessibilityLabel` on initial focus and says "More content available." The full set of custom content is accessible via the "More Content" rotor.

### The `importance` Property
An `AXCustomContent` object's `importance` property (`.high` or `.default`) controls whether that piece of content is always read inline with the main label (`.high`) or deferred to the rotor (`.default`). Setting `age.importance = .high` causes VoiceOver to always announce the age alongside the name on focus, while the other fields remain rotor-accessible.

### SwiftUI Support (New in 2021)
The `.accessibilityCustomContent(_:_:importance:)` modifier works directly on SwiftUI views, accepting either string keys or typed `AccessibilityCustomContentKey` values. Hide sub-views that would otherwise pollute the combined label using `.accessibilityHidden(true)` before applying custom content modifiers.

### AccessibilityCustomContentKey
To avoid redefining a localized key string in multiple places, extend `AccessibilityCustomContentKey` with a static property. This lets any view in the hierarchy update or remove the value for that key without recreating the string.

### User-Side Configuration
VoiceOver users can configure "More Content" verbosity in iOS VoiceOver Preferences > Verbosity > More Content (speak hints, play sound, or change pitch). An equivalent setting exists in the macOS VoiceOver Utility. On macOS, the More Content menu is opened with VO + Cmd + /.

## APIs & Frameworks

- `Accessibility` framework — top-level framework containing AXCustomContent APIs
- `AXCustomContent` **[NEW]** — a key-value pair (`label: String`, `value: String`) representing one piece of secondary accessibility content
- `AXCustomContent.importance` **[NEW]** — `.high` (always read inline) or `.default` (rotor only)
- `AXCustomContentProvider` protocol **[NEW]** — `accessibilityCustomContent: [AXCustomContent]!` property; conform `UITableViewCell`, `NSView`, etc.
- `.accessibilityCustomContent(_:_:importance:)` SwiftUI modifier **[NEW]** — attaches custom content to a SwiftUI view; accepts `String` key or `AccessibilityCustomContentKey`
- `AccessibilityCustomContentKey` **[NEW]** — typed key for custom content; extend with static properties to avoid duplicate string literals
- VoiceOver "More Content" rotor — built-in rotor entry that surfaces `AXCustomContent` values on demand
- `accessibilityLabel` — override to return a concise primary announcement (name + type only)
- `.accessibilityHidden(true)` — hide sub-views from the accessibility tree to prevent them from inflating the combined label

## Code Highlights

UIKit — basic AXCustomContentProvider conformance:
```swift
import Accessibility

class DogTableViewCell: UITableViewCell, AXCustomContentProvider {
    override var accessibilityLabel: String? {
        get { "\(name.text!), \(type.text!)" }
        set { }
    }

    var accessibilityCustomContent: [AXCustomContent]! {
        get {
            let age = AXCustomContent(label: "Age", value: age.text!)
            age.importance = .high          // always read inline
            let popularity = AXCustomContent(label: "Popularity", value: popularity.text!)
            let weight    = AXCustomContent(label: "Weight",     value: weight.text!)
            let height    = AXCustomContent(label: "Height",     value: height.text!)
            let notes     = AXCustomContent(label: "Description",value: desc.text!)
            return [age, popularity, weight, height, notes]
        }
        set { }
    }
}
```

SwiftUI — custom content with typed key:
```swift
extension AccessibilityCustomContentKey {
    static var age: AccessibilityCustomContentKey {
        AccessibilityCustomContentKey("Age")
    }
}

struct DogCell: View {
    var dog: Dog
    var body: some View {
        VStack { /* ... */ }
            .accessibilityElement(children: .combine)
            .accessibilityCustomContent(.age, dog.age, importance: .high)
            .accessibilityCustomContent("Popularity", dog.popularity)
            .accessibilityCustomContent("Weight",     dog.weight)
            .accessibilityCustomContent("Height",     dog.height)
            .accessibilityCustomContent("Description",dog.description)
    }
}
```

## Takeaways

- Too much VoiceOver output is just as bad as too little; `AXCustomContent` lets you tier data so VoiceOver reads what matters first and defers the rest to the More Content rotor.
- Use `importance = .high` sparingly for the one or two fields that a user absolutely needs without extra interaction.
- The same API works in UIKit, AppKit, and (new this year) SwiftUI with `.accessibilityCustomContent`.
- Use `AccessibilityCustomContentKey` extensions to share keys across views without repeating localized strings.

---
_Source: WWDC21 Session 10121 page (abstract, chapter summaries, code samples, and resource links)._
