# Swan's Quest, Chapter 1: Voices in the dark
**WWDC20 · Session 10681** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10681/)

_Platforms:_ iPadOS 14, macOS Big Sur 11 (Swift Playgrounds)

## Overview
Swan's Quest is a four-chapter interactive programming challenge delivered through Swift Playgrounds, designed to teach Apple platform technologies through storytelling and progressive coding exercises. Chapter 1 uses accessibility as its core topic: players must write VoiceOver accessibility labels for torches in a dark cave to allow a turtle character to navigate through it. The chapter covers VoiceOver basics (gestures, speaking rate, cursor navigation) and the Swift Playgrounds content SDK's `AccessibilityHints` API for making on-screen elements readable by assistive technologies.

The Swift Playgrounds content SDK used in Swan's Quest is open-source and available to any developer who wants to create educational playground books. It includes modules for audio, accessibility, AR, and sensor input — the same frameworks powering Apple's own guided learning experiences such as Sonic Workshop and Sensor Arcade.

## Key Topics

### VoiceOver Basics
- VoiceOver is Apple's built-in screen reader, available on all Apple platforms — uses text-to-speech, braille, or both
- Activate via: Settings → Accessibility → VoiceOver toggle, or Siri command "Turn on VoiceOver", or triple-click Home/Sleep button (after setting Accessibility Shortcut)
- iPad gestures: swipe left/right with one finger to move cursor between elements; drag finger to scan quickly; double-tap to activate a focused element; swipe up/down on a slider to adjust values (e.g., speaking rate)
- Mac: Command-F5 or triple-click Touch ID to activate; Control-Option + arrow keys to navigate; Control-Option-Shift-Down to interact with a group; Space to activate
- VoiceOver cursor: black rectangle around the focused element
- Speaking rate: adjustable via Speaking Rate slider; experienced users often use speeds up to 720+ words per minute
- Common failure mode in apps: unlabeled buttons that VoiceOver reads as "button" or the image filename — results in unusable experiences for VoiceOver users

### Swift Playgrounds Content SDK — Accessibility
- `BaseGraphic` (superclass of `Graphic`, `Button`, `Sprite`, `Label`) — all content SDK on-screen elements
- `BaseGraphic.accessibilityHints: AccessibilityHints?` — optional property; set this to make an element VoiceOver-accessible
- `AccessibilityHints` struct:
  - `makeAccessibilityElement: Bool` — when `true`, VoiceOver stops the cursor on this element
  - `accessibilityLabel: String?` — the string VoiceOver reads aloud when the cursor lands on this element
- Setting a descriptive, meaningful label (e.g., "A 12-inch rock blazing with a ball of mystical blue flame") instead of a generic one ("Torch one") is the challenge's core lesson

### Swan's Quest Structure
- Four-part progressive challenge; each chapter builds on the prior one's APIs
- Content built with Swift Playgrounds SDK modules: `SPCCore`, `SPCAccessibility`, `SPCAudio`, `SPCAR`
- Playground books are open-source — can be unpacked on Mac to inspect Apple's implementation
- Templates: Quest Create, Sonic Create, Sensor Create, AR Create — allow developers to build custom playground books using the same SDK

## APIs & Frameworks

**Swift Playgrounds Content SDK**
- `SPCCore` module — base graphics and playground infrastructure
- `SPCAccessibility` module — accessibility support for playground content
- `BaseGraphic` — base class for all renderable on-screen elements
- `BaseGraphic.accessibilityHints: AccessibilityHints?` — accessibility configuration property
- `AccessibilityHints` struct:
  - `init(makeAccessibilityElement: Bool, accessibilityLabel: String?)`
  - `makeAccessibilityElement: Bool`
  - `accessibilityLabel: String?`
- `Graphic`, `Button`, `Sprite`, `Label` — concrete subclasses of `BaseGraphic`

**UIKit Accessibility** (underlying platform APIs)
- `UIAccessibilityElement` — base class that `AccessibilityHints` hooks into
- `accessibilityLabel` property — read by VoiceOver

## Code Highlights

Making a graphic accessible with a descriptive label:
```swift
import SPCCore
import SPCAccessibility

let hints = AccessibilityHints(
    makeAccessibilityElement: true,
    accessibilityLabel: "Activate button to start the party"
)
let graphic = Graphic(name: "Let's get it Started")
graphic.accessibilityHints = hints
```

Applying descriptive accessibility labels to cave torches (the chapter's challenge):
```swift
cave.torch1.accessibilityHints = AccessibilityHints(
    makeAccessibilityElement: true,
    accessibilityLabel: "Torch next to a stairwell, where dripping water can be heard."
)
cave.torch2.accessibilityHints = AccessibilityHints(
    makeAccessibilityElement: true,
    accessibilityLabel: "Right before the edge of the platform — be careful!"
)
```

## Takeaways

- Unlabeled buttons and images produce "button, button, button" in VoiceOver — a useless experience that causes VoiceOver users to delete an app within the first minute; descriptive `accessibilityLabel` strings are the single highest-impact accessibility improvement you can make.
- The Swift Playgrounds content SDK (`SPCCore`, `SPCAccessibility`) exposes `AccessibilityHints` as a simple two-property struct, making it trivial to retrofit VoiceOver support onto any `BaseGraphic` element in a playground.
- VoiceOver supports all Apple platforms and can be enabled without any app changes — test your app with VoiceOver on regularly to catch gaps before users do.
- Swan's Quest chapters 2–4 build on chapter 1, adding audio tone generation, musical notation with variable note lengths, and MIDI-based step sequencing.

---
_Source: WWDC20 Session 10681 page (transcript and code samples)._
