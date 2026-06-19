# What's New in SF Symbols
**WWDC21 · Session 10097** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10097/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, watchOS 8, tvOS 15

## Overview
SF Symbols 3 introduces nearly 600 new symbols (bringing the total to over 3,000), expands localization coverage to seven additional scripts, and adds two brand-new rendering modes — Hierarchical and Palette — alongside the existing Monochrome and Multicolor modes. The session covers symbol anatomy and layer annotations, explains how annotations power the new color rendering modes, and surveys updates to the SF Symbols app and the system color library.

The central design insight is that symbols are built from closed vector paths grouped into annotated layers (primary, secondary, tertiary). These layer annotations are what make Hierarchical, Palette, and Multicolor rendering possible, and they also underpin the symbol interpolation system that produces consistent shapes across nine weights and three scales.

## Key Topics

### Symbol Basics Recap
Every SF Symbol comes in nine weights and three scales (small, medium, large), all maintaining the same point size with optically adjusted stroke thicknesses. Symbols are vertically centered to San Francisco's cap-height. Some have negative margins for horizontal alignment consistency.

### Symbol Variants
Symbols come in outlined (default/universal), filled, slash (inactive/unavailable), and enclosing (circle, square, rectangle) variants. Choosing the right variant for context is critical: filled works well for tab bars and selection states; outlined suits toolbars and navigation bars; enclosed variants improve legibility at small sizes.

### New Symbol Additions
Nearly 600 new symbols include: expanded Apple products and devices, video game controllers, health-related symbols, communication symbols, inset/layout symbols, watchOS-specific symbols, and new object symbols.

### Localization Expansion
New script coverage for Arabic, Hebrew, Devanagari, Thai, Chinese, Japanese, and Korean. Localized variants adapt automatically based on device language, including right-to-left support. Some symbols require custom designs for specific scripts rather than simple mirroring.

### Symbol Anatomy and Layer Annotation
Symbols are drawn with closed vector paths (important for interpolation). Strokes must be converted to outlines. Paths are grouped into layers — primary, secondary, tertiary — that define the visual hierarchy of a symbol's components. This annotation data powers all color rendering modes.

### Rendering Modes

**Hierarchical (NEW)** — single color applied to all layers but at different opacities (primary = full, secondary = reduced, tertiary = further reduced). Creates depth and visual hierarchy with one hue. Reduces visual noise while retaining essential shapes.

**Palette (NEW)** — two or more independently controlled colors assigned to layer groups. Uses the same annotation data as Hierarchical. If only two colors are specified, the tertiary layer replicates the secondary color. Enables per-symbol or per-group color palettes for distinguishing functionality.

**Multicolor** — intrinsic/native colors representing physical appearance or semantic meaning (green = add, red = remove). Symbols may be fully colored, partially colored (with accent color for non-Multicolor layers), or fall back to Monochrome if no Multicolor data exists.

**Monochrome** — single color applied uniformly to all layers without using annotation data. Most typographically neutral and consistent. Opacity and color can be customized, but they affect the whole symbol equally.

### SF Symbols App and Color Library Updates
Updated app ships with all four rendering modes and a new color picker. Color library updated across light, dark, and increased-contrast appearances. Key changes:
- `Brown` is now available on all platforms (previously macOS-only).
- `Teal` is redefined with a greener hue; the old Teal values are now named `Cyan`. Developers using `teal` must switch to `cyan` to preserve the legacy appearance.
- `Indigo` and `Purple` are now available on all platforms with more consistent hue definitions.

## APIs & Frameworks

- SF Symbols 3 library — 3,000+ symbols; available via `UIImage(systemName:)`, `NSImage(systemSymbolName:accessibilityDescription:)`, `Image(systemName:)` in SwiftUI
- `UIImage.SymbolConfiguration` — configures weight, scale, rendering mode, and color
- `UIImage.SymbolConfiguration(paletteColors:)` **[NEW]** — Palette rendering with an array of `UIColor`
- `UIImage.SymbolConfiguration(hierarchicalColor:)` **[NEW]** — Hierarchical rendering with a single `UIColor`
- `UIImage.SymbolRenderingMode` **[NEW]** — `.monochrome`, `.hierarchical`, `.palette`, `.multicolor`
- `.symbolRenderingMode(_:)` SwiftUI modifier **[NEW]** — sets rendering mode on `Image`
- `.foregroundStyle(_:_:_:)` SwiftUI modifier **[NEW]** — sets up to three layer colors for Palette rendering
- `NSImage.SymbolConfiguration` (macOS) — analogous configuration API
- `UIColor.brown` — now available on all platforms (previously macOS only)
- `UIColor.cyan` **[NEW]** — replaces the legacy `UIColor.teal` color values
- `UIColor.teal` — redefined with a greener hue in iOS 15/macOS 12
- `UIColor.indigo`, `UIColor.purple` — now cross-platform with consistent hue definitions
- SF Symbols app 3.0 — updated design tool for browsing, customizing, and exporting symbols

## Code Highlights

SF Symbols does not have standalone code samples in this session (it is a design-focused overview). The rendering mode APIs are covered in depth in companion sessions. Typical usage:

```swift
// SwiftUI — Hierarchical rendering
Image(systemName: "cloud.sun.rain")
    .symbolRenderingMode(.hierarchical)
    .foregroundColor(.blue)

// SwiftUI — Palette rendering with two colors
Image(systemName: "person.crop.circle.badge.checkmark")
    .symbolRenderingMode(.palette)
    .foregroundStyle(.blue, .green)

// UIKit — Palette rendering
let config = UIImage.SymbolConfiguration(paletteColors: [.systemBlue, .systemGreen])
let image = UIImage(systemName: "person.crop.circle.badge.checkmark",
                    withConfiguration: config)
```

## Takeaways

- SF Symbols 3 adds Hierarchical and Palette rendering modes, enabling rich single- and multi-color iconography driven entirely by symbol layer annotations — no custom artwork needed.
- Teal has been renamed to Cyan; code using `UIColor.teal` should be updated to `UIColor.cyan` to preserve legacy appearance.
- Localization is now automatic for 7 new scripts; right-to-left symbols require explicit design review rather than simple mirroring.
- The rendering mode system (Monochrome → Hierarchical → Palette → Multicolor) covers the full spectrum from typographic uniformity to expressive native color — each symbol supports all four modes.

---
_Source: WWDC21 Session 10097 page (abstract, chapter summaries, code samples, and resource links)._
