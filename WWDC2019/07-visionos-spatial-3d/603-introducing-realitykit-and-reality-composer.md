# Introducing RealityKit and Reality Composer
**WWDC19 · Session 603** · [Watch](https://developer.apple.com/videos/play/wwdc2019/603/)

_Platforms:_ iOS 13, iPadOS 13, macOS (Reality Composer)

## Overview
RealityKit is a new Swift framework built specifically for augmented reality, providing a fully integrated rendering, animation, physics, spatial audio, and networking stack. Unlike general-purpose 3D engines, RealityKit is AR-first: every design decision — physically-based rendering, camera grain, depth of field, motion blur — is optimized for making virtual content convincingly coexist with the real world. Combined with ARKit's world-tracking, RealityKit dramatically reduces the code needed to build a complete AR app to just a few lines.

Accompanying RealityKit is Reality Composer, available as a macOS tool (integrated with Xcode 11) and an iPadOS/iOS app. Reality Composer provides a "what you see is what you get" scene editor with AR preview, a built-in content library, behavior authoring (triggers + actions), and code-generation that produces a strongly typed Swift API for accessing scene objects and invoking behaviors directly from Xcode.

This is the introductory overview session. Deeper dives are in Sessions 605 (Building Apps with RealityKit), 609 (Building AR Experiences with Reality Composer), 610 (Collaborative AR), and 607 (People in AR).

## Key Topics

**Entity Component System (ECS)**
RealityKit uses ECS instead of class inheritance hierarchies. Entities are bare containers; components are struct-like objects attached to entities that supply specific behaviors and data (model geometry, physics body, collision shape, anchoring description, spatial audio source, etc.). Benefits: composability, code reuse, cache-friendly memory layout, automatic multithreaded processing, and automatic network synchronization of all components (including custom ones) in multipeer sessions.

**Core Classes**
- `ARView` — the main UIView subclass. Owns the scene, drives the render loop, applies camera effects, handles gesture recognition. Provides AR Quick Look quality out of the box.
- `Scene` — holds the anchor hierarchy. Entities are inactive until their associated real-world anchor is detected by ARKit.
- `AnchorEntity` — the glue between virtual content and real-world surfaces. Supports all ARKit anchor types: horizontal/vertical plane (with classification, e.g. `.table`), image, object, body, face, camera, and direct wrapping of `ARAnchor` or `ARRaycastResult`.
- `Entity` — base class for all virtual objects. Has transform, children list, and a component collection. Can be parented to any other entity.
- `ModelEntity` — the workhorse: an `Entity` pre-configured with a `ModelComponent` (mesh + materials), `CollisionComponent`, and `PhysicsBodyComponent`.

**ARView Camera Effects**
Four camera effects are applied automatically to all virtual content to match the live camera feed:
1. Grounding shadows — either a simple drop shadow or a high-quality contact shadow.
2. Camera-based motion blur — driven by ARKit's exposure time data.
3. Depth of field — driven by ARKit's focus distance.
4. Camera grain — driven by ARKit's `cameraGrainIntensity` to match sensor noise in low light.

**Materials**
- `SimpleMaterial` — physically-based (baseColor, roughness, metallic) — scalar or texture inputs. Simulates conductive vs. non-conductive surfaces.
- `UnlitMaterial` — flat color, unaffected by scene lighting; suitable for screens or bright labels.
- `OcclusionMaterial` — masks virtual content behind it, falling back to camera passthrough; creates the illusion of content emerging from or disappearing behind real-world objects.

**Mesh Resources**
Loadable from USDZ or `.reality` files. Also generatable procedurally from primitives: `.generateBox()`, `.generateSphere()`, `.generatePlane()`, `.generateText()` (supports all platform fonts). Mesh resources are shareable across entities — RealityKit automatically batches draw calls for entities sharing the same mesh.

**Animation**
- Skeletal animations and transform animations loaded from USDZ/Reality Files.
- Procedural animation via ARKit body motion capture (drives `BodyTrackedEntity`).
- `Entity.playAnimation(_:transitionDuration:startsPaused:)` returns an `AnimationPlaybackController` for pause/resume/stop.
- `Entity.move(to:relativeTo:duration:timingFunction:)` — declarative transform animation.

**Physics**
Collision shapes: box, sphere, capsule, compound. Rigid body dynamics: mass, inertia, friction, restitution coefficients. Physics interactions are simulated between virtual objects and can interact with real-world plane anchors used as collision surfaces.

**Networking / Multipeer**
RealityKit's synchronization system is built on MultipeerConnectivity. It automatically synchronizes the entire entity hierarchy, all component data (including custom components), and shared representations of real-world anchor data across all peers. No custom serialization required.

**Reality File Format**
A new `.reality` binary format storing pre-optimized meshes, materials, physics, audio, and animation — faster to load than USDZ and providing more content types. Exported from Reality Composer or future DCC tools.

**Reality Composer**
- Scene editor with drag-and-drop from a built-in content library (shapes, objects, buildings with procedural parameters).
- AR preview on device (lay out content at real-world scale).
- Behavior authoring: Trigger (e.g., tap, proximity, scene start) → Sequence of Actions (move, rotate, jiggle, play animation, play audio, etc.).
- Xcode integration: `.rcproject` files in the Xcode project are compiled to `.reality` at build time, and Xcode auto-generates a strongly typed Swift file exposing scene objects and behavior triggers by name. Mismatches between code and scene cause compile errors, not runtime crashes.

## APIs & Frameworks

**RealityKit** (iOS 13, iPadOS 13) **[NEW]**

Views and scene:
- `ARView: UIView` **[NEW]** — entry point; `.scene: Scene`
- `Scene` **[NEW]** — `.addAnchor(_:)`, `.removeAnchor(_:)`, `.anchors`

Anchoring:
- `AnchorEntity` **[NEW]** — wraps ARKit anchoring; initializers for plane (with classification + min bounds), image, object, body, face, camera, `ARAnchor`, `ARRaycastResult`
- `AnchoringComponent` **[NEW]** — component form of anchor description

Entities:
- `Entity` **[NEW]** — `.addChild(_:)`, `.children`, `.position`, `.transform`, `.setPosition(_:relativeTo:)`, `components[T.self]`
- `ModelEntity` **[NEW]** — `Entity` + `ModelComponent` + `CollisionComponent` + `PhysicsBodyComponent`
- `BodyTrackedEntity` **[NEW]** — entity driven by ARKit body tracking joints

Mesh resources:
- `MeshResource` **[NEW]**
  - `MeshResource.generateBox(size:)` **[NEW]**
  - `MeshResource.generateSphere(radius:)` **[NEW]**
  - `MeshResource.generatePlane(width:depth:)` **[NEW]**
  - `MeshResource.generateText(_:extrusionDepth:font:containerFrame:alignment:lineBreakMode:)` **[NEW]**
  - `Entity.loadModel(named:)` / `Entity.loadModelAsync(named:)` **[NEW]**

Materials:
- `SimpleMaterial(color:roughness:isMetallic:)` **[NEW]**
- `SimpleMaterial.baseColor: MaterialColorParameter` **[NEW]**
- `SimpleMaterial.roughness: MaterialScalarParameter` **[NEW]**
- `SimpleMaterial.metallic: MaterialScalarParameter` **[NEW]**
- `UnlitMaterial(color:)` **[NEW]**
- `OcclusionMaterial()` **[NEW]**

Animation:
- `Entity.playAnimation(_:transitionDuration:startsPaused:) -> AnimationPlaybackController` **[NEW]**
- `AnimationPlaybackController.pause()` / `.resume()` / `.stop()` **[NEW]**
- `Entity.move(to:relativeTo:duration:timingFunction:)` **[NEW]**

Physics and collision:
- `PhysicsBodyComponent` **[NEW]** — mode: `.dynamic`, `.kinematic`, `.static`
- `CollisionComponent` **[NEW]** — shapes: `ShapeResource.generateBox(size:)`, `.generateSphere(radius:)`, `.generateCapsule(height:radius:)`
- `PhysicsBodyComponent.massProperties` **[NEW]**

Networking:
- `SynchronizationComponent` **[NEW]** — auto-added to all entities; drives multipeer sync
- Custom components conforming to `Component` protocol auto-synchronize **[NEW]**

Audio:
- `SpatialAudioComponent` **[NEW]** — attaches audio source to entity; distance-based attenuation

Reality File:
- `.reality` file type **[NEW]** — pre-optimized binary; exported from Reality Composer

## Code Highlights

Minimal four-line AR app (place a model on a horizontal plane):
```swift
let arView = ARView(frame: .zero)
let anchor = AnchorEntity(plane: .horizontal)
let model = try! Entity.loadModel(named: "flyer")
anchor.addChild(model)
arView.scene.addAnchor(anchor)
```

Anchor to a table-sized horizontal surface:
```swift
let tableAnchor = AnchorEntity(
    plane: .horizontal,
    classification: .table,
    minimumBounds: [0.5, 0.5]
)
arView.scene.addAnchor(tableAnchor)
```

Generating a procedural box with a SimpleMaterial:
```swift
let mesh = MeshResource.generateBox(size: [0.1, 0.1, 0.1])
var material = SimpleMaterial()
material.baseColor = .color(.red)
material.roughness = 0.5
material.metallic = 0.0
let entity = ModelEntity(mesh: mesh, materials: [material])
```

Animating an entity with move(to:):
```swift
entity.move(
    to: Transform(translation: [0, 0, -5]),
    relativeTo: nil, // world space
    duration: 2.0,
    timingFunction: .easeInOut
)
```

## Takeaways
- RealityKit's entity component system combined with its AR-first camera effects (motion blur, depth of field, grain, contact shadows) dramatically reduces the code and expertise needed to place convincing virtual content in the real world.
- `OcclusionMaterial` on planes anchored to real surfaces creates the illusion of virtual objects emerging from or disappearing under real objects — a critical realism technique that is trivial to implement in RealityKit.
- Reality Composer's Xcode code generation turns scene authoring into a compile-time-safe API: misspelled entity names become build errors rather than silent nil crashes at runtime.
- RealityKit's networking layer synchronizes the entire entity hierarchy including custom components automatically over MultipeerConnectivity with no serialization code.

---
_Source: WWDC19 Session 603 page (transcript, chapter summaries, and resource links)._
