# Meet Safari for Spatial Computing
**WWDC23 · Session 10279** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10279/)

_Platforms:_ visionOS 1

## Overview
Safari on visionOS is the full Safari browser powered by the same WebKit engine as iPad and Mac, running on a brand-new spatial computing platform. Existing websites work out of the box thanks to comprehensive web standards support. The session covers the platform's unique eye-and-hand input model, the interactive regions system that provides visual feedback, and practical optimizations for animations and layouts. It also introduces two emerging 3D web standards—the HTML `<model>` element and WebXR—that unlock immersive experiences directly in the browser.

Existing responsive-design best practices (CSS viewport units, media queries, SVG graphics, `devicePixelRatio`) transfer directly to visionOS. The platform's concept of "screen" differs slightly: when a page enters full screen, the window is resized to a default size that is also reported as screen dimensions to JavaScript, but windows can be resized beyond those dimensions.

Safari extensions and developer tools (Web Inspector) work on visionOS as they do on other platforms, and the visionOS Simulator lets developers test interactive regions without needing a physical device.

## Key Topics

### Input Model: Eye + Hand Gestures
- Primary input: look (eyes) + tap (pinch fingers together) — indirect gestures
- Secondary input: reach out and physically touch the page — direct touch
- **Indirect gesture**: `pointerdown` dispatched at eye-fixation point when pinch begins; `pointermove` tracks hand motion; `pointerup` on release
- **Direct touch**: `pointerdown`/`pointermove`/`pointerup` based on hand/finger position as it intersects the window
- `click` events fire normally; scroll and scroll snapping work as expected
- Media query: primary input is treated as coarse pointer (like touchscreen), no hover support; Bluetooth trackpad/keyboard may be connected

### Interactive Regions
- visionOS provides visual highlight feedback when the user looks at interactive elements — no hover events from JavaScript/CSS needed
- Highlights generated automatically by WebKit based on accessible markup and CSS
- Automatically highlighted: `<a>`, `<button>`, `<input>`, `<select>`, elements with matching ARIA roles
- Custom interactive elements: add `cursor: pointer` CSS to opt in to highlighting
- Set `pointer-events: none` on inner decorative child elements to prevent them getting their own separate highlight regions
- Shape the highlight with `border-radius` — WebKit matches the element's border-radius for the highlight
- visionOS Simulator: move mouse to simulate gaze, click to simulate tap — useful for debugging highlights

### Layout and Responsive Design Best Practices
- Use CSS viewport units for layouts; respond to `resize` with media queries and container queries
- Prefer SVG for UI elements — scales cleanly at any window size or proximity
- Use `devicePixelRatio` and responsive images (`srcset`) for appropriate bitmap resolution
- Full-screen windows report a default size to JavaScript as `screen` dimensions; windows can be resized beyond that — account for this in full-screen layouts

### Animation and Scrolling Optimization
- Animations target the best available frame rate; use `requestAnimationFrame` + elapsed time measurement — never assume a fixed frame rate
- Passive scroll event listeners prevent scroll performance degradation
- Web Inspector Timelines for animation/scroll performance diagnosis
- Correct pattern: measure `timestamp` delta in `requestAnimationFrame` callback to compute animation progress independent of frame rate

### AR Quick Look on visionOS
- Same mechanism as iOS AR Quick Look: `<a rel="ar" href="model.usdz"><img src="preview.jpg"></a>`
- Places USDZ models in the user's space with advanced RealityKit lighting and rendering
- Requires USDZ files; Reality Converter and USDZ Tools available for asset conversion

### HTML `<model>` Element (Emerging Standard)
- Proposed W3C standard **[NEW / preview]** — 3D equivalent of `<img>` embedded inline in the page
- Best rendering on every device including full stereoscopic display with environmental lighting on visionOS
- Attributes: `src` for the 3D model source, interactive/non-interactive toggle
- JavaScript API: access to camera, animations, and more
- Enable via feature flag in Safari settings on any platform

### WebXR (Developer Preview)
- W3C standard for fully immersive 3D scenes in the browser **[developer preview on visionOS]**
- Built on WebGL; popular WebGL libraries have built-in WebXR support
- Request a WebXR session to enter immersive mode — full spatial environment, not just a page
- Enable via feature flag in Safari Advanced Settings on visionOS

## APIs & Frameworks

- **WebKit** – browser engine powering Safari on visionOS; same engine as iOS/macOS
- **Interactive Regions** **[NEW on visionOS]** – automatic gaze-based highlight system driven by accessible markup and CSS
- `cursor: pointer` CSS property – opt-in signal for custom interactive elements to receive highlight
- `pointer-events: none` CSS – exclude child elements from generating separate highlight regions
- `border-radius` CSS – shapes the interactive region highlight border
- Pointer Events API – `pointerdown`, `pointermove`, `pointerup` – handles both indirect and direct input
- `click` event – still dispatched normally for simple tap interactions
- `devicePixelRatio` – reflects recommended resolution for canvas/image loading on visionOS
- `requestAnimationFrame` – animation loop; must measure elapsed time (`timestamp` parameter)
- Passive scroll event listeners – `addEventListener('scroll', handler, { passive: true })`
- CSS viewport units (`vw`, `vh`, `dvw`, `dvh`) – recommended for layout on variable-size windows
- Media queries + container queries – adapt to window resize
- SVG – preferred graphics format for UI elements on visionOS
- **AR Quick Look** – `<a rel="ar" href="model.usdz"><img>` – places USDZ in user's space
- **HTML `<model>` element** **[NEW / proposed W3C standard]** – inline 3D model display
- **WebXR** **[developer preview]** – fully immersive web experiences on visionOS; based on WebGL
- Web Inspector – Safari developer tools; available on visionOS; `cursor` property inspection
- bugs.webkit.org – WebKit bug tracker for HTML/JS/CSS issues
- Reality Converter / USDZ Tools – asset conversion to USDZ for AR Quick Look

## Code Highlights

AR Quick Look anchor (same as iOS):
```html
<a rel="ar" href="teapot.usdz">
  <img src="teapot-preview.jpg" alt="Teapot">
</a>
```

CSS for custom interactive element highlight:
```css
.list-item {
  cursor: pointer;
  border-radius: 8px; /* matched by WebKit for the highlight shape */
}
.list-item .icon,
.list-item .label {
  pointer-events: none; /* prevent separate highlight regions on children */
}
```

Frame-rate-independent animation using `requestAnimationFrame`:
```javascript
let lastTimestamp = null;
function animate(timestamp) {
  if (lastTimestamp !== null) {
    const elapsed = timestamp - lastTimestamp;
    // Use elapsed to compute progress, not a fixed increment
    progress += elapsed / duration;
  }
  lastTimestamp = timestamp;
  requestAnimationFrame(animate);
}
requestAnimationFrame(animate);
```

## Takeaways
- All existing responsive websites work on visionOS out of the box; apply the same progressive-enhancement techniques already used for iPhone, iPad, and Mac.
- Interactive regions replace hover states on visionOS: use `cursor: pointer` for custom interactive elements and `pointer-events: none` on decorative children to get clean, correctly-shaped highlights.
- Always use elapsed-time-based animation in `requestAnimationFrame` — do not assume a fixed frame rate, because visionOS targets the optimal rate for the current content.
- AR Quick Look works identically to iOS and is the fastest path to placing 3D objects in the user's space; the emerging HTML `<model>` element and WebXR are the next steps toward a fully immersive web.

---
_Source: WWDC23 Session 10279 page (abstract, chapter summaries, code samples, and resource links)._
