# Supporting Dark Mode in Your Web Content
**WWDC19 · Session 511** · [Watch](https://developer.apple.com/videos/play/wwdc2019/511/)

_Platforms:_ iOS 13, macOS Catalina 10.15, Safari 12.1+

## Overview
With iOS 13 and macOS Mojave introducing system-wide Dark Mode, web content displayed in Safari, WKWebView, and Mail needs to explicitly opt in to dark appearance support. WebKit and Safari will not auto-darken arbitrary web pages — the web developer is responsible for providing appropriate dark styles.

This session covers three areas: adopting dark mode styles for websites using the new `color-scheme` CSS property and `prefers-color-scheme` media query; handling dark mode in HTML email campaigns; and using the updated Web Inspector (with its own dark appearance and a color scheme toggle) to debug dark mode web content.

## Key Topics

### The `color-scheme` CSS Property **[NEW]**
- Declaring `color-scheme: light dark;` on the `:root` element tells the rendering engine both modes are supported.
- With this property set, the browser automatically adjusts default text/background colors, form controls, scrollbars, and other named system colors to match the current appearance.
- Without this declaration, Safari leaves the page in light mode — no auto-darkening is applied.
- This is the minimal change needed to get a browser-default dark appearance for simple pages.

### `prefers-color-scheme` Media Query **[NEW]**
- Use `@media (prefers-color-scheme: dark)` to apply custom CSS overrides for dark mode.
- Mirrors responsive design breakpoints — treat dark mode as another device/environment condition.
- Best practice: define colors as CSS custom properties (variables) in the `:root` rule, then override just the variable values inside the media query rather than rewriting all rules.

### CSS Custom Properties (Variables) for Color Management
- Define semantic color variables (e.g., `--text-color`, `--background-color`) on `:root` for light mode defaults.
- Override the same variable names in the `prefers-color-scheme: dark` block for dark values.
- All other rules reference `var(--variable-name)` — this reduces duplication and makes dual-mode maintenance straightforward.

### Dark Mode Images with `<picture>` Element
- Use the `<picture>` element with a `<source media="(prefers-color-scheme: dark)">` to serve alternative images for dark mode (e.g., a night-time photo vs. a daytime photo).
- The browser selects the appropriate source automatically based on current appearance.
- Falls back to the `<img>` src for light mode (or when the feature is unsupported).

### Dark Mode in Email
- **Simple email messages** (composed in Mail on iOS 13, with only inline image attachments): Mail applies auto-darkening transformations automatically — no developer action required.
- **Rich email templates** (custom HTML/CSS with remotely loaded images, as in marketing campaigns): auto-darkening is NOT applied. The email renders in light mode by default.
- To support dark mode in marketing emails: add `color-scheme: light dark;` and use `prefers-color-scheme` media queries for custom color and image overrides — same techniques as web pages.

### Web Inspector Updates for Dark Mode Debugging **[NEW]**
- Web Inspector itself now adopts the system dark appearance when macOS is in Dark Mode.
- **Color scheme toggle** in the Elements tab lets developers switch the inspected page between light and dark appearance without changing the system setting.
- The Styles sidebar shows matching `prefers-color-scheme` media query rules for the selected element.

## APIs & Frameworks

### CSS Properties **[NEW]**
- `color-scheme: light dark;` — declares supported color schemes on `:root` **[NEW]**
- `@media (prefers-color-scheme: dark)` — media query for dark mode overrides **[NEW]**
- CSS Custom Properties (`--variable-name: value;`) — semantic color variable system
- `var(--variable-name)` — references a CSS custom property

### HTML Elements
- `<picture>` element with `<source media="(prefers-color-scheme: dark)" srcset="...">` — dark mode image loading
- `<img>` — light mode fallback within `<picture>`

### WKWebView / Safari
- Dark mode web content support in `WKWebView` (in-app web views) — same CSS APIs apply
- Safari 12.1 (macOS Mojave) — first release with `color-scheme` and `prefers-color-scheme` support
- Safari for iOS 13 — `prefers-color-scheme` support

### Web Inspector (Safari / macOS)
- Dark appearance for Web Inspector UI **[NEW]**
- Color scheme toggle in Elements tab **[NEW]**
- `prefers-color-scheme` rule display in Styles sidebar **[NEW]**

## Code Highlights

Declaring color scheme support and defining CSS variables:
```css
:root {
    color-scheme: light dark;
    --text-color: #333;
    --bg-color: #fff;
    --heading-color: #444;
}

@media (prefers-color-scheme: dark) {
    :root {
        --text-color: #eee;
        --bg-color: #1c1c1e;
        --heading-color: #f2f2f7;
    }
}

body {
    color: var(--text-color);
    background-color: var(--bg-color);
}

h1 {
    color: var(--heading-color);
}
```

Dark mode image with `<picture>`:
```html
<picture>
    <source
        srcset="desert-night.jpg"
        media="(prefers-color-scheme: dark)">
    <img src="desert-day.jpg" alt="Mojave Desert">
</picture>
```

Dark mode in a marketing email:
```html
<style>
    :root { color-scheme: light dark; }
    body { background-color: #ffffff; color: #000000; }
    @media (prefers-color-scheme: dark) {
        body { background-color: #1c1c1e; color: #ffffff; }
    }
</style>
```

## Takeaways
- Safari and WebKit never auto-darken web pages — developers must opt in using `color-scheme: light dark;` and `prefers-color-scheme` media queries.
- Use CSS custom properties to centralize color definitions; override only the variables in the dark media query block to keep styles maintainable.
- Marketing email templates must handle dark mode themselves; only simple Mail-composed messages receive automatic darkening.
- The Web Inspector color scheme toggle (new in Safari 13 on macOS) enables fast light/dark testing without toggling system-wide appearance.

---
_Source: WWDC19 Session 511 page (abstract, chapter summaries, code samples, and resource links)._
