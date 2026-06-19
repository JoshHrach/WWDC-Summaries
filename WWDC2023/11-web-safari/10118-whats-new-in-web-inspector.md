# What's new in Web Inspector
**WWDC23 · Session 10118** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10118/)

_Platforms:_ macOS Sonoma 14, iOS 17, iPadOS 17 (Web Inspector runs on macOS, targets all platforms)

## Overview
Safari's Web Inspector receives significant enhancements in 2023 across four areas: richer typography inspection with interactive variable font axis controls, a new User Preference Overrides popover to emulate `prefers-color-scheme`, `prefers-reduced-motion`, and `prefers-contrast` without changing system settings, new DOM node badges for scroll containers and event listeners, and a new Symbolic breakpoints type plus enhanced breakpoint configuration options.

These tools help web developers build more accessible, visually polished experiences and debug JavaScript more effectively — all without modifying source code.

## Key Topics

**Typography Inspection (Font Panel improvements)**
- Font panel in Elements tab sidebar shows: primary font name, font size/style/weight/stretch, supported feature properties and their used values (ligatures, small caps, numeric styles, etc.)
- Synthetic bold/oblique warnings **[NEW]**: flag when WebKit artificially synthesizes bold or italic rather than using a dedicated font file
- Variable font support **[NEW]**: when an element uses a variable font, the Font panel lists all variation axes with their tag, label, min/max range, and current value; includes **interactive sliders and input fields** to edit axis values live on the page; CSS is written to the Styles panel automatically

**User Preference Overrides**
- New User Preference Overrides popover **[NEW]** in the Elements tab toolbar
- Override CSS media features per-inspected-page (not system-wide) while Web Inspector is open:
  - `prefers-color-scheme` → Appearance: Light / Dark
  - `prefers-reduced-motion` → Accessibility: Reduce Motion On / Off / Default
  - `prefers-contrast` → Accessibility: Increase Contrast On / Off / Default
- Previous standalone "color scheme" button consolidated into this popover
- Badge icon changes color to indicate when an override is active

**Element Badges**
- Existing badges: CSS Grid, CSS Flexbox (click to toggle page overlay showing grid/flex guides)
- Scroll badge **[NEW]**: appears next to elements whose content overflows and creates a scroll container — helps identify unwanted scroll
- Event badge **[NEW]**: appears next to elements with JavaScript event listeners (built-in events and custom events); click badge to reveal popover showing:
  - Event type, handler function name, source location
  - Configuration: bubbles, once, passive, capture
  - Options: disable the listener, set an event breakpoint to pause execution

**Breakpoint Enhancements**
- JavaScript line breakpoints (existing): click gutter in Sources tab
- Breakpoint configuration (existing + improved): condition expression, ignore count, actions (eval JavaScript, log message, Probe Expression)
- "Automatically continue after evaluating expressions" option for action-only breakpoints (logging without pause)
- New breakpoint types via the Plus menu:
  - URL Breakpoints — pause on `fetch()` / XHR matching a URL
  - Event breakpoints — pause on specific DOM events
  - Micro task / animation frame / timeout / interval breakpoints
  - **Symbolic breakpoints** **[NEW]**: pause before a named function is invoked (exact match or regex); useful for built-in APIs (e.g., `navigator.share`) or multiple functions with the same name

**Other Safari Developer Feature Improvements (noted)**
- Streamlined device pairing and remote inspection
- Quick open-in-Simulator shortcut
- Refreshed Responsive Design Mode
- Details in companion session "Rediscover Safari developer features" (10262)

## APIs & Frameworks

_Web Inspector is a developer tool, not a developer-facing API. The following are the CSS media features and properties that the new tools surface:_

**CSS Media Features (inspectable/overridable)**
- `prefers-color-scheme: light | dark` — inspectable and overridable **[NEW override UI]**
- `prefers-reduced-motion: no-preference | reduce` — inspectable and overridable **[NEW override UI]**
- `prefers-contrast: no-preference | more | less` — inspectable and overridable **[NEW override UI]**

**CSS Font Properties (surfaced in Font panel)**
- `font-family`, `font-size`, `font-style`, `font-weight`, `font-stretch`
- `font-feature-settings` (supported features and values)
- `font-variation-settings` — variation axes shown and editable **[NEW interactive controls]**

**Web Inspector Tools**
- Elements tab > Font panel — synthetic bold/oblique warnings **[NEW]**, variable font axis editors **[NEW]**
- Elements tab > User Preference Overrides popover **[NEW]**
- Elements tab > Node tree — Scroll badge **[NEW]**, Event badge **[NEW]**
- Sources tab > Breakpoints — Symbolic breakpoints **[NEW]**
- Breakpoint editor — log actions, eval actions, Probe Expressions, auto-continue **[existing, improved UX]**

## Code Highlights

No code samples — this is a developer tooling session. Key CSS patterns that the new tools help authors validate:

```css
/* Variable font usage — axes now editable live in Font panel */
h1 {
  font-variation-settings: 'wght' 650, 'wdth' 90, 'GRAD' 20;
}

/* Reduced motion pattern — testable via User Preference Overrides */
.photo-zoom { animation: zoom 0.4s ease; }

@media (prefers-reduced-motion: reduce) {
  .photo-zoom { animation: fade 0.8s ease; }
}

/* Dark mode pattern — overridable in Appearance section */
@media (prefers-color-scheme: dark) {
  :root { --bg: #1a1a1a; --fg: #f5f5f5; }
}
```

## Takeaways
- Use the new User Preference Overrides popover to test dark mode, reduced motion, and high contrast without touching System Settings — it only affects the inspected page.
- The Symbolic breakpoints type is invaluable for debugging calls to browser built-ins (e.g., `navigator.share`, `fetch`) or any function whose source location you don't know.
- When building with variable fonts, use the Font panel's interactive axis controls to dial in `font-variation-settings` values live without editing CSS files.
- Watch for the Scroll badge in the Elements tree — it immediately identifies the element responsible for unwanted horizontal/vertical scroll, a common and elusive layout bug.

---
_Source: WWDC23 Session 10118 page (abstract, chapter summaries, code samples, and resource links)._
