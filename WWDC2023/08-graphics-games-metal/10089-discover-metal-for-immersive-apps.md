# Discover Metal for Immersive Apps
**WWDC23 · Session 10089** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10089/)

_Platforms:_ visionOS 1

## Overview
This session by Apple engineer Pau Sastre Miguel introduces the complete Metal rendering pipeline for fully immersive visionOS apps. It covers the new CompositorServices framework — the low-latency bridge between a custom Metal rendering engine and the visionOS compositor server — and explains how to set up a render loop, configure the LayerRenderer, render frames with correct timing, and integrate ARKit hand tracking and pinch spatial events for interactivity.

This is the session for developers who want to write their own rendering engine (in C, C++, or Swift) rather than using RealityKit. The session uses RecRoom (via Unity) as a real-world example of an app already using these APIs.

## Key Topics

### App Architecture
A visionOS app with a custom Metal renderer uses:
1. **SwiftUI** (`App` protocol + `ImmersiveSpace` scene) to bootstrap the app.
2. **CompositorServices** (`CompositorLayer`) to create and own the rendering session.
3. **Custom engine in C/C++/Swift** for the render loop, running on a dedicated render thread.
4. **ARKit** for world tracking and hand tracking to drive interaction.

The `ImmersiveSpace` scene with `CompositorLayer` is used instead of `ImmersiveSpace` with `RealityView`. Both are valid approaches; CompositorLayer is for apps with full control of the render pipeline.

### CompositorLayerConfiguration **[NEW]**
A type conforming to `CompositorLayerConfiguration` configures the LayerRenderer before rendering begins. Key configuration properties:
- **`isFoveationEnabled`**: Foveated rendering reduces power while maintaining visual quality. Uses `MTLRasterizationRateMap` per frame; the map concentrates pixel density in the center. Check `capabilities.supportsFoveation` first (not available in Simulator).
- **`layout`**: How eye renders map to Metal textures.
  - `.layered` — one texture, two array slices, two viewports; single-pass rendering with foveation. **Recommended.**
  - `.dedicated` — two textures, one slice each.
  - `.shared` — one texture, one slice, two viewports. Easiest port of existing engines (no foveation).
- **`colorFormat`**: Defaults to SDR format. Set `.rgba16Float` for HDR (EDR headroom 2.0, extended linear Display P3).

### Render Loop Structure
The render loop runs on its own thread. States from `cp_layer_renderer_get_state`:
- `.paused` — call `cp_layer_renderer_wait_until_running()`.
- `.running` — render a frame.
- `.invalidated` — clean up and exit.

### Frame Rendering Timeline
Each frame has two sections:
1. **Update** (`cp_frame_start_update` / `cp_frame_end_update`): Non-latency-critical work — update animations, characters, gather queued inputs.
2. **Submission** (`cp_frame_start_submission` / `cp_frame_end_submission`): Latency-critical work — headset-pose-dependent rendering.

`cp_frame_timing_t` provides three critical timestamps:
- **Optimal input time** — best time to sample latency-critical inputs; wait for this before starting submission.
- **Rendering deadline** — CPU and GPU work must finish by this time or the compositor skips the frame.
- **Presentation time** — when the frame appears on display.

**Drawable**: `cp_frame_query_drawable` returns the render target. Its `cp_drawable_get_frame_timing` gives final timing; set `cp_drawable_set_ar_pose` with the ARKit world pose for compositor reprojection.

### Spatial Input **[NEW]**
Pinch input is delivered via `LayerRenderer.onSpatialEvent` callback (main thread):
- `SpatialEventCollection` → map to C type and push to thread-safe queue.
- `SpatialEvent` properties: `phase` (active / ended / cancelled), `selectionRay` (ray from gaze at pinch start), `manipulatorPose` (pose of pinch, updated each frame).
- Hand skeletons: `ar_hand_tracking_provider_get_latest_anchors` in the update section.

Upper limb visibility is controlled by `.upperLimbVisibility(.hidden)` on `ImmersiveSpace` (hides passthrough hands when rendering virtual hands).

## APIs & Frameworks

### CompositorServices **[NEW]**
- `CompositorLayer` — `ImmersiveSpaceContent` type that creates a Metal rendering session **[NEW]**
- `CompositorLayerConfiguration` protocol — configure rendering behavior **[NEW]**
- `LayerRenderer` — interface to the rendering session **[NEW]**
- `LayerRenderer.Capabilities` — query device features (foveation, supported layouts) **[NEW]**
- `LayerRenderer.Configuration` — mutable config: `layout`, `isFoveationEnabled`, `colorFormat` **[NEW]**
- `LayerRenderer.Layout` — `.layered`, `.dedicated`, `.shared` **[NEW]**
- `cp_layer_renderer_t` — C type for the layer renderer
- `cp_layer_renderer_get_state` — query current state **[NEW]**
- `cp_layer_renderer_wait_until_running` — blocks until renderer is running **[NEW]**
- `cp_layer_renderer_state_paused/running/invalidated` — state enum values **[NEW]**
- `cp_layer_renderer_query_next_frame` — get next frame to render **[NEW]**
- `cp_frame_t` — C type for a frame **[NEW]**
- `cp_frame_predict_timing` — get predicted timing **[NEW]**
- `cp_frame_start_update` / `cp_frame_end_update` — bracket update section **[NEW]**
- `cp_frame_start_submission` / `cp_frame_end_submission` — bracket submission section **[NEW]**
- `cp_frame_query_drawable` — get the render target **[NEW]**
- `cp_frame_timing_t` — timing with optimal input time, rendering deadline, presentation time **[NEW]**
- `cp_frame_timing_get_optimal_input_time` **[NEW]**
- `cp_drawable_t` — C type for drawable render target **[NEW]**
- `cp_drawable_get_frame_timing` **[NEW]**
- `cp_drawable_set_ar_pose` — set world pose for compositor reprojection **[NEW]**
- `cp_time_wait_until` — wait until a specific timestamp **[NEW]**
- `MTLRasterizationRateMap` — foveation rate map (one per frame from Compositor)
- `LayerRenderer.onSpatialEvent` — callback for pinch events, delivered on main thread **[NEW]**
- `SpatialEventCollection` — collection of `SpatialEvent` objects **[NEW]**
- `SpatialEvent.phase` / `.selectionRay` / `.manipulatorPose` **[NEW]**

### ARKit (visionOS) **[NEW]**
- `ar_hand_tracking_provider_t` — C type for hand tracking provider **[NEW]**
- `ar_hand_tracking_provider_get_latest_anchors` — get current hand skeleton **[NEW]**
- `ar_pose_t` — device/world pose for reprojection **[NEW]**
- World tracking provider — provides device orientation and translation

### SwiftUI
- `ImmersiveSpace` — container scene for fully immersive content **[NEW]**
- `.upperLimbVisibility(.hidden)` — hides passthrough hands **[NEW]**
- `UIApplicationPreferredDefaultSceneSessionRole` Info.plist key — set to `CPSceneSessionRoleImmersiveSpaceApplication` to skip default window creation **[NEW]**

### Metal
- `MTLRasterizationRateMap` — foveated rendering rate map
- `rgba16Float` pixel format — for HDR/EDR rendering

## Code Highlights

```swift
// App setup with CompositorLayer
@main struct MyApp: App {
    var body: some Scene {
        ImmersiveSpace {
            CompositorLayer(configuration: MyConfiguration()) { layerRenderer in
                let engine = my_engine_create(layerRenderer)
                let renderThread = Thread { my_engine_render_loop(engine) }
                renderThread.name = "Render Thread"
                renderThread.start()
                layerRenderer.onSpatialEvent = { eventCollection in
                    var events = eventCollection.map { my_spatial_event($0) }
                    my_engine_push_spatial_events(engine, &events, events.count)
                }
            }
        }
        .upperLimbVisibility(.hidden)
    }
}
```

```c
// C render loop
void my_engine_render_loop(my_engine *engine) {
    my_engine_setup_render_pipeline(engine);
    bool is_rendering = true;
    while (is_rendering) {
        switch (cp_layer_renderer_get_state(engine->layer_renderer)) {
            case cp_layer_renderer_state_paused:
                cp_layer_renderer_wait_until_running(engine->layer_renderer); break;
            case cp_layer_renderer_state_running:
                my_engine_render_new_frame(engine); break;
            case cp_layer_renderer_state_invalidated:
                is_rendering = false; break;
        }
    }
    my_engine_invalidate(engine);
}
```

## Takeaways
- Use CompositorServices + Metal when you need a fully custom rendering engine on visionOS; use RealityKit for everything else.
- Always enable foveated rendering (`.layered` layout with `isFoveationEnabled = true`) for the best visual quality and power efficiency.
- Respect the two-section frame model: non-latency work in Update, headset-pose-dependent rendering in Submission, and always wait for `optimalInputTime` before starting Submission.
- Deliver `ar_pose` to the compositor via `cp_drawable_set_ar_pose` — without it, reprojection will be incorrect and the experience will feel unsteady.

---
_Source: WWDC23 Session 10089 page (abstract, chapter summaries, code samples, and resource links)._
