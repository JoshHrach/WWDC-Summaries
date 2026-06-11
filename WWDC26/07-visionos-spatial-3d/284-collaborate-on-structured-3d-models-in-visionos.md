# Collaborate on structured 3D models in visionOS
**WWDC26 · Session 284** · [Watch](https://developer.apple.com/videos/play/wwdc2026/284/)

_Platforms:_ visionOS 27

## Overview
This session covers building collaborative 3D model review and manipulation experiences on Apple Vision Pro. Using a complex mechanical assembly as the running example, it walks through three interlocking techniques: manipulating entity hierarchies (opening and closing assemblies so individual parts can be grabbed independently), interactive cross-sectional clipping (a new `ClippingComponent` that lets viewers see inside assemblies), and auto-expansion (an algorithm that animates parts outward to create an exploded-view diagram on demand).

The session begins with asset preparation guidance, emphasizing that USDZ files must preserve a deep, nested hierarchy with meaningful prim names so that individual components remain independently selectable and addressable at runtime. It then builds up each interaction technique with full Swift code patterns, including the `ManipulationComponent` API for making entities draggable and the `releaseBehavior` setting that determines what happens when a part is released.

The clipping section is the deepest technical segment, covering the three-state clipping machine (`.off`, `.on`, `.editing`), coordinate frame math for updating clipping plane bounds from drag gestures, and how to handle the boundary between what is clipped and what remains visible. The auto-expansion section explains how to determine the best expansion axis using volume-weighted variance and assemble `FromToBy` animations to translate parts into exploded positions.

## Key Topics

### Asset Preparation
- Export USDZ from DCC tools with hierarchy fully preserved (no flattening, no merge-all-meshes).
- Meaningful prim names in the USD hierarchy enable runtime entity lookup by name.
- Each independently selectable sub-component should be a separate prim/entity with its own transform.

### Manipulating the Hierarchy
- `ManipulationComponent` makes an entity draggable and rotatable in space; `releaseBehavior = .stay` keeps the part where the user places it.
- `InputTargetComponent` designates an entity as a touch/hand interaction target.
- `openAssembly()`: remove `ManipulationComponent` and `InputTargetComponent` from root; set them on each child.
- `closeAssembly()`: remove from children; restore on root.

### Interactive Clipping (ClippingComponent — NEW)
- `ClippingComponent` **[NEW]** defines a planar cross-section through an assembly.
- Three states: `.off` (no clipping), `.on` (clipping active, plane not interactive), `.editing` (drag handle visible).
- Plane position is expressed in the entity's local coordinate frame; drag gestures are converted via coordinate frame transforms.
- Works at any depth in the entity hierarchy — children of a clipped entity inherit clipping automatically.

### Autoexpansion
- Choose the expansion axis by computing the volume-weighted variance of each part's centroid along each world axis; the axis with highest variance is the most meaningful to explode along.
- Compute per-part offsets along the chosen axis, scaled so the largest gap is a fixed clearance distance.
- Animate using `FromToByAnimation` with a `Transform` target — move each child from its current position to its exploded position.

## APIs & Frameworks

### RealityKit
- `ManipulationComponent` — `releaseBehavior: ManipulationComponent.ReleaseBehavior` (.stay, .returnToStart) **[NEW or updated]**
- `InputTargetComponent` — existing
- `ClippingComponent` **[NEW]**: cross-sectional plane clipping for entity hierarchies
  - `.off`, `.on`, `.editing` states **[NEW]**
- `Entity.position(relativeTo:)` — coordinate frame conversions
- `FromToByAnimation<Transform>` — existing; used for exploded-view moves
- `AnimationPlaybackController` — existing
- `ModelEntity`, `Entity`, `Entity.findEntity(named:)`, `Entity.children` — existing
- `Entity.components.set(_:)`, `Entity.components[T.self]` — existing

### SwiftUI + RealityKit Gestures
- `DragGesture` → `onChanged` / `onEnded` to drive clipping plane position updates
- `TapGesture` for assembly open/close toggle

### RealityKit Sample
- [Manipulating models with RealityKit](https://developer.apple.com/documentation/RealityKit/manipulating-models-with-realitykit) — sample project

## Code Highlights

Open an assembly for per-part manipulation:
```swift
func openAssembly() {
    components[ManipulationComponent.self] = nil
    components[InputTargetComponent.self] = nil
    for child in assemblyChildren {
        child.components.set(InputTargetComponent())
        var manipulation = ManipulationComponent()
        manipulation.releaseBehavior = .stay
        child.manipulationComponent = manipulation
    }
}
```

Close it back to whole-assembly manipulation:
```swift
func closeAssembly() {
    for child in assemblyChildren {
        child.manipulationComponent = nil
        child.components[InputTargetComponent.self] = nil
    }
    components.set(InputTargetComponent())
    var manipulation = ManipulationComponent()
    manipulation.releaseBehavior = .stay
    manipulationComponent = manipulation
}
```

## Takeaways
- Preserving hierarchy in USDZ export is a prerequisite for all per-part manipulation; exporting with flattened geometry makes runtime interaction impossible.
- `ClippingComponent` unlocks a powerful new interaction paradigm — the ability to inspect the interior of complex assemblies — with just a single component addition.
- Auto-expansion via volume-weighted variance is a principled, content-agnostic algorithm that produces natural exploded views without per-asset manual configuration.
- The open/close assembly pattern (swapping `ManipulationComponent` and `InputTargetComponent` between root and children) is broadly reusable for any product-review or collaboration use case.

---
_Source: WWDC26 Session 284 page (abstract, chapter summaries, code samples, and resource links)._
