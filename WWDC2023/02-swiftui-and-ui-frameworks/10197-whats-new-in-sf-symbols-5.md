# What's new in SF Symbols 5
**WWDC23 · Session 10197** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10197/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, tvOS 17, watchOS 10, visionOS 1

## Overview
SF Symbols 5 introduces Animation — a collection of expressive, configurable animation presets that bring movement and vitality to symbols across all Apple platforms. Building on previous releases that added Hierarchical/Palette/Multicolor rendering modes (2021) and Variable Color (2022), animation is the biggest leap forward yet, making symbols a dynamic communication tool rather than static icons.

The session covers the seven new animation presets, the concepts behind layer-based animation and spatial planes, and guidance for drawing custom symbols that animate correctly. It also highlights over 700 new symbols added to the library, pushing the total past 5,000.

## Key Topics

**Animation Concepts**
- Symbols animate **by layer** (default — each layer one at a time, precise choreography) or **by whole symbol** (all layers simultaneously)
- Three animation planes: middle (reference), front (larger/closer), back (smaller/farther); directionality determines how symbols move through planes

**Animation Presets**
- **Appear** — symbol gradually emerges into view when introduced to the interface
- **Disappear** — reverse of Appear; used when removing an element
- **Bounce** — fast elastic movement; signals successful interaction or reinforces the symbol's concept
- **Scale** — changes symbol size (up or down); stateful (scaled state persists); mimics physical button press
- **Pulse** — opacity change to convey ongoing activity; works per-layer for precise feedback (e.g., pulsing only the screen layer of a screen-share symbol)
- **Variable Color** — two sub-modes: **Cumulative** (layers highlight sequentially, keeping previous state, e.g., Wi-Fi connecting) and **Iterative** (one layer at a time, e.g., Wi-Fi searching); supports a **reversing** option
- **Replace** — swaps one symbol for another to communicate state changes; supports directionality modes **Down/Up** and **Up/Up**

**Drawing for Animation**
- Symbols should be drawn as **whole shapes** with erase layers to enable smooth interpolation
- **Draw** layers render paths; **Erase** layers remove paths, adding depth and smooth transitions
- Symbols without erase layers may look fine statically but lose depth and continuity in motion
- Custom symbols should be reviewed and paths adjusted before animating

**New Symbol Categories**
- Automotive: steering wheels, car seats, seated figures
- Gaming: arcade consoles, arcade sticks, button types
- EV plugs (various connector types)
- Weather: moonrise, moonset, rainbow
- 700+ new symbols; library now exceeds 5,000 unique symbols

## APIs & Frameworks

**SF Symbols (symbol animation — integrated into SwiftUI, UIKit, AppKit)**
- Symbol animation presets **[NEW]**:
  - `.appear` **[NEW]**
  - `.disappear` **[NEW]**
  - `.bounce` **[NEW]**
  - `.scale` (up/down, stateful) **[NEW]**
  - `.pulse` (whole symbol or by layer) **[NEW]**
  - `.variableColor` with `.cumulative` and `.iterative` sub-modes, `.reversing` option **[NEW]**
  - `.replace` with `.downUp` and `.upUp` directionality **[NEW]**
- Animation scope options **[NEW]**:
  - `.byLayer` — animate each layer independently (default)
  - `.wholeSymbol` — animate all layers simultaneously
- Rendering modes (existing, work with all animations):
  - `.monochrome`
  - `.hierarchical`
  - `.palette`
  - `.multicolor`
  - Variable Color (introduced WWDC22)
- SF Symbols app — updated with animation preview; beta at developer.apple.com/sf-symbols
- Custom symbol layer annotations: `Draw`, `Erase`
- `SymbolRenderingMode` (existing enum, all modes work with animation)

_Note: The specific SwiftUI/UIKit API surface (e.g., `.symbolEffect()`, `addSymbolEffect()`) is detailed in companion sessions "Animate symbols in your app" (10258) and "Create animated symbols" (10257)._

## Code Highlights

The session is design-focused; code-level API details are covered in the companion sessions. Key usage pattern (from related session context):

```swift
// SwiftUI — apply a symbol effect
Image(systemName: "wifi")
    .symbolEffect(.variableColor.iterative.reversing)

// UIKit — add a symbol effect
imageView.addSymbolEffect(.bounce)
```

## Takeaways
- Add animation to symbols to provide clear, delightful feedback — use Bounce for action confirmation, Scale for stateful selection focus, Pulse for ongoing activity, and Replace for state transitions.
- When building custom symbols, always use erase layers; symbols without them lose visual quality when animated.
- Choose by-layer animation (default) for precision; whole-symbol animation when all layers should move together.
- Watch "Animate symbols in your app" (10258) for SwiftUI/UIKit API specifics and "Create animated symbols" (10257) for the SF Symbols app workflow.

---
_Source: WWDC23 Session 10197 page (abstract, chapter summaries, code samples, and resource links)._
