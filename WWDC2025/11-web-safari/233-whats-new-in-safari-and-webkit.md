# What's new in Safari and WebKit
**WWDC25 · Session 233** · [Watch](https://developer.apple.com/videos/play/wwdc2025/233/)

_Platforms:_ Safari 19 (iOS 26, iPadOS 26, macOS Tahoe 26, visionOS 26)

## Overview
This session surveys the new web technologies shipping in Safari 19, organized into four areas: animation (Scroll-driven Animations, Cross Document View Transitions), layout (CSS Anchor Positioning), visual effects (background-clip, the `shape()` function, `text-wrap: pretty`), and media (SVG icons, image improvements, format support).

The presenter builds a live demo site for a fictional online school throughout the session, using each new API to progressively enhance the page without JavaScript where possible.

## Key Topics

### Animation

**Scroll-driven Animations (Safari 19)**: CSS animations can now be linked to a scroll timeline rather than the default time-based timeline.
- `animation-timeline: scroll()` — ties animation progress to the nearest scrollable ancestor's scroll position (scroll from top to bottom drives 0% to 100% of the animation)
- `animation-timeline: view()` — ties animation progress to an element's position within the viewport (element entering/leaving the viewport drives the animation)
- `animation-range` — controls which portion of the scroll or view timeline drives the animation (e.g., `animation-range: entry 0% exit 100%`)
- `@scroll-timeline` at-rule — define named scroll timelines
- `@view-timeline` at-rule — define named view timelines
- Note: always respect `prefers-reduced-motion` when adding scroll-driven motion

**Cross Document View Transitions (Safari 19)**: Multi-page apps can now use the View Transitions API across page navigations.
- `@view-transition { navigation: auto; }` — opt in to cross-document view transitions in CSS; no JavaScript required for basic transitions
- `view-transition-name` CSS property — tag elements for matched transitions between pages
- `::view-transition-*` pseudo-elements — style the transition animation layers
- Works for same-origin navigations; transitions fire automatically on link clicks/form submissions
- Supports both "old" (outgoing) and "new" (incoming) page states during the transition

### Layout

**CSS Anchor Positioning (Safari 19)**: Position elements relative to another element (the "anchor") anywhere in the document — without JavaScript, regardless of DOM hierarchy or scroll position.
- `anchor-name: --my-anchor` — declare an element as an anchor
- `position: absolute; position-anchor: --my-anchor` — attach a positioned element to an anchor
- `anchor()` function — use anchor edges in inset properties: `top: anchor(--my-anchor bottom)`, `left: anchor(--my-anchor right)`
- `anchor-size()` function — use anchor dimensions: `width: anchor-size(--my-anchor width)`
- `@position-try` at-rule — define alternate positions if the primary position overflows the viewport
- `position-try-fallbacks` property — list fallbacks to try in order
- Key use case: tooltips, popovers, dropdown menus, and contextual UI positioned relative to triggers without JS

### Visual effects

**`background-clip: text` (widely supported, now reliably cross-browser)**: Clip an element's background to the shape of its text — enables gradient text without SVG.
```css
.gradient-text {
    background: linear-gradient(to right, #f97316, #8b5cf6);
    background-clip: text;
    -webkit-background-clip: text;
    color: transparent;
}
```

**`shape()` function (Safari 19)**: Draw arbitrary vector paths in CSS for `clip-path`, `offset-path`, and similar properties — replaces the need for SVG `<clipPath>` elements for complex non-rectangular clipping.
- Syntax: `clip-path: shape(from <start> curve to <end> via <controls>, ...)`
- Supports: `move`, `line`, `hline`, `vline`, `curve` (Bézier), `smooth` (smooth Bézier), `arc`, `close`
- Pairs with CSS transitions and animations for animated clip paths

**`text-wrap: pretty` (Safari 19)**: Intelligent last-line optimization for multiline text — prevents awkward one-word final lines ("orphans") by redistributing text across lines for a more visually balanced block of text. Zero JS, pure CSS.
```css
p { text-wrap: pretty; }
```

### Media

**SVG as `<img>` icon replacement**: Safari 19 improves SVG rendering fidelity and performance when used as `<img src="icon.svg">`. SVG icons can now respond to CSS properties like `currentColor` when embedded appropriately.

**Image improvements**: Better support for wide color images (Display P3) in `<img>` and `<picture>`. Improved AVIF decode performance.

**Media format support**: Expanded support for existing standards in the Web Speech API — `speechSynthesis` improvements for additional voices and languages.

## APIs & Frameworks

### CSS — Scroll-driven Animations
- **`animation-timeline: scroll()`** **[NEW in Safari 19]** — scroll-linked animation timeline
- **`animation-timeline: view()`** **[NEW in Safari 19]** — element-visibility-linked timeline
- **`animation-range`** **[NEW in Safari 19]** — `entry`, `exit`, `cover`, `contain` range keywords
- **`@scroll-timeline`** at-rule **[NEW in Safari 19]**
- **`@view-timeline`** at-rule **[NEW in Safari 19]**
- `prefers-reduced-motion` media query — use with scroll-driven animations

### CSS — Cross Document View Transitions
- **`@view-transition { navigation: auto; }`** **[NEW in Safari 19]** — automatic cross-document transitions
- **`view-transition-name`** **[NEW in Safari 19]** — tag elements for matched transitions
- `::view-transition-old()` / `::view-transition-new()` — pseudo-elements for styling transition layers
- `::view-transition-image-pair()` / `::view-transition-group()` — additional pseudo-elements

### CSS — Anchor Positioning
- **`anchor-name: --name`** **[NEW in Safari 19]** — declare an anchor
- **`position-anchor: --name`** **[NEW in Safari 19]** — attach positioned element to anchor
- **`anchor()`** function **[NEW in Safari 19]** — reference anchor edges in inset properties
- **`anchor-size()`** function **[NEW in Safari 19]** — reference anchor dimensions
- **`@position-try`** at-rule **[NEW in Safari 19]** — define alternate positions for overflow avoidance
- **`position-try-fallbacks`** property **[NEW in Safari 19]**

### CSS — Visual Effects
- `background-clip: text` — `text` keyword (improved cross-browser reliability)
- **`shape()`** function **[NEW in Safari 19]** — for `clip-path`, `offset-path`; commands: `from`, `move`, `line`, `hline`, `vline`, `curve`, `smooth`, `arc`, `close`
- **`text-wrap: pretty`** **[NEW in Safari 19]** — balanced last-line orphan prevention

### Web APIs
- Web Speech API (`speechSynthesis`) — improved voice/language support

## Code Highlights

```css
/* Scroll-driven progress bar */
.progress-bar {
    animation: grow linear;
    animation-timeline: scroll();
}
@keyframes grow {
    from { transform: scaleX(0); }
    to { transform: scaleX(1); }
}

/* Cross-document view transition */
@view-transition {
    navigation: auto;
}
.hero-image { view-transition-name: hero; }

/* Anchor positioning — tooltip */
.trigger { anchor-name: --trigger; }
.tooltip {
    position: absolute;
    position-anchor: --trigger;
    top: anchor(--trigger bottom);
    left: anchor(--trigger left);
}
@position-try --flip-above {
    top: auto;
    bottom: anchor(--trigger top);
}
.tooltip { position-try-fallbacks: --flip-above; }

/* Gradient text */
.headline {
    background: linear-gradient(135deg, #f97316, #8b5cf6);
    background-clip: text;
    -webkit-background-clip: text;
    color: transparent;
}

/* Shape clip path */
.card {
    clip-path: shape(from 0% 0%, line to 100% 0%, line to 100% 80%,
                     curve to 50% 100% via 75% 100%, curve to 0% 80% via 25% 100%,
                     close);
}

/* text-wrap: pretty */
p.description { text-wrap: pretty; }
```

## Takeaways
- Adopt `animation-timeline: scroll()` and `view()` for parallax, progress bars, and scroll-triggered animations without JavaScript — always pair with `@media (prefers-reduced-motion: reduce)` wrappers.
- Use `@view-transition { navigation: auto; }` with `view-transition-name` to add elegant page transitions to multipage sites in a single CSS file; no JavaScript required for basic transitions.
- Replace JS-positioned tooltips and popovers with CSS Anchor Positioning — `anchor-name`, `position-anchor`, and `anchor()` with `@position-try` fallbacks eliminate the most common need for `getBoundingClientRect()` calculations.
- Apply `text-wrap: pretty` globally to body text paragraphs to eliminate orphaned words automatically; it has no visual downside and improves typographic quality at zero cost.

---
_Source: WWDC25 Session 233 page (abstract, chapter summaries, code samples, and resource links)._
