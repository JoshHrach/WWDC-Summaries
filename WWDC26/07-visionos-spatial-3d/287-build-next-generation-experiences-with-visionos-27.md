# Build next-generation experiences with visionOS 27
**WWDC26 · Session 287** · [Watch](https://developer.apple.com/videos/play/wwdc2026/287/)

_Platforms:_ visionOS 27, macOS 27, iOS 27

## Overview
This session is the top-level orientation for visionOS 27, covering every major pathway to building apps and experiences on the platform. It recaps the visionOS scene model (Shared Space, Volumes, Windows, Immersive Spaces) and presents three paths to building: bringing existing iOS/iPadOS apps via compatibility or recompilation; building natively with Apple frameworks, third-party game engines, or custom Metal renderers; and a new third path for bringing existing macOS or PC experiences to Apple Vision Pro using Spatial Preview or Foveated Streaming.

The RealityKit and Reality Composer Pro highlights cover physical space lighting, cloth simulation, acoustic ray tracing, and Gaussian Splatting. Reality Composer Pro 3 introduces AI-assisted content creation, a collaborative workflow, Animation Graph, Script Graph, and enhanced Shader Graph materials. Third-party engine support is updated for Unity PolySpatial, Unreal Engine, and Godot, and new spatial controller, ARKit, and PHASE audio plug-ins are available on GitHub.

New platform capabilities include high-frame-rate object tracking (now also on iOS), spatial accessories (custom tracked hardware), Spatial Preview for sending content from a Mac to Apple Vision Pro, and Foveated Streaming for streaming full PC/Mac experiences wirelessly. The immersive media pipeline gains live production tooling via SMPTE 2110, wide-aspect-ratio portal support, and static foveation for streamable Apple Immersive Video delivery. Additional visionOS 27 updates include wider Safari windows, Web Environments on by default, a redesigned Control Center, accessory widget support, and upcoming Siri improvements, the Iceland environment, Spatial Panoramas, and Personal Environments.

## Key Topics

### visionOS Scene Model
- Shared Space, Volumes, Windows, and Immersive Spaces remain the core building blocks.
- Three paths to ship: iOS app compat/recompile, native spatial app (native/engine/custom), and new Mac/PC streaming path.

### RealityKit and Reality Composer Pro 3
- Physical space lighting: virtual lights illuminate the real world (`SpotLightComponent.SurroundingsLight`).
- Cloth simulation with `ClothBodyComponent`.
- Acoustic ray tracing for custom reverb meshes.
- 3D Gaussian Splatting: `GaussianSplatComponent` / `GaussianSplatResource`.
- Reality Composer Pro 3 is now a standalone download; includes Animation Graph, Script Graph, Compute Graph, Behavior Trees, AI assistant for generating 3D objects and materials.

### Third-Party Game Engines
- Unity PolySpatial, Unreal Engine, and Godot all updated for visionOS 27.
- Spatial controller, ARKit, and PHASE audio plug-ins on GitHub.
- Custom renderer path via `CompositorServices`.

### Spatial Preview
- New `SpatialPreview` framework (macOS 27): send 3D assets, spatial photos, and Apple Immersive Video from a Mac directly to Apple Vision Pro.
- Supports `DocumentPreviewSession` and `USDPreviewSession`; no visionOS code required.
- SharePlay support for collaborative review.

### Foveated Streaming
- New `FoveatedStreaming` framework streams existing macOS/PC OpenXR experiences to Apple Vision Pro wirelessly.
- Eye-tracked, foveated video compression achieves high visual quality at practical bandwidth.
- Demonstrated with NVIDIA CloudXR; open-source sample on GitHub.

### Object Tracking and Spatial Accessories
- High-frame-rate tracking for handheld/moving objects; now available on iOS via `ARWorldTrackingConfiguration.trackingObjects`.
- Spatial accessories: third-party custom hardware with LED constellation + IMU + Bluetooth, tracked via `AccessoryTrackingProvider` / `GCSpatialAccessory`.

### Immersive Media
- Apple Immersive Video pipeline: AIV formats, Immersive Media Support framework for rich metadata.
- Live production via SMPTE 2110 standard.
- Wide-aspect-ratio portal support and static foveation for streamable AIV delivery.

### Other visionOS 27 Updates
- Wider Safari windows; Web Environments on by default.
- Redesigned Control Center, streamlined notifications, high-quality capture.
- Accessory widget support for glanceable info on Apple Vision Pro.
- Coming later: Siri enhancements, Iceland environment, Spatial Panoramas, Personal Environments.

## APIs & Frameworks

### RealityKit
- `SpotLightComponent` — existing; `SpotLightComponent.SurroundingsLight` **[NEW]**: physical space lighting
- `SpotLightComponent.Shadow` — `lightSize`, `quality` (.low/.medium/.high) **[NEW]**: soft shadows
- `SpotLightComponent.ProjectiveTexture` **[NEW]**: projective texture on spotlights
- `ClothBodyComponent` **[NEW]**: cloth simulation
- `ClothColliderComponent` **[NEW]**: cloth collision
- `ClothSphereShape` **[NEW]**: sphere shape for pinning cloth vertices
- `NavigationMeshResource` **[NEW]**: navigation mesh for pathfinding
- `NavigationComponent` **[NEW]**
- `NavigationController` **[NEW]**: async path query (`computePath`)
- `LevelOfDetailComponent` **[NEW]**: LOD by camera distance or screen area
- `GaussianSplatResource` / `GaussianSplatResource.BufferResource` **[NEW]**
- `GaussianSplatComponent` **[NEW]**
- `ReverbComponent` **[NEW]**: custom acoustic reverb
- `ReverbMeshResource` **[NEW]**: `.shoebox(size:)` and other presets
- `Audio.Material`, `Audio.Absorption`, `Audio.Scattering` **[NEW]**: custom reverb materials
- `ManipulationComponent` — `releaseBehavior` property
- `InputTargetComponent`
- `ClippingComponent` **[NEW]**: cross-sectional clipping planes
- `ShaderGraphMaterial` — `setParameter(name:value:)`
- `ModelComponent`
- `EntityAction` protocol **[NEW]**: custom animation actions in RealityKit
- `@Scriptable` macro **[NEW]**: exposes components to Script Graph

### SpatialPreview (NEW framework)
- `DocumentPreviewSession` **[NEW]**: send documents (USDZ, AIVU, etc.) to visionOS
- `USDPreviewSession` **[NEW]**: live-edit USD stages on visionOS
- `ConnectedSpatialEndpointObserver` **[NEW]**: discover connected Apple Vision Pro
- `SpatialPreviewEndpoint` **[NEW]**
- `SpatialPreviewDevicePicker` **[NEW]**: SwiftUI device picker
- `USDPreviewSession.Error.assetUnshareable`
- Session options: `.annotations`, `.perObjectManipulation`, `.export`
- Events: `.timeChanged`, `.playbackStateChanged`

### FoveatedStreaming (NEW framework)
- `FoveatedStreamingSession` **[NEW]**: core session for streaming
- `ImmersiveSpace(foveatedStreaming:)` **[NEW]**: SwiftUI scene modifier
- `SpatialContainer` **[NEW]**: overlay SwiftUI content on streamed space

### USDKit (NEW framework)
- `USDStage`, `USDStage.open(_:)` **[NEW]**
- `stage.descendants` traversal **[NEW]**
- `stage.definePrim(at:type:)` **[NEW]**
- `prim.references.add(_:)` **[NEW]**
- `prim.addTransformOperation(type:)` **[NEW]**
- `stage.exportPackage(to:options:)` — `.preferSmallTextureFiles`, `.preferSmallMeshFiles` **[NEW]**
- `prim.applyAPISchema(_:instanceName:)` — AccessibilityAPI support **[NEW]**
- `UsdStage.ObjectsDidChange` observation **[NEW]**

### ARKit
- `ReferenceObject.Configuration` — `highFrameRateTrackingEnabled` **[NEW]**
- `myObjectAnchor.coordinateSpace(correction:)` — `.rendered` / `.none` **[NEW]**
- `ARWorldTrackingConfiguration.trackingObjects` **[NEW]**: high-frame-rate object tracking on iOS
- `AccessoryTrackingProvider` **[NEW]**: spatial accessory tracking
- `GCSpatialAccessory` (GameController) **[NEW]**: accessory discovery

### Reality Composer Pro 3 Tools (NEW)
- Animation Graph (State Machine, transitions, runtime parameters)
- Behavior Tree (Sequence, Selector, Parallel, Move To, Rotate To Face, Wait, Parameter Setter)
- Script Graph (On Initialize, On Tap, subgraphs, custom events)
- Compute Graph (GPU particle simulation via Metal: Emitter, Initialize, Simulate, Output phases)
- Navigation Mesh (bounding box config, off-mesh connections, cell size tuning)
- Shader Graph: `RealityKit PBR Surface 2` with sheen and subsurface scattering **[NEW]**, `Hair Surface` **[NEW]**, portal rendering support **[NEW]**
- `RealityComposerProPlugin` protocol **[NEW]**: Xcode plugin for editor extension
- `RealityComposerProContext.registerComponent/registerSystem/registerAction` **[NEW]**
- `RKS.Configuration`, `RKS.addConfiguration(_:)` **[NEW]**: scripting module registration

### VideoToolbox / ImmersiveMedia
- `kVTCompressionPropertyKey_ProjectionKind` — `kVTProjectionKind_AppleImmersiveVideo` **[NEW]**
- `ImmersiveMediaSupport` framework: metadata read/write for AIV
- `AVAssetWriter` for live AIV recording

### Web / Safari
- `<model>` HTML element with `src` / `environmentmap` attributes **[NEW]**: 3D USD on web
- JavaScript Immersive API: `requestImmersive()`, `immersivechange` event, `:immersive` CSS pseudo-class **[NEW]**
- `document.immersiveEnabled`, `document.immersiveElement` **[NEW]**
- `model.entityTransform` (DOMMatrix) **[NEW]**
- `usdcrush` command-line tool **[NEW]**: mesh and texture compression

## Code Highlights

Enable physical space lighting on a spotlight:
```swift
spotLightEntity.components.set(SpotLightComponent.SurroundingsLight())
```

Enable high-frame-rate object tracking:
```swift
var configuration = ReferenceObject.Configuration()
configuration.highFrameRateTrackingEnabled = true
```

Connect to a foveated streaming endpoint:
```swift
ImmersiveSpace(foveatedStreaming: session) { ... }
```

Request immersive transition on a web model element:
```javascript
await model.requestImmersive();
```

## Takeaways
- visionOS 27 adds a formal third path for Mac/PC experiences via Spatial Preview and Foveated Streaming, lowering the barrier for cross-platform content significantly.
- RealityKit gains major physical simulation capabilities (cloth, navmesh, acoustic ray tracing, Gaussian splatting) alongside a physical space lighting model.
- Reality Composer Pro 3 becomes a standalone tool with AI-assisted content creation, enabling artists to build games and spatial experiences without writing code.
- Object tracking expands to iOS and custom spatial accessories open a new accessory hardware category for Apple Vision Pro.

---
_Source: WWDC26 Session 287 page (abstract, chapter summaries, code samples, and resource links)._
