# SceneKit: What's New
**WWDC17 · Session 604** · [Watch](https://developer.apple.com/videos/play/wwdc2017/604/)

_Platforms:_ iOS 11, macOS High Sierra 10.13, tvOS 11

## Overview
SceneKit's WWDC17 update is organized around four themes: a new physically based camera system with cinematic depth of field, motion blur, and screen space ambient occlusion; GPU-accelerated tessellation and subdivision surfaces (including an integration with Pixar's OpenSubdiv via Metal); revamped animation APIs with animation blending and SCNAnimationPlayer; and a set of new developer tools including a SceneKit Instruments template and an enhanced Xcode Scene Editor. The session is anchored by the Fox 2 sample — an action game that demonstrates all the new APIs together, rendering 3,200 animated bots at 0.5 ms on iPad Pro.

The camera section replaces legacy `xFov`/`yFov` projection with real-optics concepts (`focalLength`, `sensorHeight`, `fStop`, `focusDistance`), allowing developers to configure depth of field using photography vocabulary. The tessellation section shows how `SCNGeometryTessellator` can subdivide meshes on the GPU, and how coupling tessellation with displacement maps or Pixar OpenSubdiv subdivision surfaces produces smooth, memory-efficient assets. The animation section introduces `SCNAnimationPlayer` for runtime-mutable animations and blending between multiple animation files.

## Key Topics
- **Physically based camera** — replace `xFov`/`yFov` with `fieldOfView`, `focalLength`, `sensorHeight`; depth of field via `wantsDepthOfField`, `focusDistance`, `fStop`; HDR rendering with `wantsHDR` needed for proper bokeh; bokeh blade count configurable on `SCNCamera`
- **Per-object motion blur** — new in iOS 11; extends previous camera-motion blur to cover individually moving objects; activated on `SCNCamera`
- **Screen space ambient occlusion (SSAO)** — new in iOS 11; computed from depth and normal buffers; `screenSpaceAmbientOcclusionIntensity` > 0 enables it; configurable radius and bias; complements baked ambient occlusion maps for dynamic objects
- **`SCNCameraController`** — **[NEW]** replaces the non-configurable `allowCameraControl` debug property; built-in modes: Orbit Turntable (prevents roll), Orbit Arcball, Fly; `SCNView.defaultCameraController` for drop-in use; instantiate directly for programmatic control
- **Camera constraints** — new `SCNDistanceConstraint`, `SCNReplicatorConstraint`, `SCNAccelerationConstraint` added to existing constraint system; chaining constraints (look-at + distance + acceleration) defines complete game camera behavior without code
- **SIMD properties on SCNNode** — `position`, `rotation`, `scale`, `transform`, `worldTransform` now exposed as SIMD types; easier math, more performant; not KVO/KVC compliant
- **`SCNGeometryTessellator`** — **[NEW]** uniform or screen-space adaptive tessellation; `maximumEdgeLength` in pixels for screen-space mode; constant or adaptive tessellation factors
- **Displacement maps** — new `displacement` material property on `SCNMaterial`; gray-scale (height map) or vector (RGBA) displacement; intensity animatable; works with tessellation pipeline
- **Geometry smoothing** — `pnTriangles` smoothing mode uses vertex positions and normals to project tessellated geometry onto a smooth surface
- **OpenSubdiv / subdivision surfaces** — now GPU-accelerated via Metal-based OpenSubdiv contributed by Apple; requires `preserveOriginalTopology` option on scene load; creases and corners define sharpness; feature-adaptive subdivision isolates irregular parts for efficient mesh generation; all-GPU pipeline: coarse mesh → morph → skinning → GPU subdivision
- **`SCNAnimationPlayer`** — **[NEW]** runtime mutable; change `speed` while animation runs; blend two animations by weight; conforms to new `SCNAnimation` protocol; replaces `addAnimation(_:forKey:)`
- **Animation blending** — **[NEW]** transition from one animation to another with a blend weight over a duration; enables walk→run→step transitions without abrupt cuts
- **SceneKit Instruments template** — **[NEW]** records per-frame render time, GPU rendering time, update stage time, texture upload and shader compile time; combinable with Metal GPU instrument; frame timeline reveals shader compile spikes
- **View debugger integration** — Xcode view debugger captures SceneKit scene automatically; sends to Scene Editor for inspection and camera manipulation
- **Scene Editor enhancements** — new camera behavior selector (perspective, orthographic, fly, arcball); revamped Shader Modifier Editor with material property editing on one screen; support for displacement material slot, new constraints, cascaded shadow maps, procedural sky; reference node material override
- **ARKit integration** — `ARSCNView` is an `SCNView` subclass; full SceneKit scene graph, physics, particles, post-processes available; `SCNMaterialProperty` now supports `AVCaptureDevice` and `AVPlayer` as content types (direct camera feed to texture in one line)
- **Shadow trick for AR** — configure a shadow-receiving plane with color buffer writes disabled; it still writes to depth buffer; use default shadow technique for a second-pass shadow composite over camera image
- **GameplayKit entity/component** — GKScene entities drive SceneKit nodes; behaviors implemented as GKComponents; assign in Xcode, edit properties in inspector
- **Model I/O USD improvements** — improved USD support for materials and animation
- **UIFocus on tvOS** — `SCNNode` now conforms to `UIFocusItem`; SceneKit computes projected screen area for the focus engine; focus engine calls delegate when selection changes
- **Point cloud rendering** — new properties on point-primitive geometry for screen-space size, world-space size, texture, and lighting
- **New transparency modes** — `singleLayer` (one pass back-faces, one pass front-faces only, eliminates overlapping polygon artifacts during fade); `dualLayer` (back then front, correct double-sided transparency)
- **Cascaded shadow maps** — configure cascade count, size, and `shadowCascadeSplittingFactor` on `SCNLight`; visualizable in Xcode; allocates more precision near the camera

## APIs & Frameworks

### SceneKit
- **`SCNCamera.fieldOfView`** — replaces deprecated `xFov`/`yFov`; linked to `focalLength` and `sensorHeight`
- **`SCNCamera.focalLength`** / **`SCNCamera.sensorHeight`** — physically based projection parameters; changing one updates `fieldOfView`
- **`SCNCamera.wantsDepthOfField`** — `Bool`; enables physically plausible depth of field
- **`SCNCamera.focusDistance`** — distance in scene units to the focal plane
- **`SCNCamera.fStop`** — aperture f-stop; smaller = more blur
- **`SCNCamera.wantsHDR`** — `Bool`; required for correct bokeh rendering
- **`SCNCamera.apertureBladeCount`** — number of aperture blades affecting bokeh shape
- **`SCNCamera.motionBlurIntensity`** — enables per-object motion blur **[NEW object-motion support in iOS 11]**
- **`SCNCamera.screenSpaceAmbientOcclusionIntensity`** — **[NEW]** enables SSAO; `> 0` to activate
- **`SCNCamera.screenSpaceAmbientOcclusionRadius`** — search radius in scene units
- **`SCNCameraController`** — **[NEW]** camera manipulation controller; `interactionMode` (`.orbitTurntable`, `.orbitArcball`, `.fly`); `target`, `minimumVerticalAngle`, `maximumVerticalAngle`
- **`SCNView.defaultCameraController`** — pre-configured `SCNCameraController` for simple use
- **`SCNDistanceConstraint`** — **[NEW]** minimum/maximum distance to a target node
- **`SCNReplicatorConstraint`** — **[NEW]** replicate position/orientation of another node with offset
- **`SCNAccelerationConstraint`** — **[NEW]** clamp maximum velocity and acceleration of a node
- **`SCNGeometryTessellator`** — **[NEW]** tessellation configuration object; `tessellationFactorScale`, `maximumEdgeLength`, `isAdaptive`, `isScreenSpace`
- **`SCNGeometry.tessellator`** — **[NEW]** optional `SCNGeometryTessellator`; enables tessellation pipeline for this geometry
- **`SCNMaterial.displacement`** — **[NEW]** `SCNMaterialProperty`; gray-scale or vector displacement map used with tessellation
- **`SCNAnimationPlayer`** — **[NEW]** manages one animation; `speed`, `blendFactor`, `play()`, `stop()`, `paused`
- **`SCNAnimatable.animationPlayer(forKey:)`** — returns running `SCNAnimationPlayer` by key
- **`SCNAnimation`** — **[NEW]** protocol; `CAAnimation` conforms to it; `SCNAnimationPlayer(animation:)` wraps any conforming animation
- **`SCNLight.shadowCascadeCount`** — number of cascades for cascaded shadow maps
- **`SCNLight.shadowCascadeSplittingFactor`** — distribution of cascade boundaries by distance
- **`SCNNode.simdPosition`** / **`simdTransform`** etc. — SIMD-typed accessors for all node transform properties **[NEW]**
- **`SCNMaterialProperty.contents`** — now accepts `AVCaptureDevice` and `AVPlayer` directly

### ARKit
- **`ARSCNView`** — `SCNView` subclass; scene graph, physics, particles, post-processes all available; foundation for integrating SceneKit with ARKit sessions

## Code Highlights

```swift
// Physically based depth of field
let camera = SCNCamera()
camera.fieldOfView = 55
camera.wantsDepthOfField = true
camera.focusDistance = 2.5
camera.fStop = 1.4
camera.wantsHDR = true
camera.apertureBladeCount = 6

// Screen space ambient occlusion
camera.screenSpaceAmbientOcclusionIntensity = 0.75
camera.screenSpaceAmbientOcclusionRadius = 0.05
```

```swift
// Camera constraint chain (game camera behavior)
let lookAt = SCNLookAtConstraint(target: characterNode)
let distance = SCNDistanceConstraint(target: characterNode)
distance.minimumDistance = 3.0
distance.maximumDistance = 6.0
let accel = SCNAccelerationConstraint()
accel.maximumLinearVelocity = 50.0
accel.maximumLinearAcceleration = 25.0
cameraNode.constraints = [lookAt, distance, accel]
```

```swift
// Tessellation + displacement map
let tessellator = SCNGeometryTessellator()
tessellator.isAdaptive = true
tessellator.isScreenSpace = true
tessellator.maximumEdgeLength = 8   // pixels
geometry.tessellator = tessellator

let displacementMap = UIImage(named: "terrain_height")
geometry.firstMaterial?.displacement.contents = displacementMap
geometry.firstMaterial?.displacement.intensity = 0.5
```

```swift
// SCNAnimationPlayer — blend walk into run
let walkPlayer = SCNAnimationPlayer(animation: walkAnimation)
let runPlayer  = SCNAnimationPlayer(animation: runAnimation)
characterNode.addAnimationPlayer(walkPlayer, forKey: "walk")
characterNode.addAnimationPlayer(runPlayer,  forKey: "run")

walkPlayer.play()
// Blend to run at runtime
runPlayer.blendFactor = 0.0
runPlayer.play()
UIView.animate(withDuration: 0.3) { runPlayer.blendFactor = 1.0 }
```

```swift
// AR shadow trick — plane receives shadows but is invisible
let plane = SCNNode(geometry: SCNPlane(width: 10, height: 10))
plane.geometry?.firstMaterial?.writesToDepthBuffer = true
plane.geometry?.firstMaterial?.colorBufferWriteMask = []  // no color writes
let light = SCNLight()
light.type = .directional
light.shadowMode = .default   // screen-space shadow composite
```

## Takeaways
- `SCNCameraController` and the new constraint types (`SCNDistanceConstraint`, `SCNReplicatorConstraint`, `SCNAccelerationConstraint`) make sophisticated game camera behaviors declarative — chain constraints instead of writing per-frame position math.
- Tessellation + displacement maps let artists create high-detail geometry (terrain, character skin, rocks) from a low-polygon base mesh with minimal runtime memory; screen-space adaptive mode automatically adjusts detail based on camera distance.
- GPU-accelerated OpenSubdiv (via Metal) runs subdivision on the GPU at the very end of the animation pipeline, keeping the deformed mesh compact until the final render step.
- `SCNAnimationPlayer` makes previously-static animations mutable at runtime: change speed, blend weight, and pause/resume without removing and re-adding the animation from the node.

---
_Source: WWDC17 Session 604 page (abstract, transcript, and resource links)._
