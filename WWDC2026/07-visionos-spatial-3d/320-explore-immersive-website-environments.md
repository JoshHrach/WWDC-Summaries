# Explore immersive website environments
**WWDC26 · Session 320** · [Watch](https://developer.apple.com/videos/play/wwdc2026/320/)

_Platforms:_ visionOS 27 (Safari), macOS 27 (Safari)

## Overview
This session introduces the JavaScript Immersive API in Safari for visionOS, which allows websites to transport visitors into a full 3D environment with just a few lines of code. Unlike the Fullscreen API, the Immersive API opens a spatial environment around the existing web page rather than replacing it — the page remains visible and interactive inside the environment.

The session uses two demo sites: a theater ticket sales experience (inline model preview → transition to sit inside the theater) and an escape-room marketing site (direct immersive launch with video docking and animated model). Both are built with standard HTML, CSS, and JavaScript using the new `<model>` element and the `requestImmersive()` API.

The session also covers optimization: RealityKit annotations via Reality Composer Pro or a Blender plugin, `usdcrush` for asset compression, vertex/entity count budgets, and using the Scene Understanding component to cast Safari's window shadow onto scene geometry. A brief final section shows how to add immersive controls to `<img>` elements for spatial photos.

## Key Topics

### The HTML Model Element
- `<model src="file.usdz" environmentmap="lighting.hdr">` renders an interactive 3D USDZ model inline.
- `model.entityTransform = new DOMMatrix()` positions the model in 3D space.
- `model.ready` promise resolves when the asset is loaded; always await before setting transforms.
- `model.play()` triggers embedded USD animations.

### The Immersive API
- `document.immersiveEnabled` — boolean; test before showing "Go Immersive" affordance.
- `model.requestImmersive()` — async; transitions the `<model>` into a full immersive environment surrounding the webpage.
- `:immersive` CSS pseudo-class — applied to the document when immersive; use for layout/visibility changes.
- `document.immersiveElement` — reference to the currently immersive model element.
- `model.addEventListener("immersivechange", ...)` — fires when entering or exiting immersive mode.
- `document.exitImmersive()` — programmatic exit.
- Unlike WebXR/Fullscreen, the existing page layout remains usable during immersion.

### Inline Preview + Immersive Transition
- Add a `<model>` to the page, apply a `DOMMatrix` transform to position it from the user's perspective (e.g., translate down to eye level).
- Change the transform when entering immersive mode to match the full-environment coordinate system.
- Coordinate system differs between inline (model-relative) and immersive (world-scale) modes.
- `DOMMatrix.translateSelf(x, y, z)` and `DOMMatrix.rotateSelf(x, y, z)` for positioning.

### Video Docking
- `video.requestFullscreen()` — docks a playing `<video>` element inside the immersive environment as a 3D screen.
- Exit with `document.exitFullscreen()`.
- Combine with `model.play()` to trigger model animations after video ends.

### Optimization
- **RealityKit annotations**: author contextual labels for 3D objects in Reality Composer Pro or the Blender add-on; display automatically in the immersive environment.
- **Scene Understanding component**: add to scene geometry in RCP3 to enable Safari's window shadow to cast onto the model.
- **Asset compression**: `usdcrush model.usdz -o optimized.usdz` — mesh and texture compression (see Session 285).
- **Polygon budget**: keep vertex counts low; reduce the number of entities for smooth loading.
- **Image controls**: add `controls` attribute to `<img>` elements showing spatial photos to get an immersive viewing affordance automatically.

## APIs & Frameworks

### JavaScript Immersive API (NEW — Safari / visionOS 27)
- `document.immersiveEnabled: Boolean` **[NEW]**
- `model.requestImmersive(): Promise<void>` **[NEW]**
- `document.immersiveElement: HTMLModelElement | null` **[NEW]**
- `:immersive` CSS pseudo-class **[NEW]**
- `"immersivechange"` event on `HTMLModelElement` **[NEW]**
- `document.exitImmersive()` **[NEW]** (implied by API)

### HTML Model Element (WebKit)
- `<model src="..." environmentmap="...">` **[NEW/updated]**
- `model.entityTransform: DOMMatrix` **[NEW]**: 3D transform applied to model content
- `model.ready: Promise` **[NEW]**: resolves when asset loaded
- `model.play()` **[NEW]**: trigger USD embedded animations
- `DOMMatrix` — existing Web API; `translateSelf(x,y,z)`, `rotateSelf(rx,ry,rz)` methods

### HTML / CSS (standard Web APIs — existing)
- `video.requestFullscreen()` — existing Fullscreen API; docks video in immersive environment on visionOS
- `document.exitFullscreen()`
- `<img controls>` attribute **[NEW]**: immersive viewing for spatial photos

### WebKit / Reality Composer Pro Integration
- RealityKit annotations — authored in RCP3 or Blender add-on; rendered in Safari immersive environments
- Scene Understanding component — enables window shadow casting on model geometry

### Command Line Tools
- `usdcrush model.usdz -o optimized.usdz` — mesh + texture compression

### Resources
- [Download — Immersive model add-on for Blender](https://developer.apple.com/download/files/web-env-blender-plugin.zip)
- [WebKit.org — Theater Ticket Sales demo](https://webkit.org/demos/model-demos/ticket-sales.html)
- [WebKit.org — Escape Game demo](https://webkit.org/demos/model-demos/escape-room.html)
- [GitHub: Spatial Backdrop explainer](https://github.com/WebKit/explainers/tree/main/spatial-backdrop)

## Code Highlights

Detect support and go immersive:
```javascript
if (document.immersiveEnabled) {
    immersiveButton.hidden = false;
}
immersiveButton.addEventListener("click", async () => {
    await model.requestImmersive();
});
```

Update transform and layout on immersive change:
```javascript
theater.addEventListener("immersivechange", () => {
    const isImmersive = !!document.immersiveElement;
    theater.entityTransform = buildTransform(isImmersive, currentSeat);
    document.body.classList.toggle("immersive", isImmersive);
});
```

Dock a video inside the environment, then play model animation:
```javascript
await trailerVideo.requestFullscreen();
trailerVideo.addEventListener("ended", async () => {
    await document.exitFullscreen();
    escapeRoom.play();
});
```

## Takeaways
- The Immersive API makes spatial environments a first-class web feature: any website can add a full 360° environment with a `<model>` element and a single JavaScript call.
- The unique design — the web page persists inside the environment rather than being hidden — preserves the web experience model while adding spatial depth.
- Video docking and model animations enable rich narrative arcs (watch a trailer, then enter the scene) entirely in browser JavaScript with no native app code.
- Asset optimization (`usdcrush`, polygon budgets, annotation limits) is critical for web delivery; the same tools used for native visionOS apps apply here.

---
_Source: WWDC26 Session 320 page (abstract, chapter summaries, code samples, and resource links)._
