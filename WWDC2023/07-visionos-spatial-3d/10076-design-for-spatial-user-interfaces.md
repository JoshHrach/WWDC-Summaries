# Design for Spatial User Interfaces
**WWDC23 · Session 10076** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10076/)

_Platforms:_ visionOS 1

## Overview
This session by Miquel Estany Rodriguez and Lorena Pazmino from the Apple Design team lays out the visual language and component model for building visionOS apps. It covers everything a designer or developer needs to know to translate existing iOS/macOS knowledge into great spatial UI: app icon construction, the glass material system, typography adaptations, vibrancy, ergonomic layout, and the new component set (tab bars, sidebars, ornaments, sheets, modals).

The session demonstrates how existing screen-based UI components map directly to visionOS equivalents, making the platform approachable for teams with iOS/iPadOS experience, while introducing unique spatial concepts like ornaments, concentric corner radii, the glass material, and dynamic eye-based focus feedback.

## Key Topics

### App Icons
visionOS icons are three-dimensional, using up to three flat layers: one background layer and up to two foreground layers. The system applies a glass overlay, specular highlights, and shadows automatically. All layers are 1024×1024 px; foreground layers must have transparent backgrounds. Keep graphics centered to avoid looking off-center when the icon expands on hover.

### Glass Material
The glass material is the defining visual language of visionOS. It is a system-defined translucent window surface that shows content and lighting from the user's physical environment through it. Key rules:
- Avoid solid opaque window backgrounds; they feel heavy and constricting.
- Glass dynamically adjusts contrast and color balance to the user's ambient lighting.
- The platform has no distinct light/dark appearance — glass and UI adapt to what is behind them.
- Use darker glass sub-materials for separating sections (e.g., sidebars), lighter sub-materials to highlight interactive elements (e.g., buttons), and darker materials to increase contrast in input fields.

### Typography
- All font styles use semantic names shared across platforms, tuned for legibility at any distance.
- Font weights are heavier on visionOS for legibility against vibrant materials: body uses **Medium** (not Regular), titles use **Bold** (not Semibold).
- Letter spacing (tracking) is increased for legibility.
- Two new platform-exclusive font styles: **Extra Large Title 1** and **Extra Large Title 2** for wide, editorial-style layouts. **[NEW]**
- Custom lightweight fonts can be hard to read; prefer heavier weights or system fonts.

### Vibrancy
Vibrancy brightens foreground content (text, symbols, fills) against the glass material in real time as the environment changes. Three levels: primary (standard text), secondary (descriptions/footnotes/subtitles), tertiary. Default to white text/symbols; if color is needed, use it as a background layer or full-button background. Always use system colors, which have been calibrated for legibility on glass.

### Ergonomic Layout
- Design for horizontal breadth rather than vertical height; human neck rotation is more comfortable side-to-side than up-and-down.
- Keep primary content centered; avoid placing UI too high or too low.
- Minimum interactive tap target area: **60 points** (combining visual size + surrounding empty space).
- Standard button: 44 pt visual + 8 pt margin each side = 60 pt target. Mini button: 28 pt visual in a 60 pt space.
- Use at least **16 points of spacing** between stacked buttons.
- Account for hover effect spacing: 4 pt padding between items in lists/menus to prevent hover overlap.

### Focus Feedback (Hover Effect)
System-provided components automatically display a subtle visual brightening when the user gazes at them. Custom interactive elements must define a region shape so the system can apply the hover effect. Keep small spacing between interactive elements to avoid hover overlap. Use continuous corner radius (smooth corners) for containing shapes.

### Concentric Corner Radii
All nested elements should have concentric corner radii: `outerCornerRadius = innerCornerRadius + padding`. Use continuous corners throughout.

### Core Components: Windows, Tab Bars, Sidebars
- Windows use the glass material with a window bar below for repositioning.
- **Tab bar**: vertical, floating on the left side of the window (vs. horizontal bottom bar on iOS). Auto-expands to show labels when gazed at; collapses when gaze moves away. **[NEW position/behavior]**
- **Sidebar**: lives alongside the tab bar inside the window for sub-navigation.

### Ornaments **[NEW]**
Ornaments are floating accessory UI elements attached to but outside the window bounds, slightly in front. Used for toolbars, persistent controls (e.g., Now Playing bar in Music), navigation selectors. Rules:
- Overlap the bottom edge of the window by 20 points when placed there.
- Use borderless buttons inside ornaments (interactivity is implied by context).
- Can appear/disappear, expand, or have their own navigation hierarchy.
- Best for persistent controls that shouldn't occlude main content.

### Menus, Popovers, and Sheets
- Menus and popovers can extend beyond the window boundary; they appear centered to where the user is looking.
- Selected-state buttons use black labels on white background (no colored highlights).
- Avoid white-background buttons unless in selected state.
- Sheets are modal views at the same Z depth as the parent window; the parent dims and pushes back.
- Secondary sheets stack in Z with additional dimming layers.
- Use push navigation within modals for nested views (back button, not close).
- Always place close buttons in the **top-left corner**.

## APIs & Frameworks

### visionOS UI (SwiftUI / UIKit)
- **Glass material** — system-defined `Material` for visionOS windows **[NEW]**
- **Ornament** — new visionOS presentation API for floating UI outside window bounds **[NEW]**
- **Tab bar (vertical)** — vertical `TabView` / sidebar layout for visionOS **[NEW]**
- **Hover effect** — automatic brightening on gaze; custom elements need a defined hover region **[NEW]**
- **Vibrancy** — `VibrancyEffect` applied to text/symbols on glass, three levels **[NEW]**
- **Continuous corners** / `RoundedRectangle(cornerRadius:style:.continuous)` — required for concentric elements
- **Extra Large Title 1 / Extra Large Title 2** — new visionOS-exclusive font styles **[NEW]**
- **60-point minimum tap target** — platform HIG requirement enforced in layout
- **Dynamic scale** — system window auto-scaling to preserve target areas at distance **[NEW]**
- **Three-layer app icons** (background + up to 2 foreground layers, 1024×1024 px each) **[NEW]**
- **Modal/sheet Z-depth stacking** — sheets push parent back in Z and dim it **[NEW]**

## Code Highlights
No code samples were presented. The session is design-focused; implementation details are in "Meet SwiftUI for spatial computing" (10109).

## Takeaways
- Use the glass material for all windows — avoid solid opaque backgrounds on visionOS.
- Apply the concentric corner radii formula (`outerRadius = innerRadius + padding`) everywhere nested UI shapes appear.
- Build ornaments for persistent toolbars and controls to add depth without occluding content.
- Always define a hover-effect region for custom interactive elements and maintain 4+ pt spacing between interactive items to prevent hover overlap.

---
_Source: WWDC23 Session 10076 page (abstract, chapter summaries, code samples, and resource links)._
