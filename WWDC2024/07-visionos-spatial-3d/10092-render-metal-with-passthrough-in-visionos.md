# Render Metal with Passthrough in visionOS
**WWDC24 · Session 10092** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10092/)

_Platforms:_ visionOS 2

## Overview
This session extends the Metal + Compositor Services rendering model introduced in visionOS 1 to support mixed immersion style — where rendered 3D content appears blended with the person's real physical surroundings through passthrough. The session explains exactly what needs to change from a full-immersion rendering pipeline to achieve correct passthrough compositing, and covers three primary concerns: blending rendered content with surroundings, positioning content accurately in the physical world, and handling latency-sensitive trackable anchor prediction.

The talk is aimed at developers already using ARKit, Metal, and Compositor Services. It pairs closely with the "Create enhanced spatial computing experiences with ARKit" session and prior visionOS Metal sessions.

## Key Topics

**Mix Rendered Content with Surroundings**
Switching to mixed immersion requires setting the `immersionStyle` to `.mixed`. The drawable clear color must change: full immersion uses `(0,0,0,1)` (opaque black), while mixed immersion requires all zeros `(0,0,0,0)`. The visionOS compositor expects pre-multiplied alpha in P3 display color space. Depth is in reverse-Z convention and must be zero for any pixel with zero alpha.

Upper limb visibility (hand rendering) has three modes: `.visible`, `.hidden`, and `.automatic`. Automatic mode uses per-pixel depth comparison to decide whether the hand appears in front of or behind rendered content — the system handles all compositing math.

**Position Rendered Content**
A new scene-aware projection matrix API replaces the standard projection matrix for mixed immersion apps. It combines camera intrinsics with real-time per-frame scene understanding to better align rendered content with real-world objects. Apps using Compositor Services with mixed immersion **must** use this new `computeProjection` API. The `ProjectionViewMatrix` is assembled from: `projection × (originFromDevice × deviceFromView).inverse`.

Intermediate conventions (Y-axis flipping, winding order) are the app's responsibility to reconcile. The final drawable must match Compositor Services' convention: X left-to-right, Y bottom-to-top, front-to-back winding.

**Trackable Anchor Prediction**
Compositor Services exposes four timing values per frame: Optimal Input Time, Trackable Anchor Time, Rendering Deadline, and Presentation Time. Non-critical simulation work (physics, interaction logic) should complete before Optimal Input Time. Device anchor should be queried at Presentation Time. Trackable anchors (e.g., hand tracking) must be queried at Trackable Anchor Time — not Presentation Time — for optimal accuracy.

## APIs & Frameworks

**Compositor Services**
- `CompositorLayer(configuration:)` — render loop entry point
- `.immersionStyle(selection:in:)` — now supports `.mixed` **[NEW]**
- `.upperLimbVisibility(_:)` — **[NEW]** `.visible`, `.hidden`, `.automatic`
- `LayerRenderer.Drawable` — `colorTextures`, `depthTextures`, `frameTiming`, `deviceAnchor`, `views`
- `LayerRenderer.Drawable.frameTiming` — **[NEW]** `presentationTime`, `trackableAnchorTime`
- `drawable.computeProjection(normalizedDeviceCoordinatesConvention:viewIndex:)` — **[NEW]** scene-aware projection matrix
- `cp_view_get_transform` — `deviceFromView` transform
- `LayerRenderer.Clock.Instant.epoch.duration(to:).timeInterval` — timestamp conversion

**ARKit**
- `WorldTrackingProvider` — `queryDeviceAnchor(atTimestamp:)`
- `HandTrackingProvider` — `handAnchors(at:)`
- `ar_anchor_get_origin_from_anchor` — `originFromDevice` transform
- World anchors, plane detection for scene understanding and physics

**Metal**
- `MTLRenderPassDescriptor` — color/depth clear configuration
- `MTLRenderPassColorAttachmentDescriptor` — `clearColor`, `loadAction`, `storeAction`
- `MTLRenderPassDepthAttachmentDescriptor` — `clearDepth = 0.0`
- Pre-multiplied alpha convention in shaders
- P3 display color space for assets
- Reverse-Z depth convention

**SwiftUI / App Manifest**
- `ImmersiveSpace` with `CompositorLayer`
- `PreferredDefaultSceneSessionRole` key → `CPSceneSessionRoleImmersiveApplication`
- `InitialImmersionStyle` key → `UIImmersionStyleMixed`

## Code Highlights

Mixed immersion clear color setup:
```swift
renderPassDescriptor.colorAttachments[0].clearColor = .init(red: 0.0, green: 0.0, blue: 0.0, alpha: 0.0)
renderPassDescriptor.depthAttachment.clearDepth = 0.0
```

Scene-aware projection view matrix assembly:
```swift
let originFromDevice = deviceAnchor?.originFromAnchorTransform
let deviceFromView = view.transform
let viewMatrix = (originFromDevice * deviceFromView).inverse
let projection = drawable.computeProjection(
    normalizedDeviceCoordinatesConvention: .rightUpBack,
    viewIndex: viewIndex)
let projectionViewMatrix = projection * viewMatrix
```

Trackable anchor prediction timing:
```swift
let devicePredictionTime = LayerRenderer.Clock.Instant.epoch.duration(to: presentationTime).timeInterval
let anchorPredictionTime = LayerRenderer.Clock.Instant.epoch.duration(to: trackableAnchorTime).timeInterval
let deviceAnchor = worldTracking.queryDeviceAnchor(atTimestamp: devicePredictionTime)
let leftAnchor = handTracking.handAnchors(at: anchorPredictionTime)
```

## Takeaways
- Use `(0,0,0,0)` as the clear color in mixed immersion; `(0,0,0,1)` is for full immersion only.
- Always call `drawable.computeProjection` (new API) for mixed immersion apps — it is mandatory.
- Query trackable anchors at `trackableAnchorTime` and device anchors at `presentationTime` — using the wrong timestamp degrades accuracy.
- Use `.upperLimbVisibility(.automatic)` to let the system handle hand occlusion compositing based on depth.

---
_Source: WWDC24 Session 10092 page (abstract, chapter summaries, code samples, and resource links)._
