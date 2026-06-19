# What's new in SF Symbols 7
**WWDC25 · Session 337** · [Watch](https://developer.apple.com/videos/play/wwdc2025/337/)

_Platforms:_ iOS 26, iPadOS 26, macOS Tahoe 26, tvOS 26, visionOS 26, watchOS 26

## Overview
SF Symbols 7 introduces three major visual capabilities — Draw animations, Gradients, and Magic Replace enhancements — along with a comprehensive new guide for annotating custom symbols to support all of these features. The session is design-forward: it explains how symbol paths are constructed (as outlined shapes rather than simple strokes) so that the new guide-point annotation system makes intuitive sense.

Draw is the headliner: two new animation presets (Draw On / Draw Off) that animate symbols as if a hand were drawing or erasing them along the symbol's path. Combined with a new Variable Draw rendering mode, the same annotation system can visualize continuous progress (like a download percentage). Magic Replace is extended to match enclosures between symbols and blend Draw animations into cross-symbol transitions. Gradients apply a smooth linear gradient derived from a single source color to any symbol in any rendering mode.

## Key Topics

### Draw animations
Two new symbol effect presets — **Draw On** and **Draw Off** — animate content along path guide points. Three playback modes: **By Layer** (staggered, default), **Whole Symbol** (all layers simultaneously), and the new **Individually** (layers animate one by one, waiting for the previous to finish). Draw direction is per-symbol (left-to-right, right-to-left, from center, clockwise, etc.). Composed shapes (e.g., arrowheads) can travel with their parent path via *attachments*.

**Variable Draw** — an extension of Variable Color — uses the same guide point annotations to show a partial draw at a percentage value (0.0–1.0), ideal for progress indicators. Symbols opt in per-layer; only one variable mode (color or draw) can be active at render time; use `.default` to let the system choose.

### Magic Replace enhancements
Magic Replace now recognizes **matching enclosures** (e.g., a circle around a symbol) across two related symbols and preserves the enclosure while animating only the inner content. Draw On/Off are also applied in Magic Replace transitions, mixing with enclosure matching for the most expressive possible cross-symbol animation.

### Gradients
A **linear gradient** is generated from a single source color (system or custom), adding a sense of lighting and depth. Gradients work across all rendering modes and scale well; they are most impactful at larger sizes. Toggle with `.colorRenderingMode(.gradient)` in SwiftUI or via `UIImage.SymbolConfiguration` / `NSImage.SymbolConfiguration`.

### Custom symbol annotation
Guide points are placed along paths in the SF Symbols app to tell the Draw system how to animate the symbol. Minimum: one **start point** (open circle) and one **end point** (closed circle). Complex paths need additional guide points for corners, bidirectional drawing, subpath disambiguation, and attached paths (arrowheads). Annotation is done on three template weights (Regular, Ultralight, Black); the system interpolates the rest. Guide point numbers help verify ordering consistency across weights. Adaptive end caps match the drawn path's cap style. Variable Draw is opted in per-layer via the layer list.

## APIs & Frameworks

### SwiftUI
- **`.symbolEffect(.drawOn)`** **[NEW]** — apply Draw On animation
- **`.symbolEffect(.drawOff)`** **[NEW]** — apply Draw Off animation
- `.symbolEffect(_:options:value:)` — existing modifier, now supports Draw presets
- `.symbolEffect(_:options:)` with `.playbackMode(.individually)` **[NEW]**
- **`.colorRenderingMode(.gradient)`** **[NEW]** — enable gradient rendering on a symbol
- `.symbolVariableValue(_:)` — existing; now also drives **Variable Draw** when symbol uses draw annotations
- `SymbolEffect.DrawOn` / `SymbolEffect.DrawOff` **[NEW]**
- `DiscreteSymbolEffect.PlaybackMode.individually` **[NEW]**
- `VariableValueSymbolEffect` — extended to support `.variableValueMode(.draw)` **[NEW]**

### UIKit
- `UIImage.SymbolConfiguration` — extended with gradient color rendering option **[NEW]**
- `UIImageView.addSymbolEffect(.drawOn)` / `.addSymbolEffect(.drawOff)` **[NEW]**

### AppKit
- `NSImage.SymbolConfiguration` — extended with gradient color rendering option **[NEW]**
- `NSImageView.addSymbolEffect(.drawOn)` / `.addSymbolEffect(.drawOff)` **[NEW]**

### SF Symbols App (design tool)
- **Draw toolbar** with guide point placement mode **[NEW]**
- **SliceRange** annotation types: start point, end point, corner point, bidirectional, attachment **[NEW]**
- **Variable Draw** opt-in per layer in Rendering Inspector **[NEW]**
- Enhanced Variable Rendering Picker with Draw option **[NEW]**

## Code Highlights

```swift
// SwiftUI: Draw On when isHidden becomes false
Image(systemName: "arrow.right")
    .symbolEffect(.drawOn, value: !isHidden)
    .symbolEffect(.drawOff, value: isHidden)

// Explicitly set playback mode
Image(systemName: "scribble")
    .symbolEffect(.drawOn.byLayer.individually)

// Variable Draw progress
Image(systemName: "thermometer.medium")
    .symbolVariableValue(progress)   // 0.0–1.0 drives Draw or Color

// Gradient rendering
Image(systemName: "star.fill")
    .colorRenderingMode(.gradient)
```

## Takeaways
- Download SF Symbols 7 beta and annotate custom symbols with guide points; start with simple single-path symbols to learn the system before tackling complex arrows or circles.
- Use Draw On/Off for onboarding, empty-state reveals, and tutorial moments where the symbol's meaning benefits from animation — use `Individually` playback for the most deliberate effect.
- Variable Draw is a strong alternative to progress rings for compact contexts; enable it per-layer so non-draw layers (e.g., thermometer body) remain static.
- Apply `.colorRenderingMode(.gradient)` to hero icons and large symbol displays — it reads naturally at smaller sizes and becomes visually striking at `largeTitle` scale and above.

---
_Source: WWDC25 Session 337 page (abstract, chapter summaries, and resource links)._
