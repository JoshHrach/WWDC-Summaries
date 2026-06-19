# Bring Your World into Augmented Reality
**WWDC22 · Session 10128** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10128/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13

## Overview
This session demonstrates an end-to-end workflow for bringing real-world objects into an AR experience using Object Capture and RealityKit. Object Capture — introduced in 2021 — uses photogrammetry to reconstruct high-quality USDZ 3D models from iPhone or iPad photos. The session recaps the Object Capture API, introduces two new ARKit camera enhancements for high-resolution photo capture during an active AR session, and covers best practice guidelines for capture environment setup, object selection, and photography technique.

The second half walks through a practical demo building an AR chess game from real wooden chess pieces. It covers importing Object Capture models into Xcode, using Reality Converter to swap textures, writing RealityKit startup animations, implementing tap-to-select with raycasting, building Metal surface shaders for selection glow effects, adding geometry modifiers for capture animations, and applying a bloom post-processing effect via `ARView.renderCallbacks`.

## Key Topics

### Object Capture Recap & New ARKit Camera APIs
- Object Capture (macOS, WWDC21): `PhotogrammetrySession` API producing USDZ from still photos at four detail levels (reduced, medium, full, raw)
- New ARKit high-resolution background photo API: `ARWorldTrackingConfiguration.recommendedVideoFormatForHighResolutionFrameCapturing` **[NEW]**
- `ARSession.captureHighResolutionFrame(completionHandler:)` — captures up to 12 MP native-resolution photo without interrupting the running AR session **[NEW]**
- `ARWorldTrackingConfiguration.configurableCaptureDeviceForPrimaryCamera` — direct access to `AVCaptureDevice` for manual focus, exposure, and white balance **[NEW]**
- Output USDZ from iPhone/iPad captures includes gravity vector and real-world scale estimation automatically

### Object Capture Best Practices
- Objects must have adequate surface texture, no transparency, matte surface, and be rigid when flipped
- Environment: even diffuse lighting, stable background, clear space around object
- Photography: maintain high overlap between adjacent frames; 80–100 photos for typical objects; portrait vs. landscape mode for tall vs. wide objects
- Detail level selection: reduced/medium for AR QuickLook and mobile; full/raw for games and post-production
- Reality Converter: import USDZ, replace/invert textures, export with compressed textures to minimize memory footprint

### RealityKit Integration
- Load photogrammetry USDZ assets directly into Xcode project and reference via `Entity`
- `Entity.move(to:relativeTo:duration:)` — animate transform (translation + scale) with duration for startup sequences
- `Entity.playAnimation(_:)` — play built-in USDZ animation tracks (e.g., animated border)
- `ARView.scene.raycast(origin:direction:length:query:mask:)` — pick entities by collision group mask; entities without `CollisionComponent` are ignored
- `ARView.renderCallbacks.postProcess` — set closure for full post-processing pass with access to `ARView.PostProcessContext`

### Custom Materials: Surface Shaders & Geometry Modifiers
- `CustomMaterial` with `surfaceShader` — Metal function called per-fragment; access `uniforms().time()`, `uniforms().custom_parameter()`, facing angle, texture samples
- `CustomMaterial` with `geometryModifier` — Metal function called per-vertex; modify position via `set_model_position_offset()`
- `customMaterial.custom.value: SIMD4<Float>` — pass CPU data to GPU (e.g., animation progress `capturedProgress`)
- `surface().set_emissive_color()`, `surface().set_base_color()`, `surface().set_opacity()` — set fragment output values

### Bloom Post-Processing
- `ARView.renderCallbacks.postProcess` closure receives `ARView.PostProcessContext` (device, commandBuffer, sourceColorTexture, targetColorTexture)
- `MPSImageThresholdToZero` — isolate bright regions for bloom input
- Chain MPS filters in the post-process callback and blit final result to `context.targetColorTexture`

## APIs & Frameworks

**ARKit** **[NEW]**
- `ARWorldTrackingConfiguration.recommendedVideoFormatForHighResolutionFrameCapturing: ARVideoFormat?` **[NEW]**
- `ARSession.captureHighResolutionFrame(completionHandler: (ARFrame?, Error?) -> Void)` **[NEW]**
- `ARWorldTrackingConfiguration.configurableCaptureDeviceForPrimaryCamera: AVCaptureDevice?` **[NEW]**

**RealityKit**
- `PhotogrammetrySession` (macOS) — existing WWDC21 API for photogrammetry reconstruction
- `Entity.move(to: Transform, relativeTo: Entity?, duration: TimeInterval)`
- `Entity.playAnimation(_ animation: AnimationResource)`
- `Entity.availableAnimations: [AnimationResource]`
- `ARView.scene.raycast(origin:direction:length:query:mask:) -> [CollisionCastHit]`
- `CollisionCastHit.entity: Entity`
- `ARView.renderCallbacks.postProcess: ((ARView.PostProcessContext) -> Void)?`
- `ARView.PostProcessContext` — `.device`, `.commandBuffer`, `.sourceColorTexture`, `.targetColorTexture`
- `CustomMaterial` — custom surface shaders and geometry modifiers via Metal
  - `CustomMaterial.custom.value: SIMD4<Float>` — CPU-to-GPU parameter passing
- `CollisionComponent` — required for entities to participate in raycasting

**Metal / MetalPerformanceShaders**
- `MPSImageThresholdToZero(device:thresholdValue:linearGrayColorTransform:)` — threshold filter for bloom bright-pass
- `MPSImageGaussianBlur` — blur for bloom spread
- Surface shader functions: `params.uniforms().time()`, `params.uniforms().custom_parameter()`, `params.surface().set_emissive_color()`, `params.surface().set_base_color()`, `params.surface().set_opacity()`
- Geometry modifier functions: `params.geometry().set_model_position_offset()`

## Code Highlights

ARKit high-resolution frame capture:
```swift
// Select video format that supports hi-res capturing
if let hiResCaptureVideoFormat = ARWorldTrackingConfiguration
        .recommendedVideoFormatForHighResolutionFrameCapturing {
    config.videoFormat = hiResCaptureVideoFormat
}
session.run(config)

// Capture a high-res photo on demand
session.captureHighResolutionFrame { frame, error in
    if let frame = frame {
        // Use frame.capturedImage (full native resolution)
    }
}
```

Startup animation with move and scale:
```swift
class Chessboard: Entity {
    func playAnimation() {
        checkers.forEach { entity in
            let currentTransform = entity.transform
            entity.transform.translation += SIMD3<Float>(0, 0.1, 0)
            entity.move(to: currentTransform, relativeTo: entity.parent,
                        duration: BoardGame.startupAnimationDuration)
        }
        border.availableAnimations.forEach { border.playAnimation($0) }
    }
}
```

Tap-to-select with raycasting:
```swift
guard let ray = ray(through: sender.location(in: self)) else { return }
let hits = scene.raycast(origin: ray.origin, direction: ray.direction,
                          length: 5, query: .nearest, mask: .piece)
if let piece = hits.first?.entity.parentChessPiece {
    boardGame.select(piece)
}
```

Passing animation progress to geometry modifier:
```swift
var capturedProgress: Float {
    get { (pieceEntity?.model?.materials.first as? CustomMaterial)?.custom.value[0] ?? 0 }
    set {
        pieceEntity?.modifyMaterials { material in
            guard var cm = material as? CustomMaterial else { return material }
            cm.custom.value = SIMD4<Float>(newValue, 0, 0, 0)
            return cm
        }
    }
}
```

## Takeaways
- `ARSession.captureHighResolutionFrame` captures up to 12 MP photos during a live AR session without interrupting tracking — ideal for photogrammetry capture apps that also show a guidance UI.
- Object Capture produces deployment-ready USDZ models in minutes; choose the reduced or medium detail level for AR QuickLook and mobile AR apps.
- `CustomMaterial.custom.value` (a `SIMD4<Float>`) is the bridge between CPU-side animation state and Metal surface shaders or geometry modifiers running on the GPU.
- `ARView.renderCallbacks.postProcess` combined with Metal Performance Shaders enables full-screen effects like bloom without leaving the RealityKit render loop.

---
_Source: WWDC22 Session 10128 page (abstract, transcript, and code samples)._
