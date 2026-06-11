# Get Started with the HTML Model Element

**WWDC26 · Session 215** · [Watch](https://developer.apple.com/videos/play/wwdc2026/215/)

_Platforms:_ Safari 27 (iOS, iPadOS, macOS, visionOS), WebKit, web standards (W3C Immersive Web)

## Overview

The HTML `<model>` element brings interactive 3D content to websites as naturally as `<img>` or `<video>`. Previously available only on visionOS, it expands in Safari 27 to iOS, iPadOS, and macOS, making it the first broadly available native 3D embedding element on the web. The element renders USDZ files — Apple's preferred 3D format — with built-in touch/gesture interaction, JavaScript-driven transforms, and hooks into AR Quick Look and visionOS immersive environments.

The session covers the full workflow: creating and optimizing USDZ assets (using tools like usdcrush and usdrecord), embedding a model with HTML, providing fallbacks for unsupported browsers, handling the `ready` promise, setting background color, enabling orbit interaction, driving custom transforms with `DOMMatrix`, playing baked USDZ animations, and linking to AR Quick Look. It also addresses production asset optimization and points to the W3C Immersive Web Community Group where the spec is being shaped.

The `<model>` element is a W3C-standardized element (Immersive Web Working Group) and a polyfill is available for non-supporting browsers.

## Key Topics

### Prepare the USDZ Model Asset (2:22)
USDZ is the recommended format because it bundles geometry, materials, textures, and animations into a single file. Sources for 3D content: iPhone scanning (Object Capture), file conversion from other formats, authoring in tools like Blender, or generating from images/text prompts using AI tools. Use `usdcrush` to compress files (often 4x smaller, no perceptible quality loss) and `usdrecord` to render thumbnail images from a USD file.

### Loading and Fallbacks (4:18)
Embed via the `src` attribute on `<model>`, or use a `<source>` child element with a `type` MIME type attribute. A nested `<img>` serves as a fallback for unsupported browsers. Await the `ready` promise to know when the model is loaded and displayable. Load the W3C polyfill (`model-element-polyfill.js`) conditionally when `window.HTMLModelElement` is absent.

### Model Background (6:14)
Set `background-color` on the `model` CSS selector to match the surrounding page. The 3D renderer composites the background as fully opaque — it does not inherit page styles.

### Interactions (6:48)
The `stagemode="orbit"` attribute enables built-in gesture interaction: users can rotate the model with automatic spring-back and clipping protection. For programmatic control, omit `stagemode` and set `model.entityTransform` (a `DOMMatrix`) directly from JavaScript to snap to specific orientations.

### Transition Animation (8:26)
Smooth orientation changes are implemented by interpolating angles inside `requestAnimationFrame`, computing a cubic ease-out, and setting `entityTransform` each frame. Cancel any in-flight animation with `cancelAnimationFrame()` before starting a new one to prevent conflicting transitions.

### Animation Playback (10:08)
USDZ files can contain baked animations. Trigger them with `model.play()`. Control playback direction and speed with `model.playbackRate` — positive values play forward, negative values reverse, and the magnitude scales speed.

### AR and Spatial (10:52)
Wrap `<model>` in an `<a rel="ar" href="file.usdz">` to enable AR Quick Look on iOS and iPadOS. On visionOS, `<model>` renders stereoscopically and can power immersive website environments using the new Immersive API (see Session 320).

### Optimize Assets for Production (12:29)
`usdcrush` — ships with macOS — compresses USDZ textures with no perceived quality loss, reducing file sizes by up to 4x. `usdrecord` renders a still image from a USD file, useful for generating fallback `<img>` thumbnails.

## APIs & Frameworks

**HTML Model Element**
- **[NEW]** `<model>` element — iOS, iPadOS, macOS support in Safari 27 (visionOS support existed previously)
- `src` attribute — URL of the USDZ file
- `<source>` child element — `src` + `type="model/vnd.usdz+zip"` MIME type
- `<img>` child element — fallback image for unsupported browsers
- `stagemode="orbit"` attribute — built-in touch/gesture orbit interaction with spring-back
- `HTMLModelElement` interface (W3C standard)

**JavaScript API on `HTMLModelElement`**
- **[NEW]** `model.ready` — Promise that resolves when the model is loaded and rendered, rejects on failure
- **[NEW]** `model.entityTransform` — `DOMMatrix` property; read initial transform and write to control orientation
- **[NEW]** `model.play()` — begin playback of baked USDZ animations
- **[NEW]** `model.playbackRate` — Number; positive = forward, negative = reverse, magnitude = speed multiplier

**CSS on `<model>`**
- `background-color` — sets the opaque background of the 3D renderer viewport
- Standard box-model sizing (width, height)

**AR Integration**
- `<a rel="ar" href="file.usdz">` wrapping `<model>` — triggers AR Quick Look on iOS/iPadOS

**Web Standards**
- `window.HTMLModelElement` — feature-detection check for native support
- W3C model-element polyfill (`model-element-polyfill.js`) — fallback for non-supporting browsers
- W3C Immersive Web Community Group spec: `https://immersive-web.github.io/model-element`

**3D Asset Tools (macOS)**
- **[NEW]** `usdcrush` — command-line tool; compresses USDZ files up to 4x with no quality loss
- `usdrecord` — command-line tool; renders a still thumbnail image from a USD/USDZ file
- USDZ format (`model/vnd.usdz+zip`) — bundles geometry, materials, textures, animations

**Immersive Environments (visionOS 27)**
- Immersive API for `<model>` — modeled on the Fullscreen API; launches model into full immersive visionOS environment (see Session 320)

**DOMMatrix**
- `new DOMMatrix()` — identity matrix
- `DOMMatrix.rotateSelf(x, y, z)` — in-place rotation; used to set `entityTransform`

**requestAnimationFrame Pattern**
- `requestAnimationFrame(step)` / `cancelAnimationFrame(id)` — animation loop for smooth `entityTransform` transitions

## Code Highlights

Basic embed with fallback:
```html
<model id="mallet" src="mallet.usdz">
  <img src="mallet.png" alt="Rubber mallet with wooden handle">
</model>
```

Ready promise and polyfill:
```js
model.ready.then(() => { /* hide spinner */ }).catch(() => { /* show fallback */ });

if (!window.HTMLModelElement) {
  import("model-element-polyfill.js");
}
```

Orbit interaction:
```html
<model src="product.usdz" stagemode="orbit"></model>
```

Custom entityTransform:
```js
const transform = new DOMMatrix().rotateSelf(0, 135, 0);
model.entityTransform = transform;
```

Animation playback:
```js
model.playbackRate = -5; // reverse at 5x speed
model.play();
```

AR Quick Look:
```html
<a rel="ar" href="bottle.usdz">
  <model src="bottle.usdz"></model>
</a>
```

## Takeaways

- `<model>` expands from visionOS-only to all Apple platforms in Safari 27 — a single HTML element embeds interactive 3D content as simply as `<img>`.
- The `ready` promise, `entityTransform` / `DOMMatrix`, `play()`, and `playbackRate` form the complete JavaScript API for loading state, orientation control, and animation playback.
- Use `usdcrush` before deploying — it routinely reduces USDZ file sizes by 4x with no perceptible quality loss.
- The polyfill and `<img>` fallback patterns ensure graceful degradation on all browsers; `<a rel="ar">` wrapping triggers AR Quick Look on iOS/iPadOS at no extra cost.

---
_Source: WWDC26 Session 215 page (abstract, chapter summaries, code samples, and resource links)._
