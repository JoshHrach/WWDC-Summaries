# Discover Web Inspector improvements
**WWDC21 · Session 10031** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10031/)

_Platforms:_ macOS Monterey 12, iOS 15, iPadOS 15, tvOS 15

## Overview
Web Inspector is Safari's built-in suite of developer tools for inspecting, debugging, and auditing web pages across macOS, iOS, iPadOS, and tvOS. This session covers three major additions in 2021: CSS Grid overlay visualization, expanded breakpoint configuration options, and an in-app audit editor.

CSS Grid overlays give developers a visual overlay directly on the page showing track sizes, line numbers, named areas, and extended grid lines. The overlay system is performance-optimized to support dozens of simultaneous grids with smooth scrolling. The new Layout panel in the Details sidebar centralizes all overlay controls and color settings.

Breakpoints receive powerful new configuration options—conditions referencing Web Inspector Console API, ignore counts, multiple action types (evaluate JS, log message, system beep, probe expression), and an "automatically continue" flag—all now available for every breakpoint type (JavaScript, event, DOM, URL, exception/assertion), not just line breakpoints. A new audit Edit Mode allows creating and editing test cases directly inside Web Inspector without export/import round-trips.

## Key Topics
- **CSS Grid Overlays** – Clickable `grid` badge on DOM nodes; Layout panel with per-grid color, toggle, and label options (track sizes, line numbers, line names, area names, extended gridlines); works on iOS 15/iPadOS 15 via remote inspection from Safari 15/Safari Technology Preview.
- **Breakpoint Enhancements** – Condition expressions with `$0` (selected element) and `$event` Console API references; ignore count; actions: Evaluate JavaScript, Log Message, System Beep, Probe Expression; emulate user gesture flag for JS actions; automatically continue option; all options available on all five breakpoint types.
- **Audit Edit Mode** – New Edit button in Audit tab Navigation Sidebar; create Test Case or Test Group; configure name, description, minimum audit version, setup script, and test function; `WebInspectorAudit` API available inside tests; changes auto-saved; built-in Demo Audit and Accessibility test group remain read-only (but duplicatable).
- **Elements / Styles panel refactor** – Styles sidebar now an independent panel, enabling three-panel layouts with Layout and Styles visible simultaneously.

## APIs & Frameworks
- **Web Inspector / WebKit DevTools**
  - CSS Grid overlay via Layout panel **[NEW]**
  - `grid` badge on DOM elements **[NEW]**
  - Layout panel (Details sidebar) **[NEW]**
  - Breakpoint condition with `$0`, `$event`, `$1`–`$n` Console API expressions **[NEW]**
  - Breakpoint ignore count option **[NEW]**
  - Breakpoint action: Evaluate JavaScript (with "emulate user gesture" flag) **[NEW]**
  - Breakpoint action: Log Message (template string literal)
  - Breakpoint action: System Beep
  - Breakpoint action: Probe Expression (results in Probe panel)
  - "Automatically continue after evaluating" breakpoint option **[NEW]**
  - All breakpoint config options extended to Event, DOM, URL, Exception/Assertion breakpoints **[NEW]**
  - Audit Edit Mode **[NEW]**
  - `WebInspectorAudit` object (test and setup script API)
  - Export Audit / Import Audit JSON workflow
- **Web Platform APIs referenced**
  - `navigator.share()` (Web Share API)
  - `XMLHttpRequest`, `fetch` (for URL breakpoints)
  - CSS Grid layout properties: `grid-area`, fractional units (`fr`), `min-content`, named lines, named areas

## Code Highlights
Setting a negative grid-area in the Styles panel to pin an element relative to the last explicit grid line:
```css
/* grid-area: row-start / column-start */
grid-area: -3 / -3;
```

Sample audit test function checking font families:
```javascript
function() {
  const expectedFonts = ['ui-rounded', 'system-ui', '-apple-system'];
  const failingNodes = Array.from(document.querySelectorAll('*')).filter(el => {
    const font = getComputedStyle(el).fontFamily;
    return !expectedFonts.some(f => font.includes(f));
  });
  return { level: failingNodes.length ? 'fail' : 'pass', elements: failingNodes };
}
```

## Takeaways
- CSS Grid overlays provide immediate visual feedback for complex layouts, including support for negative line numbers, named areas, and extended grid lines across all writing modes.
- All five breakpoint types now support the full set of configuration options (conditions, ignore counts, actions, auto-continue), eliminating the need for `console.log` debugging in most scenarios.
- Audit Edit Mode removes the export/import friction for creating custom test suites, making it practical to enforce design system and accessibility rules as part of a regular development workflow.
- Web Inspector is open source (WebKit project); Safari Technology Preview ships updates approximately every two weeks.

---
_Source: WWDC21 Session 10031 page (abstract, chapter summaries, code samples, and resource links)._
