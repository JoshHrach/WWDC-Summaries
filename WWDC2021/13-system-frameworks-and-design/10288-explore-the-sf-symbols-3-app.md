# Explore the SF Symbols 3 App
**WWDC21 · Session 10288** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10288/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, watchOS 8, tvOS 15

## Overview
This session tours the SF Symbols 3 app (the companion Mac application for exploring, previewing, and managing SF Symbols). It covers new features for symbol discovery, platform availability checking, localization awareness, the four rendering modes (including two new ones in SF Symbols 3), custom symbol creation and annotation, and new copy/export capabilities for design workflows.

The SF Symbols library now contains over 3,000 symbols and supports Monochrome, Multicolor, Hierarchical, and Palette rendering modes. The session is aimed at both designers and developers and is a companion to the "What's new in SF Symbols" and "Create custom symbols" sessions.

## Key Topics

**Finding and Organizing Symbols**
Symbols can be browsed by category or searched by keyword in the sidebar. Collections allow grouping symbols for a project. The information inspector shows availability by SF Symbols version (and the corresponding OS version), deprecated names for backward compatibility, and localization variants.

**Platform Availability and Localization**
The information inspector surfaces the minimum SF Symbols version for each symbol name, enabling developers to choose the right name for their deployment target. Deprecated names remain valid for older OS support (e.g., SF Symbols 1.0 = iOS 13). Certain symbols automatically adapt their glyph to the user's locale (e.g., book with the correct script and reading direction for RTL languages), requiring no extra developer work.

**Rendering Modes Preview**
The rendering inspector lets you preview all four rendering modes:
- **Monochrome** — single color; unchanged from SF Symbols 1/2
- **Multicolor** — intrinsic colors per symbol layer
- **Hierarchical** (new in SF Symbols 3) — single color, multi-opacity layers for depth
- **Palette** (new in SF Symbols 3) — two or three explicitly specified colors per layer

**Custom Symbols**
System symbols can be duplicated as a starting point for a custom symbol via _File > Duplicate as Custom Symbol_. Templates are exported as SVG for editing in a vector editor (e.g., Sketch), then dragged back in to update. The SF Symbols app includes a new Gallery view for large-scale editing. Custom symbols can be annotated with layer-based color information to opt into Multicolor, Hierarchical, and Palette rendering modes.

**Copying and Exporting**
- _Edit > Copy Symbol_ (Cmd-C): copies the symbol name as text using the SF font — best for Monochrome use alongside text in design tools.
- _Edit > Copy Image_ (Option-Cmd-C) **[NEW]**: copies a rendered PNG or SVG image of the symbol at a specified point size and pixel scale — ideal for Multicolor/Hierarchical/Palette design comps and custom symbols.
- _Export Symbol_: exports the full custom symbol SVG template for use in an Xcode asset catalog (preferred over Copy Image for asset catalogs).

## APIs & Frameworks

- **SF Symbols 3** app **[NEW]** — Mac design tool
- SF Symbols library — now 3,000+ symbols
- Rendering modes:
  - `.monochrome` — `SymbolRenderingMode.monochrome`
  - `.multicolor` — `SymbolRenderingMode.multicolor`
  - `.hierarchical` **[NEW]** — `SymbolRenderingMode.hierarchical`
  - `.palette` **[NEW]** — `SymbolRenderingMode.palette`
- Symbol availability versioning (SF Symbols 1.0 through 3.0)
- Symbol localization — automatic script and directionality adaptation
- Custom symbol template (SVG format) — layer-based annotation system
- Symbol layers: Primary, Secondary, Tertiary (for Hierarchical/Palette)
- `UIImage(systemName:)` / `NSImage(systemSymbolName:accessibilityDescription:)` — symbol usage in code (referenced, not demoed)
- SF Pro / SF Compact / SF Mono fonts — used for copy-as-text in design tools
- _Edit > Copy Image_ (Option-Cmd-C) **[NEW]** — PNG/SVG image copy
- _Edit > Copy Image As..._ **[NEW]** — format/size/scale selection dialog
- _File > Duplicate as Custom Symbol_ — custom symbol creation workflow

## Code Highlights

No Swift/Objective-C code samples in this session. The session focuses on the SF Symbols Mac app workflow. For code integration see the companion sessions:
- "SF Symbols in SwiftUI" (Session 10349)
- "SF Symbols in UIKit and AppKit" (Session 10251)
- "Create custom symbols" (Session 10250)

## Takeaways

- SF Symbols 3 adds Hierarchical and Palette rendering modes alongside the existing Monochrome and Multicolor modes; the app lets you preview all four before committing to code.
- The information inspector is the authoritative source for minimum OS/SF Symbols version per symbol name — always check it when supporting older platforms.
- Use _Copy Symbol_ (Cmd-C) with SF fonts for monochrome text-adjacent design; use the new _Copy Image_ (Option-Cmd-C) for multicolor or custom symbol design comps.
- Always use _Export Symbol_ (not _Copy Image_) to bring custom symbols into Xcode asset catalogs, to preserve full rendering mode annotations.

---
_Source: WWDC21 Session 10288 page (abstract, chapter summaries, code samples, and resource links)._
