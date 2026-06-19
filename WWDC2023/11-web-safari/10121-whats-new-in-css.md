# What's New in CSS
**WWDC23 · Session 10121** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10121/)

_Platforms:_ Safari 16.x / Safari 17, WebKit

## Overview
This session surveys more than 140 new web technologies shipped in WebKit through 2023, with deep focus on four CSS areas: layout (Masonry, margin-trim), color (wide-gamut P3 with new color models, gradients, color mixing, relative color syntax), pseudo-classes for robust form and language-direction styling, and typography (line-height units, font-size-adjust, text-box-trim, counter styles). The session distinguishes features already shipped in Safari 16.x, features arriving in Safari 17, and work-in-progress previews available in Safari Technology Preview.

## Key Topics

### Layout
- **Masonry Layout** (Safari Technology Preview) — extends CSS Grid with `grid-template-rows: masonry` to pack items into a Masonry pattern without JavaScript. Combines with all standard Grid capabilities (`fr` units, `minmax`, `auto-fill`).
- **`margin-trim`** (Safari 16.4) **[NEW]** — removes margins from child elements that push against the container edge. `margin-trim: block` trims block-axis margins; `margin-trim: inline` trims inline-axis margins. More robust than manually zeroing first/last child margins.

### Wide-Gamut Color
- **Display P3 color gamut** — 50% more colors than sRGB; supported on Apple hardware since 2015. CSS color gamut media query (`@media (color-gamut: p3)`) shipped in Safari 10.0.
- **New color model functions** (Safari 15.0 / 15.4) — `lch()`, `oklch()`, `lab()`, `oklab()` — represent colors beyond sRGB; each takes Lightness plus Chroma+Hue (LCH/OKLCH) or A+B axes (LAB/OKLAB).
- **`color()` function** — specify any color gamut explicitly: `color(display-p3 R G B / A)`.
- **Relative Color Syntax** **[NEW]** — `color(from <existing-color> <function> ...)` — derive a new color from an existing one by manipulating individual channels (e.g., halve the lightness in LAB, drain chroma in OKLCH, set opacity in sRGB).
- **Color interpolation in gradients** (Safari 16.2) — explicitly specify color space for gradient calculation: `linear-gradient(in oklab, white, blue)`. Different spaces produce visually distinct mid-tone transitions.
- **`color-mix()`** (Safari 16.2) **[NEW]** — mix two colors in a specified color space: `color-mix(in oklch, blue, white 30%)`. Supports arbitrary ratios; totals below 100% yield translucent results. Works with `currentcolor`.

### Pseudo-Classes
- **`:user-valid` / `:user-invalid`** (Safari 16.5) **[NEW]** — style form fields as valid/invalid only after the user has interacted with them (not immediately on page load like `:valid`/`:invalid`). Particularly powerful combined with `:has()`.
- **`:has()` enhancements** — now supports `:has(:lang())` for language-conditional styling and media pseudo-classes (`:has(:playing)`, `:has(:paused)`) for audio/video state.
- **`:dir()`** **[NEW]** — select elements based on text direction (`:dir(ltr)`, `:dir(rtl)`); enables direction-aware transforms without `[dir="rtl"]` attribute selectors.

### Typography
- **Line-height units** **[NEW]** — `lh` (line height of current element) and `rlh` (root element line height); connect layout dimensions to typographic rhythm (e.g., `padding: 2rlh`, `margin-block: 1rlh`).
- **`font-size-adjust`** (basic: Safari 16.4; advanced: Safari 17) **[NEW]** — normalizes the visual size of fallback fonts by aligning their x-height ratio to the primary font. `from-font` value lets the browser automatically determine the ratio. Two-value syntax allows `cap-height`, `ch-width`, `ic-width`, or `ic-height` metrics. The `size-adjust` descriptor works in `@font-face` rules.
- **`text-box-trim`** (Safari Technology Preview) — trims the reserved whitespace above/below glyphs for precise vertical alignment and layout. Replaces the earlier `leading-trim` name; syntax still evolving.
- **Counter Styles** (Safari 17) **[NEW]** — `@counter-style` rule allows custom list numbering systems with `system`, `symbols`, and `additive-symbols` descriptors. Supports any script, arbitrary symbols, and emoji. W3C Internationalization WG's Ready-made Counter Styles document provides copy-paste snippets for hundreds of cultures.
- **CSS Nesting** (Safari 16.5) **[NEW]** — native CSS nesting without a preprocessor.

### Additional Features Shipped in 2023
- **`@property`** (Safari 16.4) — define custom property types and initial values.
- **Media Queries range syntax** (Safari 16.4) — `@media (width >= 600px)` style comparisons.
- **Last baseline alignment** (Safari 16.2) — for Grid and Flexbox.
- **`font-variant-alternates` functions and `@font-feature-values`** (Safari 16.2) — advanced OpenType feature access.
- **`contain-intrinsic-size`** (Safari 17) — layout containment with intrinsic size hint.
- **`text-transform: full-width` / `full-size-kana`** (Safari 17) — CJK text transform support.
- **Feature detection for font tech/format** (Safari 17) — `@supports font-tech()` / `@supports font-format()`.

## APIs & Frameworks

- CSS Grid `grid-template-rows: masonry` **[NEW — Safari TP]** — Masonry layout axis
- `margin-trim` property **[NEW — Safari 16.4]** — values: `block`, `inline`, `block-start`, `block-end`, `inline-start`, `inline-end`
- `lch()`, `oklch()`, `lab()`, `oklab()` color functions — wide-gamut color models (Safari 15.x)
- `color()` function — explicit color gamut: `display-p3`, `srgb`, `srgb-linear`, `a98-rgb`, `prophoto-rgb`, `rec2020`
- Relative Color Syntax `color(from <color> ...)` **[NEW]**
- `color-mix()` function **[NEW — Safari 16.2]**
- Color interpolation in gradients: `linear-gradient(in <color-space>, ...)` **[NEW — Safari 16.2]**
- `@media (color-gamut: p3)` — color gamut media query
- `:user-valid` pseudo-class **[NEW — Safari 16.5]**
- `:user-invalid` pseudo-class **[NEW — Safari 16.5]**
- `:dir(ltr)` / `:dir(rtl)` pseudo-class **[NEW]**
- `:has(:lang())` — language-conditional selector **[NEW]**
- `:has(:playing)` / `:has(:paused)` — media state selectors **[NEW]**
- `lh` unit (line height) **[NEW]**
- `rlh` unit (root line height) **[NEW]**
- `font-size-adjust` property **[NEW — Safari 16.4 basic, Safari 17 advanced]**
- `font-size-adjust: from-font` **[NEW — Safari 17]**
- `size-adjust` descriptor in `@font-face` **[NEW — Safari 17]**
- `text-box-trim` property **[NEW — Safari TP, evolving]**
- `@counter-style` rule **[NEW — Safari 17]**
- `CSS Nesting` **[NEW — Safari 16.5]**
- `@property` rule (Safari 16.4)
- `drawingBufferColorSpace` on WebGL Canvas (Safari 16.4) — P3 support in WebGL
- Safari Technology Preview — Feature Flags pane (Safari 17)

## Code Highlights

```css
/* Masonry layout (Safari Technology Preview) */
main {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(14rem, 1fr));
  grid-template-rows: masonry;
}

/* margin-trim removes child margins that touch the container edge */
.card {
  padding: 2rlh;
  margin-trim: block;
}
h2, p { margin: 1rlh 0; }

/* Wide-gamut P3 color */
.vibrant { color: color(display-p3 1 0.2 0.1); }

/* Relative Color Syntax — derive from existing color */
.muted { color: oklch(from var(--brand-color) l calc(c / 3) h); }
.translucent { color: color(from var(--brand-color) srgb r g b / 0.7); }

/* color-mix */
a:hover { color: color-mix(in srgb, currentcolor 60%, white); }

/* Gradient with explicit color space */
.banner {
  background: linear-gradient(in oklab, white, royalblue);
}

/* :user-invalid — only shows error after user interaction */
:has(input:user-invalid) label::before {
  content: "✗ ";
  color: red;
}

/* :dir — direction-aware transform */
:dir(rtl) .icon { transform: scaleX(-1); }

/* Line-height units for vertical rhythm */
html { line-height: 1.4; }
section { padding: 2rlh; }
h2, p { margin-block: 1rlh; }

/* font-size-adjust — normalize fallback font sizes */
article code { font-size-adjust: 0.47; }        /* manual ratio */
article code { font-size-adjust: from-font; }   /* auto (Safari 17) */

/* @counter-style — custom emoji list */
@counter-style emoji-counter {
  system: cyclic;
  symbols: "🌟" "🔥" "💎";
  suffix: " ";
}
ol { list-style: emoji-counter; }
```

## Takeaways
- `color-mix()` and Relative Color Syntax eliminate the need for preprocessor color functions for palette generation and hover-state derivation in plain CSS.
- Wide-gamut P3 color is now broadly supported — use `oklch()` or `color(display-p3 ...)` for colors outside sRGB, and `@media (color-gamut: p3)` for conditional P3 styling.
- `:user-valid`/`:user-invalid` finally make native CSS form validation styling practical without JavaScript; combining with `:has()` enables label-level error styling.
- `margin-trim` and line-height units (`lh`/`rlh`) are purpose-built tools for typographic precision that previously required fragile first/last-child hacks or JavaScript layout loops.
- `@counter-style` and CSS Nesting ship in Safari 17, completing two long-requested CSS features for internationalization and author ergonomics.

---
_Source: WWDC23 Session 10121 page (abstract, transcript, and code samples)._
