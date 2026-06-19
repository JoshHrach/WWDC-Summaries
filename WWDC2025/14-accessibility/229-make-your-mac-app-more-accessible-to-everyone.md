# Make your Mac app more accessible to everyone
**WWDC25 · Session 229** · [Watch](https://developer.apple.com/videos/play/wwdc2025/229/)

_Platforms:_ macOS Tahoe 26, iOS 26

## Overview
This session focuses on practical accessibility improvements for Mac apps built with SwiftUI, covering VoiceOver navigation, keyboard accessibility, and the new `accessibilityDefaultFocus` API. The session uses a real-world document-based app as its running example, showing how a handful of targeted modifier additions can transform an app from partially accessible to fully navigable without a mouse.

The session is structured around three problem areas developers commonly encounter: grouping and combining elements so VoiceOver gives useful descriptions, ordering and priority so the focus cursor visits elements in a logical sequence, and custom rotors so power users can jump directly to meaningful content categories. A new iOS 26 / macOS Tahoe 26 API, `.accessibilityDefaultFocus`, lets apps declare which element should receive focus automatically when a view appears.

## Key Topics

### Grouping Elements for VoiceOver
SwiftUI's `.accessibilityElement(children:)` modifier controls whether child views are individually focusable or merged into a single accessible element. Using `.combine` on a card view that contains an image, title, and subtitle makes VoiceOver read them as a single coherent unit rather than three separate stops.

### Labels and Sort Priority
`.accessibilityLabel()` overrides what VoiceOver speaks when focus lands on an element. `.accessibilitySortPriority()` controls the relative order in which elements receive focus within a container — critical for layouts where visual order and view hierarchy order differ.

### Custom Rotors
`.accessibilityRotor()` registers a named rotor that appears when VoiceOver users swipe with two fingers to rotate. `AccessibilityRotorEntry` populates the rotor with specific items, enabling "jump to heading" or "jump to image" navigation patterns.

### Default Focus (New in 2025)
`.accessibilityDefaultFocus()` is a new modifier that nominates an element to receive VoiceOver or keyboard focus when its containing view first appears. This is analogous to `@FocusState` for regular keyboard focus but operates in the accessibility focus system, allowing apps to land VoiceOver on the primary action of a sheet or alert automatically.

### AccessibilityFocusState
`@AccessibilityFocusState(for: .voiceOver)` is a property wrapper that reads and drives the VoiceOver focus position programmatically, enabling focus restoration after data refresh or sheet dismissal.

## APIs & Frameworks

- **SwiftUI Accessibility modifiers:**
  - `.accessibilityElement(children: .contain/.combine/.ignore)` — grouping strategy
  - `.accessibilityLabel(_:)` — spoken label override
  - `.accessibilitySortPriority(_:)` — focus order within containers
  - `.accessibilityRotor(_:entries:)` — custom VoiceOver rotor registration
  - `AccessibilityRotorEntry` — individual entry in a custom rotor
  - `.accessibilityDefaultFocus()` **[NEW]** — nominate element for initial accessibility focus
  - `.accessibilityAction(_:)` — custom actions surfaced in VoiceOver's actions menu
  - `@AccessibilityFocusState(for: .voiceOver)` — read/write VoiceOver focus position

## Code Highlights

```swift
// Combine card subviews into a single VoiceOver element
VStack {
    Image(item.thumbnail)
    Text(item.title)
    Text(item.subtitle)
}
.accessibilityElement(children: .combine)
.accessibilityLabel("\(item.title), \(item.subtitle)")
```

```swift
// New: set default accessibility focus when a sheet appears
struct DetailSheet: View {
    var body: some View {
        VStack {
            Button("Primary Action") { performAction() }
                .accessibilityDefaultFocus()
            Button("Cancel") { dismiss() }
        }
    }
}
```

```swift
// Custom rotor for jumping between headings
.accessibilityRotor("Headings") {
    ForEach(headings) { heading in
        AccessibilityRotorEntry(heading.title, id: heading.id)
    }
}
```

## Takeaways

- Use `.accessibilityElement(children: .combine)` on composite card and list-row views; VoiceOver users should hear one description, not a fragmented list of sub-elements.
- `.accessibilityDefaultFocus()` is the correct way to direct VoiceOver to the primary action of modally presented views — do not rely on view hierarchy order for this.
- Custom rotors are a high-leverage investment for content-rich apps; they let VoiceOver power users navigate in seconds instead of minutes.
- Test with VoiceOver keyboard on macOS (Tab and VO+arrow keys) in addition to trackpad VoiceOver; keyboard-only navigation reveals a different class of ordering bugs.

---
_Source: WWDC25 Session 229 page (abstract, chapter summaries, code samples, and resource links)._
