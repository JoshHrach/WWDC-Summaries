# What's New in SF Symbols 4
**WWDC22 · Session 10157** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10157/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, tvOS 16, watchOS 9

## Overview
SF Symbols 4 adds more than 700 new symbols, growing the library to over 4,000 unique symbols. Beyond new symbols, the release introduces three significant capabilities: Automatic Rendering mode (each symbol picks its own preferred rendering mode automatically), Variable Color (opacity-based sequential color layers that communicate strength or progress), and Unified Annotations (a single layer structure that drives all four rendering modes plus Variable Color in the SF Symbols app).

Five new categories in the SF Symbols app — Camera & Photos, Accessibility, Privacy & Security, Home, and Fitness — help navigate the expanded library. Particularly notable are new Home symbols (lights, blinds, windows, doors, switches, outlets, furniture, appliances) and new Fitness figure symbols.

## Key Topics

### New Symbols and Categories
Over 700 new symbols in iOS 16 covering: Home automation (light switches, power outlets, blinds, doors, windows, furniture, appliances), health, fitness figures, expanded currency symbols, and expanded localized symbols for additional scripts and right-to-left writing systems. Five new app categories: Camera & Photos, Accessibility, Privacy & Security, Home, Fitness.

### Automatic Rendering Mode **[NEW]**
Previously, if no rendering mode was specified, symbols defaulted to Monochrome. In SF Symbols 4, each symbol now has a preferred rendering mode. Setting the rendering mode to **Automatic** applies each symbol's preferred mode automatically, without explicit specification. Examples:
- `camera.filters` → Hierarchical (emphasizes translucency of camera lens layers)
- SharePlay symbol → Hierarchical (person shape foreground, waves background)
- AirPods Pro → Hierarchical by default but may need Monochrome override at small sizes

Rendering modes can still be explicitly set to override Automatic when context requires (e.g., Monochrome for small-size legibility or uniform appearance across symbol sets).

### Variable Color **[NEW]**
A new feature for dynamic symbols that communicate varying levels of strength, progress, or sequential states. Key concepts:
- Symbol vector paths are organized into **sequential layers**; each layer can opt in or out of the variable sequence
- Color is distributed through layers as a **percentage value** (0% = fully off; 100% = fully highlighted)
- Not all paths participate — e.g., the speaker shape in `speaker.wave.3` opts out; only the three wave paths participate in the volume sequence
- Not for creating depth — specifically for communicating sequences and changing states over time
- Available in **all four rendering modes** (Monochrome, Hierarchical, Palette, Multicolor); opacity-based
- Examples: Wi-Fi signal strength, speaker volume, room occupancy levels, timer progress

### Unified Annotations **[NEW]**
A new approach to annotating custom symbols in the SF Symbols app. Instead of annotating each rendering mode independently, a single unified layer structure drives all rendering modes:
- **Draw** — a layer draws (renders) its paths normally
- **Erase** — a layer erases overlapping areas of shapes beneath it (useful for badges, cutouts)
- Z-order of layers determines rendering priority
- One annotation workflow generates correct output for Monochrome, Hierarchical, Palette, and Multicolor simultaneously
- Variable Color annotation: split participating paths into individual layers in sequence order

## APIs & Frameworks

**SF Symbols / UIKit / AppKit / SwiftUI**
- `SymbolRenderingMode.automatic` **[NEW]** — selects each symbol's preferred rendering mode
- `SymbolRenderingMode.monochrome` — uniform single-color rendering
- `SymbolRenderingMode.hierarchical` — single hue with opacity-based depth
- `SymbolRenderingMode.palette` — two or more contrasting colors
- `SymbolRenderingMode.multicolor` — intrinsic/native color representation
- Variable Color support via `symbolVariableValue:` parameter **[NEW]**
  - `Image(systemName:variableValue:)` in SwiftUI **[NEW]**
  - `UIImage(systemName:variableValue:withConfiguration:)` in UIKit **[NEW]**
  - `NSImage(systemSymbolName:variableValue:accessibilityDescription:)` in AppKit **[NEW]**
- SF Symbols 4 app — new beta with unified annotation editor, Variable Color preview, new categories

## Code Highlights

```swift
// Automatic rendering mode (SwiftUI) — uses each symbol's preferred mode
Image(systemName: "camera.filters")
    .symbolRenderingMode(.automatic)

// Variable Color — 0.0 to 1.0 value drives sequential layer highlight
Image(systemName: "speaker.wave.3", variableValue: 0.75)

// Explicit palette rendering override for consistent multi-symbol layouts
Label("SharePlay", systemImage: "shareplay")
    .symbolRenderingMode(.monochrome)  // override for small-size legibility
```

```swift
// UIKit: variable color symbol
let config = UIImage.SymbolConfiguration(paletteColors: [.systemBlue, .systemGray])
let image = UIImage(systemName: "wifi",
                    variableValue: signalStrength,  // 0.0–1.0
                    withConfiguration: config)
```

## Takeaways

- Set rendering mode to `.automatic` by default and override only when context demands it (small sizes, uniform icon grids) — each symbol's preferred mode is designed to best communicate its meaning.
- Use Variable Color for any symbol representing a measurable quantity, level, or sequence (signal strength, volume, battery, progress, occupancy); pass a `Double` from 0.0 to 1.0 and the framework handles layer highlighting.
- When creating custom symbols, use the unified annotation approach in the SF Symbols 4 app — one layer structure covers all four rendering modes and Variable Color; use Draw/Erase designations to handle overlapping shapes like badges.
- Over 700 new symbols in SF Symbols 4 cover Home automation, fitness figures, health, and expanded localized scripts — audit your use of third-party icon sets before adding new icons.

---
_Source: WWDC22 Session 10157 page (abstract, chapter summaries, and resource links)._
