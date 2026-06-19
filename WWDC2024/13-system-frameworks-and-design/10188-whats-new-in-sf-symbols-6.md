# What's New in SF Symbols 6
**WWDC24 · Session 10188** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10188/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia 15, visionOS 2, watchOS 11, tvOS 18

## Overview
SF Symbols 6 adds three new universal animation presets — Wiggle, Rotate, and Breathe — alongside improvements to Replace (now with Magic Replace as the default behavior) and Variable Color (closed-loop repeat support). The library also expands to over 6,000 symbols with 800+ new additions. The session walks through each animation preset, shows how to annotate custom symbols in the SF Symbols app to support the new behaviors, and demonstrates all presets in a sample plant-care app.

The SF Symbols app receives updates including finer animation repeat controls (play once, repeat with delay, continuous), new annotation workflows for Wiggle direction and Rotate anchor points, and a Snap to Points tool for centering rotation anchors precisely.

## Key Topics

### Wiggle (New Preset)
Wiggle produces a short oscillating motion that draws attention to a control or reinforces what a symbol represents. Direction options include: right, left, forward/backward (reading-direction-aware), up, down, clockwise, counterclockwise, and a custom angle (e.g., 315° for a paper plane symbol). Many system symbols have a built-in preferred direction; custom symbols can declare one via annotation.

### Rotate (New Preset)
Rotate spins a symbol to convey ongoing activity or imitate real-world object behavior. Entire-symbol rotation is supported, as is by-layer rotation — a specific layer (e.g., fan blades, circular arrows) can be set as `canRotate` and will spin around a configurable anchor point while the rest of the symbol stays still. The anchor point is set in three weights in the SF Symbols app; the system interpolates for all other weights and scales.

### Breathe (New Preset)
Breathe smoothly increases and decreases a symbol's presence — combining opacity and size changes — to convey a living, ongoing-activity quality (e.g., a water-drop animation during a watering timer). It differs from Pulse (opacity only, introduced in SF Symbols 5): Breathe adds scale change for more presence. The `pulses` option in Breathe also activates any layers annotated for Pulse, adding depth.

### Magic Replace (Replace Enhancement)
Magic Replace is the new default behavior for the Replace animation. When transitioning between two related symbols, slashes can draw on/off and badges can appear, disappear, or swap independently of the base symbol. If the two symbols are not related enough for Magic Replace, the system falls back to a standard Replace with the configured direction. Symbols built with symbol components are compatible; re-export from SF Symbols 6 app and import into Xcode 16 to enable.

### Variable Color Improvements
Variable Color's repeat behavior is improved to honor the design of each symbol. Symbols with a closed-loop design (where start and end points meet) now repeat seamlessly in a smooth continuous loop. Custom symbols can be annotated to opt into this closed-loop repeat optimization.

### SF Symbols App Updates
- Animation inspector shows all new presets alongside existing ones
- Wiggle: Default or custom direction per-layer annotation
- Rotate: `canRotate` layer option with visual anchor point editor; Snap to Points aligns anchor to path vectors; three-weight anchor definition
- Breathe: Pulses toggle for layered depth effect
- Animation repeat modes: play once, repeat with delay (configurable pause), continuous
- 800+ new symbols: Automotive (batteries, convertibles, temperature indicators), diverse activity figures, localized symbols (Greek, Cyrillic, Indic numeral systems), progress indicators, haptics, home and widget symbols, and many objects; total library exceeds 6,000 symbols

### Custom Symbol Annotation
Custom symbols need layer annotations in the SF Symbols app to take full advantage of new presets. Key steps: assign layers to Wiggle (set preferred direction), set `canRotate` on layers and position the rotation anchor point, assign Breathe layers in desired animation order. Symbols previously annotated for Pulse work automatically with Breathe's Pulses option.

## APIs & Frameworks

**SwiftUI**
- `Image(systemName:)` — SF Symbols entry point (existing)
- `.symbolEffect(_:)` modifier — apply animations (existing)
- `WiggleSymbolEffect` **[NEW]** — `SymbolEffect.wiggle`
  - Direction variants: `.right`, `.left`, `.up`, `.down`, `.clockwise`, `.counterClockwise`, `.byLayer`, custom angle
- `RotateSymbolEffect` **[NEW]** — `SymbolEffect.rotate`
  - `.byLayer` — rotate only annotated layer
- `BreatheSymbolEffect` **[NEW]** — `SymbolEffect.breathe`
  - `.pulses` option — activates Pulse-annotated layers alongside Breathe
- Magic Replace **[NEW]** — new default for `.replace` transition between related symbols
  - `.replace.magic(fallback:)` — explicit Magic Replace with fallback direction
- `VariableColorSymbolEffect` — closed-loop repeat support **[NEW behavior]**
- `.contentTransition(.symbolEffect(.replace))` — used for symbol-swap transitions (existing, Magic Replace default added)

**UIKit**
- `UIImageView.addSymbolEffect(_:)` — Wiggle, Rotate, Breathe in UIKit (existing infrastructure, new presets)
- `UIImage.SymbolConfiguration` — weight, scale, rendering mode (existing)

**AppKit**
- `NSImageView` symbol effect support — same new presets available

**SF Symbols App (Tooling)**
- Wiggle annotation: preferred direction per layer **[NEW]**
- Rotate annotation: `canRotate` layer flag, visual anchor point editor, Snap to Points tool, three-weight anchor definition **[NEW]**
- Breathe annotation: layer ordering for sequential animation **[NEW]**
- Variable Color closed-loop annotation **[NEW]**
- Animation repeat controls: once, delay, continuous **[NEW]**

## Code Highlights

Wiggle to call attention to a control:
```swift
Image(systemName: "chevron.right")
    .symbolEffect(.wiggle, options: .repeating)
```

Rotate (by layer) for a progress indicator in a Live Activity:
```swift
Image(systemName: "arrow.trianglehead.2.clockwise.rotate.90")
    .symbolEffect(.rotate.byLayer)
```

Breathe for an ongoing activity timer:
```swift
Image(systemName: "drop.fill")
    .symbolEffect(.breathe, isActive: isWatering)
```

Magic Replace when switching between weather warning symbols:
```swift
Image(systemName: currentWeatherSymbol)
    .contentTransition(.symbolEffect(.replace.magic(fallback: .replace.offUp)))
```

## Takeaways
- Use Wiggle to highlight calls to action or UI elements that may be overlooked; prefer the symbol's built-in preferred direction for coherent motion.
- Rotate `.byLayer` keeps complex symbols anchored while animating only the meaningful part (fan blades, circular arrows) — set the rotation anchor point precisely in the SF Symbols app using Snap to Points.
- Adopt Magic Replace as the default transition wherever symbols change state; it produces the most polished results with minimal code and automatically falls back to standard Replace when symbols are unrelated.
- Before creating custom icons, browse the 800+ new symbols added in SF Symbols 6 — the library now exceeds 6,000 symbols across automotive, localization, home, haptics, and many other categories.

---
_Source: WWDC24 Session 10188 page (abstract, chapter summaries, and resource links)._
