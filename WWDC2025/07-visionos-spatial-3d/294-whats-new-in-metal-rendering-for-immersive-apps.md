# What's new in Metal rendering for immersive apps
**WWDC25 · Session 294** · [Watch](https://developer.apple.com/videos/play/wwdc2025/294/)

_Platforms:_ visionOS 26, macOS Tahoe 26

## Overview
This session covers four major additions to Metal rendering for immersive apps built with Compositor Services on Apple Vision Pro. The presentation builds on "Discover Metal for immersive apps" (WWDC23) and is aimed at developers already familiar with the Compositor Services framework and Metal rendering.

The headline features are: hover effects on interactive 3D scene objects using a new tracking areas texture; dynamic render quality that adjusts foveated rendering resolution at runtime; a new progressive immersion style letting users dial their immersion level via the Digital Crown; and macOS spatial rendering, which allows a Mac app to stream Metal-rendered immersive content directly to Vision Pro.

## Key Topics

### New render loop APIs
The existing `queryDrawable()` is replaced by **`queryDrawables()`**, returning an array. Normally this contains one drawable; when a high-quality video is being captured with Reality Composer Pro, a second drawable with `.capture` target appears alongside the primary `.builtIn` drawable. The Xcode visionOS template now offers both Metal and Metal 4 options for new projects.

### Hover effects
Apps can highlight interactive 3D objects as the user looks at them. The drawable now exposes a **tracking areas texture** (8-bit, up to 255 concurrent interactive objects). Each interactive object gets a `TrackingArea` with a unique identifier; calling `trackingArea.addHoverEffect(.automatic)` tells the system to highlight it automatically. The fragment shader writes the tracking area render value to a second color attachment. A `SpatialEventCollection.Event` now includes a nullable `trackingAreaIdentifier` that maps directly to scene objects, simplifying hit-testing. MSAA requires a custom tile resolver because normal multisample resolve would average identifiers.

### Dynamic render quality
Apps can now set a **maximum render quality** in `LayerRenderer.Configuration` and adjust the **runtime render quality** per-frame via `layerRenderer.renderQuality`. Foveation must be enabled. Higher quality expands the high-density area of the foveated texture, improving text and UI clarity at the cost of more memory and power. Instruments and the Metal debugger are the recommended tools for finding the right balance.

### Progressive immersion
A new `.progressive` immersion style lets users rotate the Digital Crown to smoothly adjust their immersion level from 0% to 100%. The app masks rendering using a **portal stencil**: `drawable.addRenderContext(commandBuffer:)` returns a `DrawableRenderContext`; calling `drawableRenderContext.drawMaskOnStencilAttachment(commandEncoder:value:)` writes the stencil mask; pixels outside the portal are skipped; the system then applies the fading edge effect. Only works with the `.layered` layout.

### macOS spatial rendering
Mac apps can open a **`RemoteImmersiveSpace`** scene (new scene type) and stream Metal-rendered immersive content to a paired Vision Pro. ARKit and `WorldTrackingProvider` are now available on macOS, connected to Vision Pro's sensors via a `remoteDeviceIdentifier` SwiftUI environment object passed to `ARKitSession(device:)`. Mac apps support mouse/keyboard/gamepad input, and spatial pinch events via `onSpatialEvent`. All APIs have C equivalents (Compositor Services `cp_` prefix, ARKit `ar_` types).

## APIs & Frameworks

### Compositor Services (CompositorServices)
- **`LayerRenderer.Drawable.queryDrawables()`** **[NEW]** — returns `[Drawable]` (replaces single `queryDrawable()`)
- **`Drawable.target`** **[NEW]** — `.builtIn` or `.capture`
- **`LayerRenderer.Configuration.trackingAreasFormat`** **[NEW]** — `MTLPixelFormat` for the tracking areas texture (`.r8Uint`)
- **`LayerRenderer.Capabilities.supportedTrackingAreasFormats`** **[NEW]**
- **`Drawable.addTrackingArea(identifier:)`** **[NEW]** — returns `TrackingArea`
- **`TrackingArea.addHoverEffect(_:)`** **[NEW]** — `.automatic`
- **`TrackingArea.renderValue`** **[NEW]** — scalar to write into the tracking areas texture
- **`LayerRenderer.Drawable.trackingAreaTexture`** **[NEW]** — the 8-bit tracking area texture
- **`LayerRenderer.Configuration.maxRenderQuality`** **[NEW]** — `LayerRenderer.RenderQuality`
- **`LayerRenderer.RenderQuality`** **[NEW]** — `init(_ value: Double)` (0.0–1.0)
- **`layerRenderer.renderQuality`** **[NEW]** — runtime settable property
- **`LayerRenderer.Configuration.isFoveationEnabled`** (existing)
- **`LayerRenderer.Configuration.drawableRenderContextStencilFormat`** **[NEW]** — `.stencil8`
- **`LayerRenderer.Capabilities.drawableRenderContextSupportedStencilFormats`** **[NEW]**
- **`Drawable.addRenderContext(commandBuffer:)`** **[NEW]** — returns `DrawableRenderContext`
- **`DrawableRenderContext.drawMaskOnStencilAttachment(commandEncoder:value:)`** **[NEW]**
- **`DrawableRenderContext.endEncoding(commandEncoder:)`** **[NEW]** — replaces direct encoder end

### SwiftUI
- **`RemoteImmersiveSpace`** **[NEW]** — macOS scene type for streaming to Vision Pro
- **`\.remoteDeviceIdentifier`** **[NEW]** — `SwiftUI.Environment` property
- **`\.supportsRemoteScenes`** **[NEW]** — environment variable to check Mac capability
- `ImmersionStyle.progressive` **[NEW]**
- `onSpatialEvent` modifier (existing, now also on macOS for spatial accessory input)

### ARKit
- `ARKitSession(device:)` **[NEW]** — initializer accepting a remote device identifier (macOS)
- `WorldTrackingProvider` — now available on macOS
- `ar_device_t` / `ar_session_create_with_device(_:)` — C API equivalents

### SpatialEventCollection
- `SpatialEventCollection.Event.trackingAreaIdentifier` **[NEW]** — nullable tracking area ID

## Code Highlights

```swift
// Replace queryDrawable with queryDrawables
let drawables = frame.queryDrawables()
guard !drawables.isEmpty else { return }
scene.render(to: drawables)

// Add a hover effect
if object.isInteractive {
    let trackingArea = drawable.addTrackingArea(identifier: object.identifier)
    if object.usesHoverEffect { trackingArea.addHoverEffect(.automatic) }
    renderValue = trackingArea.renderValue
}

// Set max and runtime render quality
configuration.maxRenderQuality = MyScene.Constants.maxRenderQuality
layerRenderer.renderQuality = scene.renderQuality  // per-scene-type

// Progressive immersion stencil masking
let drawableRenderContext = drawable.addRenderContext(commandBuffer: commandBuffer)
drawableRenderContext.drawMaskOnStencilAttachment(commandEncoder: renderEncoder, value: 200)
renderEncoder.setStencilReferenceValue(200)
drawableRenderContext.endEncoding(commandEncoder: renderEncoder)
```

## Takeaways
- Replace `queryDrawable()` with `queryDrawables()` as the first step — all new features require it.
- Use hover effects with the tracking areas texture to give users clear signals about interactive objects; handle MSAA with a custom tile resolver.
- Profile with Instruments and Metal debugger before raising `maxRenderQuality`; higher values cost memory and power.
- For existing Mac apps, `RemoteImmersiveSpace` is the fastest path to adding immersive experiences without a separate visionOS app.

---
_Source: WWDC25 Session 294 page (abstract, chapter summaries, code samples, and resource links)._
