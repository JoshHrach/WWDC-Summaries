# Adopt Variable Color in SF Symbols
**WWDC22 · Session 10158** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10158/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, tvOS 16, watchOS 9

## Overview
Variable Color is a new SF Symbols 4 feature that lets a symbol's appearance respond to a percentage value (0–100%), enabling expressive representations of quantities like signal strength, battery level, progress, and timers. Existing system symbols that support Variable Color work in all four rendering modes: Monochrome, Hierarchical, Palette, and Multicolor.

The session explains how Variable Color thresholds are computed (evenly spaced layers, rounded to the nearest percentage point, activated one point above), then walks through the new "unified annotation" workflow in the SF Symbols app that replaces the separate layer structures previously required for different rendering modes. Unified annotation uses one layer structure shared across all four rendering modes, allowing Variable Color, Monochrome control, and color to be configured simultaneously.

A new template format (4.0) is introduced along with new layer options: Erase (a layer cuts a hole in layers behind it) and Hidden (excludes a layer from specific rendering modes). Custom symbols annotated in previous versions are automatically upgraded.

## Key Topics

### Variable Color Behavior
- Controlled by a `Double` percentage value (0.0–1.0 in API, conceptually 0–100%)
- Exactly 0% = all variable layers inactive (visually empty)
- Any value > 0% activates the first layer; layers activate at evenly spaced thresholds between 0 and 100%
- Thresholds are rounded to the nearest integer percentage; next layer activates one percentage point above the rounded threshold
- Works identically across all four rendering modes: Monochrome, Hierarchical, Palette, Multicolor

### Unified Annotation (SF Symbols App)
- Single layer structure shared across all rendering modes — one annotation workflow instead of separate Hierarchical/Palette and Multicolor layer structures
- Now includes control over Monochrome rendering in addition to the other three modes
- New layer options: **Erase** (subtracts from layers behind; useful for badges) and **Hidden** (excludes layer from specific rendering modes)
- Variable Color enabled per-layer via a checkbox in the rendering inspector; layers lower in the z-order fill first
- Preview area shows all four rendering modes and a Variable Color slider simultaneously
- "Split into New Layers" context menu action helps quickly separate symbol paths into Variable Color layers

### Template Format 4.0
- Required for Variable Color and Monochrome control in Xcode
- Existing custom symbols annotated with previous formats are auto-upgraded
- Template formats 3.0 and 2.0 remain available for backward compatibility

### API Usage (SwiftUI / UIKit / AppKit)
- Pass a percentage value to symbols using `.variableValue` parameter
- Works with `Image(systemName:variableValue:)` in SwiftUI and `UIImage(systemName:variableValue:)` in UIKit

## APIs & Frameworks

**SF Symbols 4** (SF Symbols framework / UIKit / AppKit / SwiftUI)
- `Image(systemName:variableValue:)` **[NEW]** — SwiftUI, accepts `Double?` (0.0–1.0)
- `UIImage(systemName:variableValue:)` **[NEW]** — UIKit
- `UIImage(systemName:variableValue:withConfiguration:)` **[NEW]** — UIKit with symbol configuration
- `NSImage(systemSymbolName:variableValue:accessibilityDescription:)` **[NEW]** — AppKit

**SF Symbols App Features (Design Tool)**
- Unified annotation layer structure **[NEW]** — single layer set shared across all rendering modes
- Variable Color layer option **[NEW]** — enables variable color per layer
- Erase layer option **[NEW]** — layer subtracts/erases from layers below
- Hidden layer option **[NEW]** — excludes layer from specific rendering mode
- Template format 4.0 **[NEW]** — required for Variable Color and Monochrome control
- Rendering mode preview area **[NEW]** — shows all four rendering modes simultaneously
- Variable Color slider **[NEW]** — previews symbol at any percentage in real time
- Automatic rendering mode preference per symbol (new "Automatic" option in rendering mode picker)

**Rendering Modes (all now Variable Color compatible)**
- `.monochrome` — single color, Variable Color dims layers
- `.hierarchical` — primary/secondary/tertiary hierarchy, Variable Color dims layers
- `.palette` — multi-layer custom colors, Variable Color dims layers
- `.multicolor` — system-defined colors, Variable Color dims layers

## Code Highlights

Using Variable Color in SwiftUI:
```swift
// variableValue ranges from 0.0 to 1.0
Image(systemName: "speaker.wave.3.fill", variableValue: signalStrength)
    .symbolRenderingMode(.multicolor)
```

Using Variable Color in UIKit:
```swift
let image = UIImage(systemName: "wifi", variableValue: signalStrength)
imageView.image = image
```

Animating Variable Color for a progress timer:
```swift
// Update periodically
let elapsed = Date().timeIntervalSince(startTime)
let percentage = min(elapsed / totalDuration, 1.0)
timerImageView.image = UIImage(systemName: "cube.fill", variableValue: percentage)
```

## Takeaways
- Variable Color works across all four rendering modes with no per-mode adoption — annotate once in the SF Symbols app and all modes respond to the percentage value automatically.
- Threshold computation is percentage-point-safe: layers won't activate unexpectedly due to floating-point rounding when you pass round percentage values.
- Unified annotation in SF Symbols 4 replaces separate Hierarchical and Multicolor layer structures — existing symbols upgrade automatically, but export as template 4.0 to access Variable Color in Xcode.
- The Erase layer option solves the classic "badge with cutout" problem in Monochrome mode, previously requiring separate symbol variants.

---
_Source: WWDC22 Session 10158 page (abstract, chapter summaries, code samples, and resource links)._
