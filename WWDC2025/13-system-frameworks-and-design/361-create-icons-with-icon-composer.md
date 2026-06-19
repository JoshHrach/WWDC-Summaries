# Create Icons with Icon Composer
**WWDC25 · Session 361** · [Watch](https://developer.apple.com/videos/play/wwdc2025/361/)

_Platforms:_ iOS 26, iPadOS 26, macOS Tahoe 26, watchOS

## Overview
Apple introduced Icon Composer, a new tool designed to streamline the creation of app icons that adopt the new Liquid Glass design system across iPhone, iPad, Mac, and Watch. Prior to Icon Composer, developers had to manage many separately sized and mode-specific image assets. Icon Composer takes in layered SVG and PNG exports from your design tool, applies real-time Liquid Glass properties, and outputs a single `.icon` file that Xcode uses to automatically generate all required assets for all platforms and appearance modes.

This session covers the Icon Composer workflow end-to-end: design preparation in vector tools, layer export, working inside Icon Composer, and delivery to Xcode.

## Key Topics

### Why Icon Composer
iOS 26 expands app icon appearance modes (Default, Dark, Clear Light, Clear Dark, Tinted Light, Tinted Dark) to iPhone, iPad, Mac, and Watch simultaneously. Managing separate assets for each combination is impractical. Icon Composer collapses this to one workflow: one file, one tool, all platforms.

### Design Workflow
Icons should be designed in a vector tool (Figma, Sketch, Illustrator, Photoshop) using one of Apple's 1024px icon templates (the same size now applies to iPhone, iPad, and Mac; Watch uses 1088px with the same grid). Source artwork should be kept flat, opaque, and simple — dynamic effects like blur, shadow, specular, and opacity are applied non-destructively inside Icon Composer.

Layers should be organized to separate Z-depth (background, mid, foreground) and to isolate distinct color regions. The Illustrator template includes a "Layer to SVG" script for automated export.

### Export Rules
- Export layers as SVGs for vector artwork; use PNGs for rasters (gradients, images) with transparent backgrounds.
- Number files to control Z-order (Icon Composer uses filenames to determine stacking).
- Do NOT include the rounded rectangle or circle mask — Icon Composer applies the correct crop automatically.
- Simple background colors and gradients are configured inside Icon Composer, not exported.
- Text must be converted to outlines before SVG export.

### Working in Icon Composer
- **Sidebar:** Canvas (background), groups (up to 4; control glass properties), layers, and platforms/appearances at the bottom.
- **Inspector:** Appearance controls per layer (toggle glass on/off, color, blend mode, opacity) and per group (Liquid Glass properties: specular, shadow type, translucency, blur).
- **Groups:** 1–4 groups per icon; glass properties are set at the group level.
- **Preview panel:** Live preview for all six appearances and platforms. Background and wallpaper can be changed to test legibility.
- **Dark mode tips:** Use "fills" to remap colors; keep legibility at minimum by ensuring at least one element is white in mono.
- **Chromatic vs. neutral shadows:** Use chromatic shadows for colorful art on white; use neutral for universal compatibility.

### Delivery
Save the `.icon` file, drag into Xcode, select it in the Project Editor under the app icon target. Xcode handles all size variants and appearances automatically.

## APIs & Frameworks

**Icon Composer (Xcode 26 / standalone tool)**
- **[NEW]** Icon Composer app — layered icon authoring with Liquid Glass
- **[NEW]** `.icon` file format — portable icon project file for Xcode
- Liquid Glass group properties: specular, shadow (neutral/chromatic), opacity, blend mode, fill, translucency, blur
- Appearance modes: Default, Dark, Mono (generates Clear Light, Clear Dark, Tinted Light, Tinted Dark)
- Platform targets: iOS/iPadOS, macOS, watchOS (all from one file)
- Property variants: per-appearance overrides for any property

**Xcode 26**
- Accepts `.icon` file in Project Editor for app icon assignment

**Design Resources**
- App icon templates for Figma, Sketch, Photoshop, Illustrator (1024px for iOS/iPadOS/macOS, 1088px for watchOS)
- Illustrator Layer-to-SVG export script

## Code Highlights
No code required — this is a design tool session. The delivery step is:
```
1. Save file.icon from Icon Composer
2. Drag file.icon into Xcode asset catalog or Project Editor
3. Select as app icon in target settings
4. Build and run — all sizes and appearances generated automatically
```

## Takeaways
- Use Icon Composer for any icon that benefits from Liquid Glass; use static PNGs in Xcode directly only for highly illustrative icons where glass properties don't apply.
- Design source art flat and opaque — apply specular, shadows, blur, and opacity non-destructively inside Icon Composer so adjustments per appearance are one-click changes.
- Take advantage of "Vary by Property" to create per-appearance variants only for properties that need to change (e.g., fill color for dark mode, shadow type for tinted mode).
- Chromatic shadows add vibrancy to colorful icons on light backgrounds; switch to neutral shadows for dark/mono appearances using per-appearance variants.

---
_Source: WWDC25 Session 361 page (abstract, chapter summaries, code samples, and resource links)._
