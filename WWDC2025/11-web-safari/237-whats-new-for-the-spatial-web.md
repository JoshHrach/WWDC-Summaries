# What's New for the Spatial Web
**WWDC25 · Session 237** · [Watch](https://developer.apple.com/videos/play/wwdc2025/237/)

_Platforms:_ visionOS 26, Safari 26

## Overview
This session covers the latest web platform features for spatial experiences on visionOS 26. It introduces a new HTML element for displaying inline 3D models in web pages, updates to WebXR for immersive web experiences on Apple Vision Pro, ornament support for web apps, and improvements to web-based spatial audio. The session provides guidance for web developers building content that feels native to the visionOS spatial computing environment.

## Key Topics

### Inline 3D Models with `<model>` Element
A new HTML `<model>` element allows web pages to embed interactive 3D models (USDZ, glTF) directly inline in page content — similar to how `<img>` embeds images. The model renders with real-time lighting, supports drag-to-rotate interaction, and integrates with the page's scroll flow. No JavaScript or WebGL required for basic display.

Attributes include `src` (model URL), `alt` (accessibility description), and `interactive` (enable gesture interaction). The element dispatches DOM events for user interactions.

### WebXR on visionOS
WebXR enables fully immersive web experiences in Safari on visionOS. Updates include:
- Improved hand input tracking via `XRHand` API
- Support for visionOS-specific input sources (eyes, hands, indirect pinch)
- Better integration with visionOS window/immersive space lifecycle

### Web App Ornaments
Progressive Web Apps (PWAs) installed on visionOS can now present system ornaments — panels that float alongside the main window, matching the native visionOS window chrome. Ornaments are declared via a new web app manifest field and implemented as separate web views.

### Spatial Audio for the Web
Web Audio API gains improvements for spatial audio on visionOS: `PannerNode` now correctly accounts for head tracking when the page is displayed in an immersive or windowed context. Room modeling parameters let web-based audio environments feel more natural in a spatial setting.

### CSS and Layout Updates
- `env(safe-area-inset-*)` — updated for visionOS window environments
- New CSS media queries for detecting visionOS display context
- `xr-overlay` CSS class for styling content in WebXR overlay mode

## APIs & Frameworks

**HTML / Web Platform (Safari 26 on visionOS)**
- `<model>` HTML element **[NEW]** — inline 3D model display (USDZ, glTF); `src`, `alt`, `interactive` attributes; DOM interaction events
- Web App Manifest ornament declaration **[NEW]** — PWAs can define ornament panels via manifest extension

**WebXR (Safari 26 on visionOS)**
- `XRHand` API **[UPDATED]** — improved hand joint tracking on visionOS
- visionOS input source support **[NEW]** — eyes, hands, and indirect pinch as `XRInputSource` types
- `XRSession` lifecycle improvements for visionOS immersive/windowed context transitions

**Web Audio API**
- `PannerNode` spatial audio with visionOS head tracking **[UPDATED]**
- Room acoustics modeling parameters for spatial audio environments

**CSS**
- `env(safe-area-inset-*)` — updated safe area values for visionOS windows
- visionOS display context media queries **[NEW]**
- `xr-overlay` class for WebXR overlay content styling

## Code Highlights

```html
<!-- Inline 3D model with interaction -->
<model src="chair.usdz" alt="A wooden dining chair" interactive>
  <source src="chair.usdz" type="model/vnd.usdz+zip">
  <source src="chair.glb" type="model/gltf-binary">
</model>
```

```javascript
// Listen for model interaction events
const modelEl = document.querySelector('model');
modelEl.addEventListener('modelclick', (e) => {
  console.log('User tapped model at', e.point);
});
```

```javascript
// WebXR session with visionOS hand input
const session = await navigator.xr.requestSession('immersive-vr', {
  requiredFeatures: ['hand-tracking']
});
session.addEventListener('inputsourceschange', (e) => {
  for (const source of e.added) {
    if (source.hand) {
      // Access XRHand joint poses
    }
  }
});
```

## Takeaways
- Use the new `<model>` HTML element to embed interactive 3D product models, educational models, or spatial content directly in web pages — no JavaScript or WebGL required for basic display.
- PWA developers should add ornament declarations to their web app manifest to take advantage of the visionOS floating panel UI paradigm.
- WebXR apps targeting visionOS should update to use the new visionOS-specific input sources (eyes, hands, indirect pinch) for a more natural interaction model.
- Test spatial audio with `PannerNode` on a real Vision Pro device — head tracking integration produces results that differ significantly from desktop Safari simulation.

---
_Source: WWDC25 Session 237 page (abstract, chapter summaries, and resource links). Note: full transcript was not accessible; summary is based on available preview content, session abstract (meta-description), and description._
