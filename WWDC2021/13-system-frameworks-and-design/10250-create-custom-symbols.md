# Create Custom Symbols
**WWDC21 · Session 10250** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10250/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, tvOS 15, watchOS 8

## Overview
SF Symbols 3 introduces a new version 3.0 custom symbol template with support for hierarchical and palette rendering modes, custom margins for any design variant, and a new interpolation system that can generate all 27 weight/scale combinations from just three source drawings. This design-focused session walks through the complete custom symbol workflow: creating a 3.0 template, drawing for multicolor and hierarchical rendering, annotating layers in the SF Symbols app, using the variable template setup for interpolation, and distributing the final `.svg` template to Xcode or colleagues.

## Key Topics

### Custom Symbol Template v3.0
The new 3.0 template (requires Xcode 13+) introduces several advances over 2.0:
- **Named margins**: Left and right margin guidelines now carry a variant suffix (e.g., `Regular-M`), allowing per-variant optical alignment control.
- **Rendering mode data**: The template embeds annotation data for monochrome, multicolor, hierarchical, and palette modes.
- **Interpolation support**: When three specific design sources are present with compatible paths, the remaining 24 variants are generated automatically.

Existing 1.0 and 2.0 templates can be imported into the SF Symbols app, which automatically up-converts them to 3.0.

### Template Setup: Static vs. Variable
When exporting a template from the SF Symbols app, choose:
- **Static** (27 variant slots, one per weight/scale combination) — best when targeting a specific weight/scale or drawing only 1–2 variants. Regular-Medium is always required.
- **Variable** (3 source slots: Ultralight-Small, Regular-Small, Black-Small plus 3 margin sets) — best when planning to support all 27 variants via interpolation. Requires strict path compatibility between sources.

### Drawing for Rendering Modes
Three rules for symbols intended for multicolor or hierarchical rendering:
1. **Convert all strokes to paths** before finalizing. Filled closed shapes can take on colors; live strokes cannot.
2. **Use only closed paths.** Open paths (paths with disconnected start/end points) have no fill area and cannot receive color assignments.
3. **Avoid non-flat fills.** Gradients, drop shadows, and multi-color fills override rendering mode annotation data and must not be present.

**Consistent path count and order** across design variants is required for both multicolor annotation and interpolation. Reordering paths across variants causes rendering mode annotations to misalign.

### Annotation in the SF Symbols App
In the SF Symbols app gallery view, the inspector's rendering mode tab allows annotation for multicolor, hierarchical, and palette modes.

**Multicolor annotation**: Create layers by grouping paths; assign a system color or custom color to each layer. Use system colors when possible — they adapt to light/dark mode and vibrancy. Layers have explicit z-order. Low-opacity layers can simulate a stroked+filled effect; set "clear behind" on transparent layers to prevent blending with layers below.

**Hierarchical annotation**: Create layers and assign a hierarchy group: `primary`, `secondary`, or `tertiary`. This data is shared between hierarchical and palette modes. The "clear behind" toggle per layer controls how transparent shapes interact with layers below.

**Palette mode interpretation**: With 2 colors, colors distribute across available hierarchy groups regardless of level. With 3 colors, each maps to primary/secondary/tertiary respectively.

### Interpolation Requirements
For the SF Symbols app to generate all 27 variants automatically:
1. Three design sources present: Ultralight-Small, Regular-Small, Black-Small.
2. All paths have the same count and order across the three sources.
3. Each corresponding path across sources has the same number of points (point compatibility).

When drawing interpolatable symbols: start with Regular-Small, bring it to a finished state, then copy-paste those paths to Ultralight-Small and Black-Small and only reposition points — never add or remove points.

### Distribution
- **3.0 template** (`.svg` with embedded annotation): Use when deploying to iOS 15+ or macOS 12+. Supports all rendering modes. Not suitable for direct editing in design tools after annotation — import back to SF Symbols app first.
- **2.0 template** (`.svg`, monochrome only): Use when back-deploying to iOS 14. Lossy conversion — strips annotation data and explicit margins.
- For dual-target apps (iOS 14+): export both templates; do a version check at runtime to load the appropriate asset.
- To share with a colleague for further editing: export 3.0 template → colleague imports into their SF Symbols app.

## APIs & Frameworks

**SF Symbols app** (standalone macOS utility)
- Version 3.0 template: embeds multicolor, hierarchical, and palette annotation data
- Gallery view → Rendering Mode inspector: create/edit layers for each mode
- Export: File → Export Template (Command+E); choose Static or Variable setup; choose version 2.0 or 3.0
- Import: drag `.svg` onto custom symbol cell to update; drag 1.0/2.0 templates to auto-convert to 3.0
- "Duplicate as Custom Symbol" from any SF Symbol to create a starting point

**Xcode** (version 13 required for 3.0 templates)
- Add custom `.svg` symbol to asset catalog; Xcode reads embedded annotation data

**UIKit / AppKit / SwiftUI** (rendering, covered in companion sessions)
- Hierarchical rendering: `UIImage.SymbolConfiguration(hierarchicalColor:)`, `Image(systemName:).symbolVariant(.fill).foregroundStyle(.secondary)`
- Palette rendering: `UIImage.SymbolConfiguration(paletteColors:)`, `Image(systemName:).foregroundStyle(.red, .blue)`
- Multicolor: automatic when using system images with multicolor annotations

## Code Highlights

Loading a custom symbol in SwiftUI (iOS 15):
```swift
// Custom symbol added to asset catalog
Image("queen.heart")
    .symbolRenderingMode(.hierarchical)
    .foregroundStyle(.red)

// With palette rendering (2 colors maps to available hierarchy groups)
Image("queen.heart")
    .symbolRenderingMode(.palette)
    .foregroundStyle(.yellow, .red)
```

Loading a custom symbol in UIKit (iOS 15):
```swift
let hierarchicalConfig = UIImage.SymbolConfiguration(hierarchicalColor: .systemRed)
let image = UIImage(named: "queen.heart")?.applyingSymbolConfiguration(hierarchicalConfig)

let paletteConfig = UIImage.SymbolConfiguration(paletteColors: [.systemYellow, .systemRed])
let paletteImage = UIImage(named: "queen.heart")?.applyingSymbolConfiguration(paletteConfig)
```

Version-aware asset loading (supporting iOS 14 and iOS 15):
```swift
let symbolName = "queen.heart"
if #available(iOS 15, *) {
    // 3.0 template in asset catalog: use hierarchical/palette configs
    let config = UIImage.SymbolConfiguration(hierarchicalColor: .systemRed)
    imageView.image = UIImage(named: symbolName)?.applyingSymbolConfiguration(config)
} else {
    // 2.0 template fallback: monochrome only
    imageView.image = UIImage(named: symbolName)
}
```

## Takeaways
- SF Symbols 3.0 templates embed hierarchical and palette annotation data directly in the SVG; Xcode 13 and iOS 15 / macOS 12 are required to use this data.
- To support all rendering modes, every path must be a closed, flat-filled vector shape — convert strokes to outlines and remove gradients before annotating.
- Path count and order must be identical across all design variants (for both annotation consistency and interpolation eligibility).
- The variable template setup enables automatic generation of all 27 weight/scale combinations from just three compatible source drawings (Ultralight-Small, Regular-Small, Black-Small).
- Distribute finished symbols as 3.0 templates to colleagues (for further editing in SF Symbols app) and to Xcode (for production). Back-deploy with 2.0 templates for iOS 14 targets.

---
_Source: WWDC21 Session 10250 page (abstract, chapter summaries, code samples, and resource links)._
