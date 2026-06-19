# Refine Accessibility for Custom Controls
**WWDC26 · Session 220** · [Watch](https://developer.apple.com/videos/play/wwdc2026/220/)

_Platforms:_ iOS, iPadOS, macOS, visionOS, watchOS, tvOS

## Overview
Custom controls present unique accessibility challenges because their interactive behaviors are implicit in visual design but invisible to assistive technologies like VoiceOver. This session provides a systematic approach to surfacing those behaviors — translating visual cues into explicit labels, values, traits, and actions that assistive technologies can communicate to users.

The session walks through two categories: guiding principles applied to simpler controls (like a custom coffee-dispenser slider), and advanced techniques for complex, multi-dimensional controls (like an equalizer pad and an interactive virtual cat surface). Each example reveals how layering the right accessibility APIs produces a natural, complete experience.

The core insight is that custom controls must bridge the gap between what a sighted user perceives at a glance and what an assistive technology user needs explicitly stated. Failing to do so doesn't just create friction — it makes parts of an app entirely unusable for some people.

## Key Topics

### Guiding Principles (0:01–8:41)
Every custom control needs four things: a clear **label** naming the control, a **value** describing its current state, one or more **traits** that tell assistive technologies what kind of element it is (e.g., adjustable, button), and **actions** that expose what interactions are possible. The session uses a custom coffee-dispenser slider to illustrate each layer step by step, including setting an `accessibilityActivationPoint` to improve spatial accuracy and posting `AccessibilityNotification.Announcement` when values change significantly.

### Complex Controls (8:41–end)
Multi-dimensional pads and highly interactive surfaces require going beyond basic labeling. An equalizer pad with an X/Y axis is demonstrated using `accessibilityActions` with named directional actions (Move Up/Down/Left/Right) in place of the adjustable trait, since the control has two independent axes. For surfaces that depend on raw touch input — like a virtual pet — `accessibilityDirectTouch` with the `.requiresActivation` option lets VoiceOver users opt into passthrough touch after a deliberate activation gesture, preserving the tactile experience without removing standard navigation.

## APIs & Frameworks

### SwiftUI — Accessibility Modifiers
- `.accessibilityElement()` — collapses child views into a single accessible element
- `.accessibilityLabel(_:)` — sets the spoken name of the element
- `.accessibilityValue(_:)` — sets the spoken current value
- `.accessibilityAddTraits(_:)` — adds semantic traits to an element
- `.accessibilityAdjustableAction(_:)` — handles `.increment` / `.decrement` gestures for the `.adjustable` trait
- `.accessibilityActivationPoint(_:)` — overrides where activation (double-tap) is delivered within the element; accepts `UnitPoint` for proportional positioning **[NEW usage highlighted]**
- `.accessibilityActions(_:_:)` — adds named custom actions accessible from the VoiceOver actions rotor
- `.accessibilityDirectTouch(_:)` — enables raw passthrough touch input for interactive surfaces
- `.onChange(of:)` — used to trigger announcements when state changes

### SwiftUI — Accessibility Traits
- `.adjustable` — marks a control as having increment/decrement behavior
- `AccessibilityDirectTouchOptions.requiresActivation` — requires a deliberate activation before passthrough touch begins

### SwiftUI — Accessibility Notifications
- `AccessibilityNotification.Announcement(_:).post()` — posts a spoken announcement for dynamic value changes

### SwiftUI Documentation Resources
- [Accessible controls](https://developer.apple.com/documentation/SwiftUI/Accessible-controls)
- [Accessible descriptions](https://developer.apple.com/documentation/SwiftUI/Accessible-descriptions)
- [Accessibility fundamentals](https://developer.apple.com/documentation/SwiftUI/Accessibility-fundamentals)
- [Creating accessible views](https://developer.apple.com/documentation/SwiftUI/creating-accessible-views)

## Code Highlights

**Adjustable custom slider with label, value, and action:**
```swift
CoffeeSlider(value: coffee)
    .accessibilityElement()
    .accessibilityLabel("Coffee Dispenser")
    .accessibilityValue("\(Int(coffee)) ounces")
    .accessibilityAddTraits(.adjustable)
    .accessibilityAdjustableAction { direction in
        switch direction {
        case .increment: increaseCoffeeAmount()
        case .decrement: decreaseCoffeeAmount()
        }
    }
```

**Activation point for proportional spatial accuracy:**
```swift
CoffeeSlider(value: coffee)
    .accessibilityActivationPoint(UnitPoint(x: 0.5, y: 1 - coffee))
```

**Posting announcements throttled to avoid spam:**
```swift
.onChange(of: coffee) { _, newValue in
    if sufficientTimeSinceLastAnnouncement() && valueHasChanged() {
        cacheLastSpokenValue(newValue)
        AccessibilityNotification.Announcement(newValue).post()
    }
}
```

**Named directional actions for a 2D equalizer pad:**
```swift
EqualizerPad()
    .accessibilityActions("Move Up")    { increaseY(by: 10) }
    .accessibilityActions("Move Right") { increaseX(by: 10) }
    .accessibilityActions("Move Down")  { decreaseY(by: 10) }
    .accessibilityActions("Move Left")  { decreaseX(by: 10) }
```

**Direct touch with required activation for an interactive surface:**
```swift
InteractiveCatSurface()
    .accessibilityLabel("Virtual Cat")
    .accessibilityValue(cat.currentReaction.description)
    .accessibilityDirectTouch([.requiresActivation])
```

## Takeaways
- Every custom control needs label, value, trait, and action — visual design alone conveys none of these to assistive technologies.
- Use `.accessibilityAdjustableAction` for single-axis controls and named `.accessibilityActions` for multi-axis or complex controls.
- `accessibilityActivationPoint` improves precision for controls where spatial position carries meaning (e.g., sliders, drag handles).
- `.accessibilityDirectTouch([.requiresActivation])` is the right tool for surfaces that require raw touch, providing a safe opt-in for VoiceOver users rather than forcing passthrough unconditionally.

---
_Source: WWDC26 Session 220 page (abstract, chapter summaries, code samples, and resource links)._
