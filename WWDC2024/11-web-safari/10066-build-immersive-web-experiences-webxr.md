# Build Immersive Web Experiences with WebXR
**WWDC24 · Session 10066** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10066/)

_Platforms:_ visionOS 2, Safari on visionOS (WebXR APIs run in Safari's web content process)

## Overview
visionOS 2 significantly expands WebXR support in Safari, enabling fully immersive VR web experiences and richer interaction models. WebXR allows web developers to build spatial experiences using standard web APIs (JavaScript, WebGL, WebGPU) that run inside Safari's browser and can request immersive sessions—replacing the browser window with a fully immersive rendering.

This session covers the new WebXR features available on visionOS 2: immersive VR sessions (replacing the existing immersive-ar mode), transient pointer input (hand-based indirect input for web content), hand tracking via the WebXR Hand Input module, and hit testing for placing content on real-world surfaces. The session also covers how to handle the session lifecycle, request the right permissions, and design web XR experiences that feel native to the visionOS platform.

## Key Topics
- **Immersive VR sessions** — `navigator.xr.requestSession('immersive-vr')` on visionOS 2; fully replaces the browser; the web app renders into the full space
- **Transient pointer input** — visionOS's gaze+pinch input model exposed to WebXR via `transient-pointer` input source type; no persistent hand position, only discrete tap events
- **Hand tracking** — `WebXR Hand Input` module; `XRInputSource.hand` returns `XRHand` joint data when user has granted permission
- **Hit testing** — `XRSession.requestHitTestSource()` for detecting intersections with real-world geometry detected by ARKit (plane detection)
- **Depth API** — `depth-sensing` feature; access to depth data for occlusion and world-understanding
- **Session lifecycle** — `xrsession.addEventListener('end')`, visibility state changes, returning to flat Safari after immersive session
- **WebGL / WebGPU** — both rendering APIs work in immersive sessions; WebGPU preferred for performance; `XRSession` binds to a WebGL context or WebGPU device
- **Permissions** — spatial tracking data requires user permission; `navigator.xr.requestSession` triggers system prompt on first use

## APIs & Frameworks
### WebXR (JavaScript — Web Standard)
- `navigator.xr` — `XRSystem`; entry point for all WebXR
- `navigator.xr.isSessionSupported('immersive-vr')` — feature detection; returns Promise<boolean>
- `navigator.xr.requestSession('immersive-vr', { requiredFeatures, optionalFeatures })` — request session; required/optional features include `'hand-tracking'`, `'hit-test'`, `'depth-sensing'`, `'local-floor'`, `'bounded-floor'`
- `XRSession` — active session; `requestAnimationFrame`, `inputSources`, `requestHitTestSource`
- **[NEW on visionOS 2] `'immersive-vr'` mode** — fully immersive; replaces Safari window entirely
- `XRInputSource` — input controller/hand; `targetRayMode` (`.transient-pointer` for gaze+pinch), `.handedness`
- **[NEW] `XRInputSource.hand`** — `XRHand` joint map; 25 joints per hand; requires `'hand-tracking'` feature
- `XRHand.get(jointName)` — returns `XRJointSpace` for a named joint (e.g., `'index-finger-tip'`)
- `XRHitTestSource` — `requestHitTestSource({ space })` for world hit testing; `.getHitTestResults(frame)` per frame
- `XRFrame.getPose(space, baseSpace)` — get pose of any tracked space relative to a reference space
- `XRReferenceSpace` — `'local'`, `'local-floor'`, `'bounded-floor'`; choose appropriate for your use case
- `XRWebGLLayer` / `XRGPUBinding` — bind WebGL context or WebGPU device to the XR session for rendering

### WebGL / WebGPU (rendering)
- `WebGLRenderingContext` / `WebGL2RenderingContext` — existing WebGL; works in XR sessions
- `GPUDevice` (WebGPU) — preferred for performance; `XRGPUBinding(session, device)` for WebGPU XR rendering

## Code Highlights
```javascript
// Check support and request immersive VR session (visionOS 2)
const supported = await navigator.xr.isSessionSupported('immersive-vr');
if (!supported) { showFallback(); return; }

const session = await navigator.xr.requestSession('immersive-vr', {
    requiredFeatures: ['local-floor'],
    optionalFeatures: ['hand-tracking', 'hit-test']
});

// Render loop
session.requestAnimationFrame(function onFrame(time, frame) {
    const pose = frame.getViewerPose(referenceSpace);
    if (!pose) return;

    // Render each eye
    for (const view of pose.views) {
        const viewport = glLayer.getViewport(view);
        gl.viewport(viewport.x, viewport.y, viewport.width, viewport.height);
        // ... WebGL draw calls
    }

    // Handle transient pointer input (gaze+pinch)
    for (const source of session.inputSources) {
        if (source.targetRayMode === 'transient-pointer') {
            const rayPose = frame.getPose(source.targetRaySpace, referenceSpace);
            // rayPose.transform = gaze ray at moment of pinch
        }
    }

    // Hand tracking
    for (const source of session.inputSources) {
        if (source.hand) {
            const indexTip = source.hand.get('index-finger-tip');
            const tipPose = frame.getJointPose(indexTip, referenceSpace);
        }
    }

    session.requestAnimationFrame(onFrame);
});

// Session end cleanup
session.addEventListener('end', () => { /* return to 2D */ });
```

## Takeaways
- visionOS 2 enables fully immersive WebXR sessions (`'immersive-vr'`) in Safari—web developers can build complete spatial experiences without a native app
- Transient pointer input (`targetRayMode === 'transient-pointer'`) is the primary interaction model on visionOS; design your input handling around discrete pinch events rather than continuous cursor position
- Hand tracking via `XRHand` unlocks gesture-based interactions; request it as an optional feature and degrade gracefully when not granted or not available
- WebGPU is the recommended rendering API for performance-sensitive WebXR content on visionOS; the unified memory architecture and TBDR GPU benefit significantly from WebGPU's explicit resource management

---
_Source: WWDC24 Session 10066 page (abstract, chapter summaries, code samples, and resource links)._
