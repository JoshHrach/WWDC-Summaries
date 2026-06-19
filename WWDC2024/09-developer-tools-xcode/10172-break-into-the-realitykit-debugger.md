# Break into the RealityKit Debugger
**WWDC24 · Session 10172** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10172/)

_Platforms:_ visionOS (RealityKit Debugger runs in Xcode 16 on macOS; targets visionOS Simulator and device)

## Overview
This session introduces the RealityKit Debugger, a new tool in Xcode 16 that captures a 3D snapshot of a running spatial app and lets you inspect the entity hierarchy, component properties, and scene rendering state—all inside Xcode. The tool is demonstrated using an extension of the BOTanist sample app transformed into a virtual club.

Four major bug categories are addressed: rogue transformations (inherited transforms causing entity distortion), misconfigured ECS components (broken systems because components were never written back to entities), rendering pitfalls (entities invisible due to bad transforms, missing anchors, clipped bounds, opacity issues, bad mesh normals), and complex system debugging using custom visualization entities and debug-build components.

The session closes with a practical pattern: leverage RealityKit's Entity Component System (ECS) to build debug visualizations and custom inspector data directly in your app code, behind `#if DEBUG` compilation blocks, so they disappear from production builds.

## Key Topics
- **RealityKit Debugger overview** — "Capture Entity Hierarchy" button in Xcode debug area; shows scene list, entity hierarchy outline, 3D viewport, and entity inspector
- **Transform hierarchy bugs** — entity final placement is cumulative from all ancestors; secondary viewport in inspector shows mesh without inherited transforms
- **ECS component debugging** — inspecting component properties at runtime; the common mistake of modifying a component value without writing it back to the entity
- **Rendering pitfalls** — occluded/clipped/inside-out/disabled entities; Anchoring components without matching ARKit anchors; missing ModelComponent; misconfigured opacity threshold; bad mesh normals; out-of-scene-bounds entities
- **Custom debug visualizations** — add model entities and custom components in `#if DEBUG` blocks; RealityKit Debugger can display most Swift types including `UIImage` (for SwiftUI charts)
- **Entity links** — custom debug components can store entity references that render as clickable links in the inspector

## APIs & Frameworks
### RealityKit (Xcode 16 / visionOS 2)
- **[NEW] RealityKit Debugger** — integrated in Xcode 16; "Capture Entity Hierarchy" in debug bar; entity hierarchy outline, 3D viewport, component inspector, statistics inspector
- `Entity` — base class for all scene objects; hierarchy traversal in debugger
- `ModelComponent` — mesh + materials on an entity; visible in inspector preview viewport
- `Transform` — position, rotation, scale; inherited from ancestors in world space
- `HoverEffectComponent` — `.highlight(color:strength:)` (new), `.shader(.default)` (new); previously only spotlight style
- `AnchoringComponent` — binds entity to ARKit anchor; entity invisible if anchor not found in scene
- Custom `Component` types — show their stored properties in the RealityKit Debugger inspector (most Swift types supported, including `UIImage`)
- **`#if DEBUG` compilation blocks** — recommended pattern for debug visualization code; excluded from release builds

### Entity Component System (ECS) Pattern
- Modifying a component value requires assigning it back: `entity.components[MyComponent.self] = updatedComponent`
- Systems query for entities by component set; missing/misconfigured components cause silent system failures

### Reality Composer Pro
- Recommended for preparing, testing, and packaging assets to avoid rendering pitfalls before they reach the debugger

## Code Highlights
```swift
// Common ECS bug: component modification not written back
func update(context: SceneUpdateContext) {
    for entity in context.scene.performQuery(ControlCenterComponent.query) {
        var controlCenter = entity.components[ControlCenterComponent.self]!
        controlCenter.countdown -= context.deltaTime
        // BUG: change is lost without this line:
        entity.components[ControlCenterComponent.self] = controlCenter
    }
}

// Custom debug component with entity reference (shows as link in inspector)
struct AttractorDebugComponent: Component {
    var state: AttractorState
    var targetRobot: Entity?   // RealityKit Debugger renders this as a clickable link
}

// Debug visualization behind compilation block
#if DEBUG
func setupDebugVisualization(in scene: Scene) {
    let debugParent = Entity()
    debugParent.isEnabled = false  // invisible in play mode
    for attractor in attractors {
        let vis = ModelEntity(mesh: .generateSphere(radius: 0.05),
                              materials: [SimpleMaterial(color: .orange, isMetallic: false)])
        vis.components.set(AttractorDebugComponent(state: .attracting))
        debugParent.addChild(vis)
    }
    scene.addAnchor(AnchorEntity(world: .zero))
}
#endif
```

## Takeaways
- The RealityKit Debugger's 3D viewport + inspector makes it fast to confirm whether invisible content is missing from the hierarchy, disabled, misconfigured, or a rendering artifact
- Always assign modified components back to their entity—the most common silent ECS bug
- Build custom debug components and model visualizations behind `#if DEBUG` to add first-class debuggability to your ECS systems without impacting shipping performance
- Inherited transform bugs are easy to miss in code; the secondary inspector viewport (which shows the mesh without ancestor transforms) makes them immediately visible

---
_Source: WWDC24 Session 10172 page (abstract, chapter summaries, code samples, and resource links)._
