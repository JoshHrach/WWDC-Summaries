# Create Animated Symbols
**WWDC23 · Session 10257** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10257/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, watchOS 10, tvOS 17, visionOS 1

## Overview
SF Symbols 5 introduces symbol animations—a new dimension of expressiveness that makes symbols come alive in apps. This session is the SF Symbols app companion to "Animate symbols in your app" (10258), focusing on how to use the SF Symbols app to preview animations, update custom symbols for animation support, leverage the new Symbol Components feature, and ensure compatibility across older platforms.

The session covers four major areas: (1) the new Animation Inspector in the SF Symbols app for experimenting with animation presets and configurations; (2) how to annotate custom symbols for motion using layer groups and pulse marks; (3) Symbol Components, a powerful new workflow for combining custom symbols with system-style design elements (enclosures, badges, slashes) to produce consistent, animation-ready symbols; and (4) redesigned export and compatibility tooling to safely target older OS versions.

## Key Topics

### Animation Inspector (SF Symbols App — New)
A new "Animation" tab in the SF Symbols app lets developers preview all available animation effects on any SF Symbol or custom symbol. Features:
- **Animation presets**: Bounce, Pulse, Variable Color, Appear, Disappear, Replace, and Scale.
- **Configuration controls**: Options like "By Layer" vs. "Whole Symbol," upward vs. downward bounce, Cumulative vs. Iterative variable color, Reversing toggle.
- **"Automatic" vs. "Modified" indicator**: Shows whether you've changed from the default configuration.
- **Code snippet copy button**: Copies the Swift/Obj-C effect name for use in code.
- Preview shown simultaneously in Gallery view, symbol row, and sidebar for all rendering modes.

### Animating Custom Symbols
Custom symbols from SF Symbols 4 or earlier (or exported for Xcode 14) move as a whole unit for motion animations. To enable per-layer motion in SF Symbols 5 / Xcode 15:

- **Pulse marks**: New annotation checkbox per layer; layers without marks will not pulse in "By Layer" mode—the whole symbol pulses instead.
- **Layer Groups** (new): Group multiple layers so they animate together rather than independently. Essential for symbols with complex geometry where parts shouldn't move separately. Created by selecting layers and adding a new group in the annotation pane.
- Grouped layers retain individual annotations (color, hierarchy, variable color) but move as one unit.
- **Variable Color**: Symbols with variable color annotation support Variable Color animation automatically.

### Symbol Components (New)
Symbol Components let developers combine a custom base symbol with reusable system design elements to produce a new symbol that looks and behaves like a system-provided SF Symbol:
- **Available components**: Enclosures (`.circle`, `.circle.fill`, etc.), badges (`.badge.plus`, etc.), slash, and more—40+ total.
- The resulting symbol inherits the component's artwork, rendering mode behavior (e.g., monochrome erase-through for `.circle.fill`), multicolor badge colors, animation behavior (slash animates separately), and variable color settings from the base symbol.
- Minor adjustments supported: badge position, slash length, enclosed symbol scale, weight compensation for enclosures.
- Powered by **Variable Templates**: base symbol must use variable templates (3 drawings: Small+Ultralight, Small+Regular, Small+Black). The system generates all 27 scale/weight combinations automatically with weight compensation as needed.
- Override Ultralight/Black weight variants independently when needed.
- **Limitation**: Symbol components cannot be exported as editable templates; they are designed to be finalized artwork.

### Compatibility and Export Redesign
- **Template export (wireframe view)**: Exported templates now show wireframe instead of filled shapes, helping designers focus on structure before annotation.
- **Compatibility picker** (new): Choose target SF Symbols version (SF Symbols 2/3/4/5 corresponding to iOS 14–17) when exporting templates. Older targets get simpler, appropriate drawings.
- **Annotation compatibility**: Setting a target platform in the annotation pane disables features not supported on that version (e.g., monochrome annotation customization grayed out for SF Symbols 3 and earlier).
- **Xcode export**: Specify target Xcode version; the app chooses the optimal file format and ensures compatibility when compiled for older platforms.

## APIs & Frameworks

This session focuses on the **SF Symbols app** (design tool), not code APIs. The runtime APIs for animations are covered in Session 10258 "Animate symbols in your app." Key design concepts:

**SF Symbols App (v5) — New Features**
- Animation Inspector tab **[NEW]** — preview animation presets
- Animation presets **[NEW]**: Bounce, Pulse, Variable Color, Appear, Disappear, Replace, Scale
- Pulse layer annotation marks **[NEW]** — per-layer pulse control
- Layer Groups **[NEW]** — group layers for unified motion
- Symbol Components **[NEW]** — combine custom symbol with 40+ system design elements
- Variable Templates — 3-drawing (Ultralight/Regular/Black) foundation for components
- Weight compensation — automatic for enclosed symbols
- Compatibility picker **[NEW]** — target-platform-aware export
- Wireframe template export **[NEW]** — cleaner drawing workflow

**SF Symbols Version / Platform Mapping**
- SF Symbols 2 → iOS 14 / macOS Big Sur (static templates only)
- SF Symbols 3 → iOS 15 / macOS Monterey (variable templates; no customizable monochrome annotation)
- SF Symbols 4 → iOS 16 / macOS Ventura (customizable monochrome; no motion layer groups)
- SF Symbols 5 → iOS 17 / macOS Sonoma (full animation and Symbol Components)

**Runtime APIs (brief mentions; see Session 10258)**
- SwiftUI: `Image(systemName:).symbolEffect(_:)` — apply animation effect
- UIKit/AppKit: `UIImageView.addSymbolEffect(_:)`, `NSImageView.addSymbolEffect(_:)`

## Code Highlights
This is a design/tooling session; no code samples are included. For runtime animation code, see Session 10258 "Animate symbols in your app."

## Takeaways
- The SF Symbols 5 app's Animation Inspector makes it easy to preview all animation presets and copy the exact Swift/Obj-C configuration name for use in code.
- Custom symbols must use Layer Groups to control which parts move together in "By Layer" motion animations—without groups, all layers move independently, often producing unexpected results.
- Symbol Components are the recommended way to add enclosures, badges, and slashes to custom symbols: they inherit all the system's rendering, animation, and variable color behavior automatically.
- The redesigned export flow with compatibility pickers prevents shipping symbols that break on older OS versions by generating platform-appropriate templates and file formats.

---
_Source: WWDC23 Session 10257 page (abstract, chapter summaries, and resource links)._
