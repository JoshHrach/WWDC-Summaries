# Say Hello to the New Look of App Icons
**WWDC25 · Session 220** · [Watch](https://developer.apple.com/videos/play/wwdc2025/220/)

_Platforms:_ iOS 26, iPadOS 26, macOS Tahoe 26, watchOS 26

## Overview
This session introduces a completely reimagined app icon design language for iOS 26, iPadOS 26, macOS Tahoe 26, and watchOS 26. Inspired by the layered icons of visionOS and real-world glass properties, Apple created a new "Liquid Glass" material specifically for app icons. The material combines edge highlights, frostiness, and translucency to create a sense of depth and a look of icons lit from within, with gyro-driven light reflections on the home screen.

The session covers the new icon appearance modes, the unified iconography design system spanning all Apple platforms, and practical guidance for designing icons that work beautifully with the new material effects.

## Key Topics

### New Appearance Modes
iOS 26 adds new translucent appearance modes beyond standard light and dark:
- **Monochrome glass** — light or dark version
- **Dark tint** — adds color to the foreground layer
- **Light tint** — infuses color directly into the glass material
All modes are available on iPhone, iPad, and Mac. On Apple Watch, light-mode icons receive the updated look. Updated icons appear automatically on App Store product pages.

### Unified Design System
- New simplified grid with rounder corner radius (more concentric with UI/hardware)
- Circular format for watch icons uses a 1088 px canvas that overshoots the rounded rectangle for easier cross-platform translation
- macOS icons are now masked/extended to fit the canvas template automatically; complex compound shapes are auto-scaled
- Updated design templates available for Figma, Sketch, Photoshop, and Illustrator on the Apple Design Resources page

### Drawing Icons with Liquid Glass
- **Layering** — icons have a background layer and one or more foreground layers; stacking layers creates genuine dimensionality; the material adds translucency and per-layer shadows automatically
- **Translucency & blur** — easier than ever; works on light/dark and transparent modes; wallpaper shows through all translucent layers
- **Simplicity** — fewer overlapping shapes let material edge highlights shine; remove baked-in static effects (drop shadows, bevels) from source artwork
- **Perspectives** — frontal and flat views work better than realistic 3D; dimensional shapes should complement glass materiality
- **Backgrounds** — soft light-to-dark gradients recommended; use System Light and System Dark gradients instead of pure white/black; lean toward colored backgrounds for better light/dark distinction
- **Line weights** — avoid sharp/thin lines; use rounder corners so light travels smoothly; use bolder line weights to preserve details at small scale

## APIs & Frameworks

**Icon Composer** **[NEW]** — new Mac app for authoring Liquid Glass icons  
**Apple Design Resources** — updated templates for Figma, Sketch, Photoshop, Illustrator  
**Human Interface Guidelines: App icons** — updated guidance

_(This session is a design/guidelines session; there are no programmatic APIs. Icon appearance mode selection by the user is handled automatically by the OS.)_

## Code Highlights
No code samples — this is a purely design-focused session. Developers configure icon assets in the Xcode asset catalog as before; the OS applies Liquid Glass material effects automatically based on the provided layers.

## Takeaways
- Redesign icon artwork to use the new Liquid Glass layering system — start with background + foreground(s), remove any pre-baked shadows, bevels, or material effects.
- Use the new design grid templates (Figma/Sketch/Photoshop/Illustrator) and Icon Composer to build and preview icons across all modes.
- Favor simplified, flat or frontal shapes with rounder corners; let the material provide depth and nuance rather than baking it into the artwork.
- Download updated design resources from developer.apple.com/design/resources and test icons against all appearance modes (light, dark, monochrome, tinted).

---
_Source: WWDC25 Session 220 page (abstract, chapter summaries, code samples, and resource links)._
