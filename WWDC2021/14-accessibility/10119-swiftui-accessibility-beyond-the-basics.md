# SwiftUI Accessibility: Beyond the Basics
**WWDC21 · Session 10119** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10119/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, watchOS 8, tvOS 15

## Overview
This session goes beyond default SwiftUI accessibility to demonstrate advanced techniques for delivering exceptional accessible experiences. It introduces Xcode 13's new Accessibility Preview in SwiftUI Previews, which lets developers inspect accessibility elements in real time without running the app, dramatically improving the iteration cycle for accessibility work.

The session walks through a sample finance app called "Wallet Pal" to illustrate five key areas: making custom controls accessible, handling accessibility with children, auditing navigation problems, adding VoiceOver rotors, and managing accessibility focus. Each area is addressed with newly introduced or enhanced APIs in SwiftUI.

A major theme is that custom views—especially those built with `Shape`, `Canvas`, or custom button styles—are not accessible by default and require explicit work. The new `accessibilityRepresentation` and `accessibilityChildren` modifiers make this straightforward and composable.

## Key Topics

### Xcode 13 Accessibility Preview
SwiftUI Previews in Xcode 13 ship with an Accessibility Preview panel that shows accessibility elements in sorted order, their labels, traits, values, and actions in real time. This allows developers to audit accessibility without running on a device or knowing every assistive technology deeply.

### Making Custom Controls Accessible with `accessibilityRepresentation`
Custom controls built from `Shape` or gesture-driven views are not accessible by default. The new `accessibilityRepresentation(representation:)` modifier lets one view's accessibility be entirely defined by another view—for example, wrapping a custom slider shape so it is perceived as a standard `Slider` by assistive technologies.

### Accessibility Children with `accessibilityChildren`
The new `accessibilityChildren(children:)` modifier transforms an accessibility element into an accessibility container, allowing custom children (e.g., individual bars in a `Canvas`-drawn chart) to be defined separately from what is drawn visually. Children can later be combined using the `.combine` child behavior.

### Navigation Grouping and Sort Order
`accessibilityElement(children:)` with the `.contain` behavior groups elements into accessibility containers, fixing sort order. `accessibilitySortPriority(_:)` fine-tunes the order within a container. The `.combine` behavior merges children into a single element and promotes button actions to accessibility actions.

### VoiceOver Rotors
`accessibilityRotor(_:entries:)` adds custom named rotors (e.g., "Warnings") to a container, enabling VoiceOver users to jump between specific elements. `accessibilityRotorEntry` explicitly marks elements for inclusion in a rotor when views are not directly identified by a `ForEach`. A text-range variant enables rotors over specific strings in a `TextEditor`.

### Accessibility Focus with `AccessibilityFocusState`
The new `AccessibilityFocusState` property wrapper reads the current assistive-technology focus and allows programmatic focus changes. It is bound to a view via `accessibilityFocused(_:)`. The session emphasizes that programmatic focus changes should be used sparingly to avoid disorienting users.

## APIs & Frameworks

- `accessibilityRepresentation(representation:)` **[NEW]** — define a view's accessibility via another view
- `accessibilityChildren(children:)` **[NEW]** — set children of an accessibility container using a `ViewBuilder`
- `AccessibilityFocusState` **[NEW]** — property wrapper to read/drive assistive-technology focus
- `accessibilityFocused(_:)` **[NEW]** — binds an `AccessibilityFocusState` binding to a view
- `accessibilityRotor(_:entries:)` **[NEW]** — adds a named VoiceOver rotor with array of entries
- `accessibilityRotor(_:textRanges:)` **[NEW]** — adds a rotor over text ranges in a `TextEditor`
- `accessibilityRotorEntry` **[NEW]** — marks a specific view for inclusion in a rotor by namespace/ID
- `accessibilityElement(children:)` — existing modifier; `.contain`, `.combine`, `.ignore` behaviors
- `accessibilitySortPriority(_:)` — controls sort order within an accessibility container
- `accessibilityAddTraits(_:)` — adds traits such as `.isHeader`, `.isStaticText`
- `AccessibilityChildBehavior` — `.contain`, `.combine`, `.ignore`
- `Canvas` — drawing API; shapes inside Canvas are not accessible by default
- `AccessibilityTraits.isHeader` — marks a view as a heading for the headings rotor
- `Section` — automatically applies `.isHeader` to its header view
- `AccessibilityNotification` — post announcements to VoiceOver (e.g., for low-priority notifications)
- `onChange(of:perform:)` — used in conjunction with `AccessibilityFocusState` to trigger focus

## Code Highlights

Making a custom slider accessible via representation:
```swift
BudgetSliderShape(value: $budget)
    .accessibilityRepresentation {
        Slider(value: $budget, in: 0...1000) {
            Text("Budget: \(budget, format: .currency(code: "USD"))")
        }
    }
```

Setting children on a Canvas-drawn chart:
```swift
Canvas { context, size in /* draw bars */ }
    .accessibilityLabel("Budget History")
    .accessibilityChildren {
        HStack {
            ForEach(budgets) { budget in
                Rectangle().accessibilityLabel(budget.label)
            }
        }
    }
```

Adding a custom VoiceOver rotor:
```swift
AlertsView()
    .accessibilityElement(children: .contain)
    .accessibilityRotor("Warnings") {
        ForEach(alerts.filter { $0.isWarning }) { alert in
            AccessibilityRotorEntry(alert.title, id: alert.id)
        }
    }
```

## Takeaways

- Custom shapes and `Canvas`-drawn views are not accessible by default; use `accessibilityRepresentation` or `accessibilityChildren` to expose them.
- Xcode 13's Accessibility Preview lets you audit element order and properties without a device.
- `accessibilityElement(children: .combine)` reduces noise and promotes actions, making list rows far easier to navigate.
- Use `AccessibilityFocusState` and custom rotors to deliver a supercharged navigation experience, but move programmatic focus only when user context warrants it.

---
_Source: WWDC21 Session 10119 page (abstract, chapter summaries, code samples, and resource links)._
