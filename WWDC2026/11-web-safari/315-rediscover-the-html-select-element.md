# Rediscover the HTML Select Element

**WWDC26 · Session 315** · [Watch](https://developer.apple.com/videos/play/wwdc2026/315/)

_Platforms:_ Safari 27, WebKit, web standards (also Chrome 135+)

## Overview

The HTML `<select>` element is receiving a major upgrade that finally allows full CSS styling while preserving its built-in accessibility and native form-control behavior. The key enabler is a new CSS property value: `appearance: base-select`. Applying it to a `<select>` opts the element out of the browser's default rendering and into a mode where every part — the button, the dropdown popup, option items, and the checkmark — can be targeted and styled independently with standard CSS.

The session walks through building two real select menus on a photographer's portfolio site: a simple "Sort by" text menu and a rich category picker with SVG icons arranged in a CSS grid. Both examples are built with progressive enhancement in mind — browsers that do not support Customizable Select automatically fall back to the native popup, so no user loses functionality and no JavaScript shim is needed.

A new `<selectedcontent>` HTML element enables an advanced pattern: embedding the rich content of the currently selected `<option>` (including icons and labels) directly into the select's trigger button, keeping the button and dropdown visually consistent without JavaScript.

## Key Topics

### Style the Select Button (2:32)
Apply `appearance: base-select` to the `<select>` to unlock styling. The button then accepts standard CSS properties (background, border, padding, font). The `::picker-icon` pseudo-element targets the dropdown arrow; assign a custom SVG with `content: url(...)`. The `:open` pseudo-class applies styles while the listbox is visible.

### Customize the Drop-Down (3:47)
The listbox popup is targeted via `::picker(select)`. Apply `appearance: base-select` to it as well, then style with padding, border-radius, box-shadow, and margin-top. Individual options respond to `:checked` (the selected option) and `:not(:checked)`. The `::checkmark` pseudo-element on `option` controls the indicator icon — replace the default with `content: url(checkmark.svg)` or set `display: none` to remove it.

### Go Beyond Text Options (5:00)
`<option>` elements can contain rich HTML: `<img>` tags with SVG icons, `<span>` for labels. Screen readers read the option's text content; use empty `alt=""` on decorative images to prevent double-announcement. The popup container (`::picker(select)`) accepts `display: grid` with `grid-template` and `gap`, enabling multi-column layouts of icon options.

### The `<selectedcontent>` Element (6:50)
Place a `<button>` as the first child of `<select>`, then nest `<selectedcontent>` inside it. The browser automatically mirrors the DOM content of the currently selected `<option>` into `<selectedcontent>`, making the chosen icon and label visible in the trigger button — all without JavaScript.

### Fallback for Unsupported Browsers (7:46)
Because the feature is built on the native `<select>` element, browsers that do not support Customizable Select ignore the new CSS and render the standard system popup. Progressive enhancement is automatic.

## APIs & Frameworks

**CSS — Customizable Select**
- **[NEW]** `appearance: base-select` — opts `<select>` into the styleable mode (and `::picker(select)` into its styleable mode)
- **[NEW]** `::picker-icon` pseudo-element — the dropdown indicator arrow on the select button
- **[NEW]** `::picker(select)` pseudo-element — the popup listbox container
- **[NEW]** `::checkmark` pseudo-element — the checkmark on `<option>` elements
- **[NEW]** `:open` pseudo-class — matches `<select>` while its popup is open
- `:checked` pseudo-class — matches the currently selected `<option>`
- `content: url(...)` — assign custom SVG/image to pseudo-elements
- `display: grid`, `grid-template`, `gap` — usable on `::picker(select)` for multi-column option layouts
- Standard box-model properties (padding, border, border-radius, box-shadow, background-color, color, font-weight) on `<select>`, `::picker(select)`, and `<option>`

**HTML — Customizable Select**
- **[NEW]** `<selectedcontent>` element — auto-mirrors the selected option's DOM content into the trigger button
- **[NEW]** `<button>` as first child of `<select>` — custom trigger button
- Rich content in `<option>`: `<img>`, `<span>`, SVG — now fully supported
- `<select>`, `<option>` — standard elements; behavior and accessibility unchanged

**Accessibility**
- Native `<select>` semantics retained; keyboard navigation, screen reader announcement, and OS-level accessibility all preserved
- `alt=""` on decorative images inside `<option>` prevents screen reader double-announcement

## Code Highlights

Opt in and style the button:
```css
select {
  appearance: base-select;
  background-color: var(--green-10);
  border: none;
  padding: 0.6em 1em;
}
select:open { background-color: var(--green-100); color: white; }
select:open::picker-icon { content: url(icons/arrow-white.svg); }
```

Style the popup and options:
```css
::picker(select) {
  appearance: base-select;
  padding: 4px;
  margin-top: 0.5em;
  border: 1px solid rgba(0,0,0,0.2);
  border-radius: 9px;
  box-shadow: 0 4px 20px rgba(0,0,0,0.2);
}
option:checked { font-weight: 600; }
option::checkmark { content: url(checkmark.svg); width: 0.65em; }
```

Rich options with grid layout:
```html
<option value="flowers">
  <img src="icons/flower.svg" alt="">
  <span class="text">Flowers</span>
</option>
```
```css
::picker(select) { display: grid; grid-template: 1fr 1fr / 1fr 1fr 1fr; gap: 1rem; }
```

`<selectedcontent>` pattern:
```html
<select>
  <button><selectedcontent></selectedcontent></button>
  <option value="flowers"><img src="..." alt=""> Flowers</option>
</select>
```

## Takeaways

- `appearance: base-select` on `<select>` is the single opt-in that unlocks the entire Customizable Select system — no JavaScript required.
- New pseudo-elements (`::picker(select)`, `::picker-icon`, `::checkmark`) and pseudo-classes (`:open`, `:checked`) provide surgical styling hooks for every part of the control.
- `<option>` elements now support arbitrary HTML content, enabling icon-based or grid-layout pickers without losing accessibility.
- Fallback to the native popup is fully automatic in older browsers — progressive enhancement is inherent to the design.

---
_Source: WWDC26 Session 315 page (abstract, chapter summaries, code samples, and resource links)._
