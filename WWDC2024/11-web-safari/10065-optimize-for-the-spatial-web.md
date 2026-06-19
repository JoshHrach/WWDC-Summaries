# Optimize for the Spatial Web
**WWDC24 · Session 10065** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10065/)

_Platforms:_ visionOS 2, Safari

## Overview
Safari in visionOS 2 expands the set of web platform capabilities available to spatial computing. This session, delivered by a Safari team engineer, demonstrates how standard web technologies — interaction regions, Web Speech API, Web Audio API, element fullscreen, Quick Look, and WebXR — map onto visionOS's natural input model and immersive environment.

All features covered are either existing web standards that gain new significance in visionOS, or visionOS 2-specific enhancements to how Safari applies those standards. No proprietary APIs are required: spatial web experiences are authored with the same HTML, CSS, and JavaScript that run on Mac, iPhone, and iPad.

## Key Topics

**Interaction Regions (Natural Input)**
- visionOS Natural Input uses eyes + hands: the user looks at an element to target it, then pinches to activate it; highlights appear where the user's gaze dwells, but eye-tracking data stays private (handled outside Safari)
- Best practices for interaction regions: add padding to links/buttons so the target area is larger; use `border-radius` to round highlight shapes to match site aesthetics — these hints work even without a visible background
- New in visionOS 2: SVG path-based interaction regions — the same SVG used for a `hover` effect can define the exact highlight shape, enabling complex non-rectangular interactive areas
- Media content interaction regions automatically fade the highlight after a few seconds so large image/video thumbnails are not persistently overlaid; just wrap the media inside the `<a>` tag

**Voice Input — Web Speech API**
- `SpeechRecognition` (prefixed `webkitSpeechRecognition` on Safari) — listen for voice input as a separate, continuous input channel; ideal for hands-free interactions like games
- `SpeechSynthesis` + `SpeechSynthesisUtterance` — produce spoken feedback from the page
- All speech processing happens locally on-device; no data leaves the device, no API keys required
- Must be initiated from a user event; a microphone permission prompt is shown once

**Spatial and Panorama Photos (Element Fullscreen API)**
- Spatial and panoramic photos displayed inline on a page show flat/monoscopic; the correct immersive view requires centering and exclusive-mode rendering
- Use the standard `requestFullscreen()` method on the image element (triggered by a user click) to launch the Photos-style fullscreen view: a feathered portal in the window, then an immersive button for full-scale exclusive display
- `document.exitFullscreen()` exits programmatically; the home gesture or crown button also exits
- Use `<picture>` / `srcset` to serve the spatial image file format with a fallback for other browsers
- Same pattern works for panorama images; any image with large enough dimensions is treated as a panorama

**3D Models — Quick Look**
- Link to a `.usdz` file with `rel="ar"` on the `<a>` tag to let visitors open the model in Quick Look on visionOS (same syntax as iOS AR Quick Look)
- Visitors can place and scale the 3D object in their real environment; the website stays open in Safari alongside Quick Look
- Check `anchorElement.relList.supports('ar')` to gate the link UI to browsers that support Quick Look

**Spatial Audio — Web Audio API**
- `AudioContext` — create an audio processing graph of nodes
- `PannerNode` — position audio sources in 3D space; configure `position`, `orientation`, `coneAngle`, and `refDistance` to create convincing spatial soundscapes
- Connect `PannerNode` outputs back to `AudioContext.destination`; combine multiple sources and effects to build dynamic environments

**Immersive VR — WebXR**
- W3C standard for cross-platform immersive experiences running inside Safari with no downloads required
- Deep-dive content deferred to the companion session "Build Immersive Web Experiences with WebXR"

**Inspect and Debug**
- Connect Apple Vision Pro to Web Inspector on Mac: same network, enable Web Inspector on device (`Apps > Safari > Advanced > Web Inspector`), set `Settings > General > Remote Devices` on Vision Pro, then `Develop` menu in Safari on Mac → `Use for Development`; one-time pairing with a six-digit code
- Inspect DOM, CSS rules, JavaScript console, and apply CORS exemptions for debugging; Web Inspector can run in the virtual Mac display alongside the spatial content

## APIs & Frameworks

**Web APIs (standard; Safari/WebKit)**
- `SpeechRecognition` / `webkitSpeechRecognition` — real-time speech-to-text via Web Speech API
- `SpeechSynthesis` — text-to-speech output via Web Speech API
- `SpeechSynthesisUtterance` — wraps the string to be spoken
- `Element.requestFullscreen()` — launch element into exclusive fullscreen; triggers visionOS immersive mode for spatial/panorama images
- `document.exitFullscreen()` — exit fullscreen programmatically
- `rel="ar"` on `<a>` linking a `.usdz` file — opens model in Quick Look on visionOS/iOS
- `HTMLAnchorElement.relList.supports('ar')` — feature-detect Quick Look support
- `AudioContext` — root of the Web Audio processing graph
- `PannerNode` — spatialize audio sources in 3D (`position`, `orientation`, `coneAngle`, `refDistance`)
- SVG path-based interaction regions **[NEW in visionOS 2]** — use existing SVG `<path>` elements to define custom highlight shapes for Natural Input
- Media interaction region fade behavior **[NEW in visionOS 2]** — highlights on media-containing links automatically fade after dwell

**WebXR Device API**
- WebXR — W3C standard for immersive VR/AR in Safari; covered in depth in companion session

**Safari Web Inspector**
- Remote inspection of visionOS Safari via Mac Web Inspector — pair over same network with six-digit code

## Code Highlights

Web Speech recognition loop:
```javascript
const recognition = new webkitSpeechRecognition();
recognition.onresult = (event) => {
    const result = event.resultIndex;
    const transcript = event.results[result][0].transcript;
    // match against expected color names
};
recognition.start(); // must be called from a user event
```

Text-to-speech feedback:
```javascript
const utterance = new SpeechSynthesisUtterance("Your score is 42!");
speechSynthesis.speak(utterance);
```

Spatial photo fullscreen:
```html
<picture>
  <source srcset="photo.heic" type="image/heic">
  <img src="photo.jpg" id="spatialPhoto">
</picture>
<script>
document.getElementById('spatialPhoto').addEventListener('click', (e) => {
    e.target.requestFullscreen().catch(console.error);
});
</script>
```

Quick Look link:
```html
<a href="model.usdz" rel="ar">View in Quick Look</a>
```

## Takeaways
- Add padding and `border-radius` to interactive elements — these CSS properties shape the Natural Input highlight regions in visionOS with no extra code.
- Use `element.requestFullscreen()` (triggered by a user tap) to present spatial and panorama photos in visionOS's exclusive immersive view — no special APIs, no app download required.
- Link USDZ models with `rel="ar"` to give visitors a one-tap Quick Look experience; gate the UI with `relList.supports('ar')` to avoid showing the link on unsupported browsers.
- The Web Speech and Web Audio APIs both run fully on-device in Safari — no keys or servers needed, and they pair naturally with WebXR for fully immersive spatial experiences.

---
_Source: WWDC24 Session 10065 page (abstract, chapter list, and full transcript)._
