# Design for Safari 15
**WWDC21 · Session 10029** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10029/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12

## Overview
Safari 15 represents a fundamental redesign of the browser experience, removing the traditional tall toolbar frame and letting web content extend to all four edges of the window. The tab bar now blends into each website by adopting a color derived from or specified by the page itself, and on iOS the tab bar moves to the bottom of the screen for easier one-handed reach.

This session covers the new `theme-color` meta tag behavior, CSS environment variables for safe area insets, two major new CSS features (`aspect-ratio` and flex `gap`), redesigned iOS/iPadOS form controls, Live Text integration in images, and how to craft rich links and web app Home screen icons. Throughout, both design and implementation details are shown via a demo app (Dog Dog DC).

## Key Topics

### Theme Color and Tab Bar Blending
- Safari 15 automatically picks a tab bar color from the page background or header; override with `<meta name="theme-color" content="…">` in the HTML `<head>`.
- Supports separate Light and Dark Mode values by adding `media="(prefers-color-scheme: light|dark)"` attributes to meta tags—a newly proposed HTML specification addition.
- Theme color can be set per page (each HTML file has its own `<head>`) or changed dynamically via JavaScript by modifying the meta element's `content` attribute.
- Very extreme colors (near-black, near-white) may be overridden by Safari to preserve usability.
- Theme color can also be declared in a Web App Manifest file; the `<head>` meta tag takes precedence when both are present.

### iOS Tab Bar at the Bottom and Safe Area Insets
- On iOS 15, the tab bar moves to the bottom; it minimizes to just the domain name while scrolling.
- Content that would be obscured by the tab bar must be adjusted using CSS environment variables.
- Four variables: `env(safe-area-inset-top)`, `env(safe-area-inset-right)`, `env(safe-area-inset-bottom)`, `env(safe-area-inset-left)`.
- Usable anywhere a length is accepted in CSS, including inside `calc()`.
- Values update automatically as the tab bar expands/collapses; Safari re-lays out the page.
- Landscape iPhone: use `viewport-fit=cover` in the `<meta viewport>` to extend content edge-to-edge, then apply insets manually via env variables.

### Tab Groups and Favicons
- Tab Groups let users organize and sync tabs across macOS, iOS, and iPadOS.
- With many tabs open, the favicon becomes the primary identifier; provide a high-resolution favicon with a transparent background.
- Page `<title>` tag: put the content title first (not the site name) for easier tab identification when multiple pages from the same site are open.

### Home Screen Web Apps
- Saving to Home screen creates an app icon using the site's specified icon; theme color fills the top of the screen just as in Safari.
- No tab bar appears; the experience feels native.

### Rich Links and Shared with You
- Rich links appear across Messages, Mail, and the new "Shared with You" feature.
- Improve with Open Graph meta tags: `og:title`, `og:image`, `og:video`.
- Videos in rich links autoplay silently; the user taps to unmute.

### Live Text (Visual Intelligence) in Images
- Safari 15 on macOS Monterey and iOS/iPadOS 15 uses on-device machine learning to detect and expose text inside images.
- Detected text is injected into the Shadow DOM within the image element; JavaScript cannot access it, but z-order can block it (overlaying elements prevent detection).
- Users can select, copy, look up, translate, search, share, and listen to detected text.
- Integrated with VoiceOver; does not replace the need for proper `alt` attributes on images.

### CSS: `aspect-ratio` Property **[NEW]**
- Sets a preferred aspect ratio on any element: `aspect-ratio: 16 / 9`, `aspect-ratio: 1`, `aspect-ratio: 2.35`.
- Elements grow beyond the ratio if content overflows by default; add `min-height: 0` or `overflow` to enforce the ratio strictly.
- Solves the long-standing "padding hack" needed for responsive embedded `<iframe>` video.
- Combines powerfully with CSS Grid for fixed-ratio grid cells.

### CSS: Flex `gap` **[NEW in Safari 14.1]**
- `gap` property on Flexbox containers guarantees minimum spacing between flex items without per-item margins.
- Landed in Safari 14.1 (spring 2021).

### Other CSS Updates
- Logical properties (for international layout support)
- Individual transform properties: `rotate`, `translate`, `scale` **[NEW]**
- `scroll-margin` and `scroll-snap` improvements
- New color space support
- Animation of pseudo-elements and discrete properties

### Form Controls
- Date and Time inputs (`<input type="date">`, `<input type="time">`) now show Calendar and Time picker controls on macOS in Safari 14.1.
- iOS and iPadOS 15: completely redesigned form controls matching UIKit aesthetics, with Dark Mode support.
- New color picker for `<input type="color">` on iOS/iPadOS with three-panel UI and eyedropper **[NEW]**.

## APIs & Frameworks

**HTML / Meta Tags**
- `<meta name="theme-color" content="…">` — specifies the browser chrome color **[existing, expanded]**
- `<meta name="theme-color" content="…" media="(prefers-color-scheme: dark)">` — Light/Dark Mode per-theme color **[NEW]**
- Open Graph `og:title`, `og:image`, `og:video` — rich link metadata **[existing]**
- Web App Manifest `theme_color` — alternate theme color declaration **[existing]**

**CSS**
- `aspect-ratio: <ratio>` property **[NEW]**
- `gap` in Flexbox containers **[NEW in Safari 14.1]**
- `env(safe-area-inset-top | right | bottom | left)` — CSS environment variables for safe areas **[existing, now critical for bottom tab bar]**
- `viewport-fit=cover` in `<meta viewport>` — extends content edge-to-edge in landscape **[existing]**
- `rotate`, `translate`, `scale` individual transform properties **[NEW]**
- Logical properties (e.g., `margin-inline-start`) **[expanded support]**
- `scroll-margin`, `scroll-snap` improvements **[expanded]**

**HTML Form Inputs**
- `<input type="date">` / `<input type="time">` — native picker on macOS **[NEW in Safari 14.1]**
- `<input type="color">` — new three-panel color picker with eyedropper on iOS/iPadOS **[NEW]**

**Shadow DOM**
- Live Text injects detected image text into the Shadow DOM of image elements (read-only, not JavaScript-accessible)

## Code Highlights

Setting theme color with Light and Dark Mode support:
```html
<meta name="theme-color" content="#ffffff" media="(prefers-color-scheme: light)">
<meta name="theme-color" content="#1a1a1a" media="(prefers-color-scheme: dark)">
```

Dynamically updating theme color via JavaScript:
```javascript
const meta = document.querySelector('meta[name="theme-color"]');
meta.content = '#0a84ff'; // e.g., when entering Theater Mode
```

Safe area inset for bottom content:
```css
footer {
    padding-bottom: calc(1rem + env(safe-area-inset-bottom));
}
```

Responsive embedded video with `aspect-ratio`:
```css
iframe {
    width: 100%;
    aspect-ratio: 16 / 9;
}
```

## Takeaways
- Set `<meta name="theme-color">` with Light and Dark Mode variants so Safari 15's tab bar blends seamlessly with your site's design.
- Use `env(safe-area-inset-bottom)` in padding, margin, or grid tracks to ensure content is not obscured by the bottom tab bar on iOS 15.
- The new CSS `aspect-ratio` property finally provides a clean, one-line solution for responsive embedded iframes and consistent grid cells.
- Always provide descriptive Open Graph metadata and a high-resolution favicon—rich links and tab groups make these more visible than ever.

---
_Source: WWDC21 Session 10029 page (abstract, full transcript, and resource links)._
