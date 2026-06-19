# Accessibility in SwiftUI
**WWDC19 · Session 238** · [Watch](https://developer.apple.com/videos/play/wwdc2019/238/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15, watchOS 6, tvOS 13

## Overview
SwiftUI deeply integrates accessibility from the ground up, automatically generating accessibility elements alongside the views it creates. Developers get a large baseline of accessibility support — correct labels, types, actions, and state change notifications — without writing any accessibility-specific code. This session explains what constitutes a great accessibility experience (understandable, interactable, and navigable elements) and then demonstrates exactly how SwiftUI achieves this automatically.

When automatic support is not enough, SwiftUI introduces a new unified Accessibility API — consistent across all Apple platforms — that lets developers add labels, values, hints, traits, custom actions, and structural groupings to refine the Accessibility Tree their app exposes to VoiceOver, Voice Control, Full Keyboard Access, and other assistive technologies.

The session covers the Accessibility Tree structure in depth, showing how combining child elements at a parent node (e.g., collapsing table-cell buttons into custom actions) and setting sort priorities can dramatically simplify navigation and reduce cognitive load for assistive technology users.

## Key Topics

### What Makes a Great Accessibility UI
- **Understandable** — elements have meaningful labels, values, and hints (not raw asset filenames).
- **Interactable** — appropriate default actions and custom named actions wherever needed.
- **Navigable** — correct ordering, grouping via `.accessibilityElement(children: .combine)`, and header traits.

### Automatic Accessibility in SwiftUI
- SwiftUI generates accessibility elements (not UIView/NSView) automatically for every view it renders.
- Standard controls ship with fully correct labels, types, and actions out of the box.
- State-driven notifications: SwiftUI tracks `@State` / `@Binding` changes and automatically posts accessibility notifications — developers never call `UIAccessibility.post(notification:)` manually.
- Custom control styles (`ButtonStyle`, etc.) preserve full accessibility even with completely custom drawing.

### Accessible Images
- Use the label-based `Image(_:label:)` initializer to attach a localizable, human-readable description to non-decorative images.
- Use `Image(decorative:)` for purely decorative images — SwiftUI hides them from the accessibility tree entirely.

### Labeled Controls
- SwiftUI pickers and most controls accept a built-in label parameter; SwiftUI links the visible label to the control as an accessibility relationship automatically.

### SwiftUI Accessibility API
- `.accessibility(label:)` / `.accessibility(value:)` / `.accessibility(hint:)` — set describing information.
- `.accessibility(addTraits:)` / `.accessibility(removeTraits:)` — add/remove `AccessibilityTraits` such as `.isButton`, `.isHeader`, `.isSelected`.
- `.accessibilityAction(named:_:)` — add named custom actions discoverable in all assistive products.
- `.accessibility(hidden:)` — hide an element (and its children) from the accessibility tree.
- `.accessibilityElement(children:)` with `.combine` — merge child elements into a single parent element; child button actions become custom actions automatically.
- `.accessibility(sortPriority:)` — override default Z-order/document-order traversal sequence.

### Accessibility Tree
- The view tree maps directly to an accessibility subtree; SwiftUI subtrees interoperate with UIKit/AppKit trees via `UIViewRepresentable`/`NSViewRepresentable`.
- Grouping related UI (e.g., table cells with name + Follow/Share buttons) at a higher node greatly reduces the number of top-level elements and improves navigation speed.

## APIs & Frameworks

**SwiftUI Accessibility API** (all **[NEW]** in iOS 13 / macOS 10.15)
- `View.accessibility(label:)` **[NEW]**
- `View.accessibility(value:)` **[NEW]**
- `View.accessibility(hint:)` **[NEW]**
- `View.accessibility(addTraits:)` **[NEW]**
- `View.accessibility(removeTraits:)` **[NEW]**
- `View.accessibility(hidden:)` **[NEW]**
- `View.accessibility(sortPriority:)` **[NEW]**
- `View.accessibilityAction(named:_:)` **[NEW]**
- `View.accessibilityElement(children:)` **[NEW]** — `.combine`, `.contain`, `.ignore`
- `AccessibilityTraits` **[NEW]** — `.isButton`, `.isHeader`, `.isSelected`, `.isImage`, `.isLink`, `.playsSound`, `.startsMediaSession`, `.allowsDirectInteraction`, `.causesPageTurn`, `.isKeyboardKey`, `.isStaticText`, `.isSummaryElement`, `.updatesFrequently`, `.isSearchField`
- `Image.init(_:label:)` **[NEW]** — label-based initializer for accessible images
- `Image.init(decorative:)` **[NEW]** — marks image as decorative (excluded from accessibility tree)
- `ButtonStyle` / `PrimitiveButtonStyle` — custom style protocol preserving automatic accessibility
- `Toggle`, `Picker`, `Slider`, `Button` — standard controls with built-in accessible labels and values

**Accessibility Inspector** (Xcode tool) — referenced for debugging accessibility trees

## Code Highlights

```swift
// Setting a custom label and value on a result display
Text(contrastRatio)
    .accessibility(label: Text("Contrast ratio"))
    .accessibility(value: Text("11.7 to 1"))

// Adding an isHeader trait
Text("Background")
    .accessibility(addTraits: .isHeader)

// Hiding a decorative label and exposing its info through the slider
Text("Red")
    .accessibility(hidden: true)
Slider(value: $red, in: 0...255)
    .accessibility(label: Text("Red"))
    .accessibility(value: Text("\(Int(red))"))

// Custom action for a gesture
ContrastRatioView()
    .accessibilityAction(named: Text("Swap colors")) { swapColors() }

// Combining a table cell's children into one element
HStack {
    Text(person.name)
    Spacer()
    Button("Follow") { ... }
    Button("Share") { ... }
}
.accessibilityElement(children: .combine)

// Adjusting sort priority
Button("Reset") { ... }
    .accessibility(sortPriority: 1)   // scanned before lower-priority elements
```

## Takeaways
- SwiftUI's declarative, state-driven model enables automatic accessibility notifications and correct element generation with zero extra code for the common case.
- The unified Accessibility API (same modifiers on every Apple platform) means developers learn it once and apply it everywhere.
- Combining child elements with `.accessibilityElement(children: .combine)` is one of the highest-leverage techniques for improving navigation in list/table UIs.
- Always test with real assistive technologies (VoiceOver, Voice Control, Full Keyboard Access) — the Accessibility Inspector is a supplement, not a replacement.

---
_Source: WWDC19 Session 238 page (abstract, chapter summaries, code samples, and resource links)._
