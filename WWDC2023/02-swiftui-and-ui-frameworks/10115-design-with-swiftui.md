# Design with SwiftUI
**WWDC23 · Session 10115** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10115/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, watchOS 10

## Overview
Apple Maps designers Will and Philip share how they used SwiftUI as a primary design tool when building the redesigned watchOS 10 Maps app. Rather than a how-to coding tutorial, this session is a practitioner's case study that makes the case for SwiftUI as a design-iteration tool — one that lowers the floor for non-programmers while raising the ceiling for sophisticated prototypes with real sensors, data, and animations.

The session covers six specific ways SwiftUI improves design work: as a general design tool, for surface-level detail exploration, for interaction design, for realistic testing, for creating bespoke design tools, and for presenting work to stakeholders with live on-device demos. All examples come from real Maps design work including the walking radius, compass interaction, Digital Crown zoom, and split-screen search.

## Key Topics

### SwiftUI as a Design Tool
SwiftUI is declarative (you say what you want, not how to build it), making it approachable for designers without programming backgrounds. Common UI elements are trivially instantiated (`Button`, `Image`, `Text`). Modifiers handle styling (shadows, borders, font, padding, aspect ratio) the same way design tools handle properties. Xcode's canvas gives immediate visual feedback as code changes. Result: designers can iterate with a speed and precision unmatched by static tools.

### Getting the Details Right (Surfacing Hidden Decisions)
Static prototypes conceal dynamic states: what does an image look like while loading? How does a button look pressed? Building on device in SwiftUI forces these decisions to the surface immediately. Example: testing the Maps Digital Crown zoom speed on an actual Watch revealed it was too fast; the team tuned the sensitivity directly in the prototype. Adding walking radius, POIs, and UI controls revealed further interaction edge cases that only became visible when the parts worked together.

### Designing for Interaction
SwiftUI has first-class support for gestures, transitions, and animations. Animations are performant and fully interruptible. Key examples:
- **Compass interaction**: Built a SwiftUI prototype using the Watch's internal sensor data (heading) to test a real-time compass transition. Took only hours to prototype with real sensor accuracy.
- **Split-screen search scroll**: Custom Digital Crown scroll interaction — slow movements used a loose spring, fast movements used a tighter spring, each triggering haptic feedback at a threshold. Impossible to prototype accurately in static tools.
- **Ticker animation**: Designed as a separate prototype then integrated, revealing further complexity.

### Testing Ideas Realistically
SwiftUI enables on-device testing in real-world conditions. Maps designers tested outdoors in sunlight to evaluate cartography contrast and legibility. They used real route data (San Francisco hills vs. New York flatlands) to expose a Y-axis scale bug in the elevation chart for flat routes. Realistic data reveals design failures that ideal-case mockups hide.

### Bespoke Design Tools
SwiftUI makes it practical to build one-off parameterized design tools. Example: a mini app to test how the walking radius renders over different map surfaces, letting designers scrub through line width, opacity, and blend mode values. These targeted tools answer specific design questions far faster than any static tool.

### Presenting Work
On-device SwiftUI demos outperform slide decks for stakeholder reviews. Multiple prototypes were bundled into a single demo app and used in design reviews. Demos are self-explanatory, build consensus faster, and reduce explanation overhead. Crucially: SwiftUI prototypes can be shipped — the code is production-quality material.

## APIs & Frameworks

### SwiftUI
- `Text` — declarative text view
- `Button` — interactive control with built-in pressed states
- `Image` — image display with loading states
- `ViewModifier` — applied via modifier syntax (`.font()`, `.padding()`, `.shadow()`, `.border()`, `.aspectRatio()`, `.opacity()`, `.blendMode()`)
- `Animation` — fully interruptible animation system; spring animations with configurable stiffness/damping
- `Gesture` / `DragGesture` / `TapGesture` — gesture recognizers
- `spring()` animation — tunable spring physics for scroll and transition interactions
- `withAnimation(_:_:)` — wraps state changes in animation
- `ScrollView` / Digital Crown scroll integration on watchOS
- Haptic feedback API — triggered on gesture thresholds
- `MapKit` integration — live map data in SwiftUI prototypes
- `WeatherKit` integration — live weather data
- `RealityKit` integration — AR content in SwiftUI
- Hardware sensor APIs — accelerometer, heading/compass sensor (`CMHeading`) for watchOS prototypes
- Color picker — system `ColorPicker` view
- Push transitions — `NavigationStack` / `.navigationTransition`
- Xcode canvas / live preview — real-time visual feedback during editing

## Code Highlights
The session is conceptual/design-focused; no specific code snippets were presented. Key patterns discussed:

- Tuning Digital Crown scroll sensitivity with a sensitivity coefficient
- Threshold-based spring animation switching (loose spring for slow scroll, tight spring for fast scroll)
- Parameterized tool: sliders bound to line width, opacity, and blend mode values rendered over a live map

## Takeaways
- SwiftUI is a genuine design tool, not just a developer tool — designers without programming backgrounds can learn it quickly and produce real prototypes.
- Build on device from day one; it surfaces interaction and layout decisions that are completely invisible in static mockups.
- Use real data in prototypes — ideal placeholder data consistently hides bugs and edge cases that appear immediately with realistic content.
- Ship the prototype: SwiftUI prototypes are built from the same material as production code, so a high-quality prototype can become the shipping product.

---
_Source: WWDC23 Session 10115 page (abstract, chapter summaries, code samples, and resource links)._
