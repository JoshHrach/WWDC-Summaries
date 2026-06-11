# Extend Reality Composer Pro 3 functionality with Xcode
**WWDC26 · Session 281** · [Watch](https://developer.apple.com/videos/play/wwdc2026/281/)

_Platforms:_ visionOS 27, macOS 27

## Overview
This session explains how to extend Reality Composer Pro 3 (RCP3) with custom Swift code through a new plugin system that lets engineers and artists collaborate in a single shared repository. A Swift dynamic library compiled from Xcode is loaded by RCP3 at runtime, giving custom components, systems, and animation actions first-class treatment in the editor's inspector — running live as the artist adjusts values, without any app rebuild or deploy cycle.

The session builds up a complete example: a `Cauldron` component with water-level and vortex properties, a `CauldronSystem` that moves a mesh and drives a `ShaderGraphMaterial` in real time, a custom `SetWaterLevelAction` that animates the water level on the sequencer timeline, and finally the `@Scriptable` macro that exposes the component's properties as nodes in Script Graph for no-code artist workflows.

The plugin architecture is based on a `@_cdecl("createRealityComposerProPlugin")` entry point that returns a `RealityComposerProPlugin` instance. The plugin's `setup(context:)` method registers all custom types with the editor. Because RCP3 and the Xcode project share the same git repository, changes to Swift code are hot-reloaded into the editor without restarting it.

## Key Topics

### Plugin Architecture
- RCP3 and Xcode share a single project via a common git repository.
- Swift code is compiled into a dynamic library; the editor loads and trusts it at runtime.
- Entry point: `@_cdecl("createRealityComposerProPlugin") func createRealityComposerProPlugin() -> UnsafeMutableRawPointer`
- All registrations happen in `RealityComposerProPlugin.setup(context:)`.

### Custom Components and Systems
- Conform component to `Component & Codable`; properties are automatically reflected in RCP3 inspector.
- Conform system to `System`; implement `update(context: SceneUpdateContext)` for per-frame logic.
- `context.entities(matching: query)` iterates matching (entity, component) pairs.
- Register both with `context.registerComponent(T.self)` and `context.registerSystem(T.self)`.
- System runs live in the editor (not just at app runtime) — artists see real-time updates as they edit component values.

### Driving ShaderGraphMaterial from a System
- Access `ModelComponent` → `ShaderGraphMaterial` from the entity.
- Call `mat.setParameter(name:value:)` to push computed values (surface level, vortex coefficients) to the shader every frame.
- Artists control behavior through component inspector sliders; no shader graph editing required for parameter-driven effects.

### Custom Animation Actions
- Conform to `EntityAction & Codable`; implement `animatedValueType` (e.g., `Transform.self`).
- Subscribe to `.updated` event: `SetWaterLevelAction.subscribe(to: .updated) { event in ... }`.
- Interpolate using `event.playbackController.time`, `event.startTime`, `event.duration`.
- Register with `context.registerAction(T.self)` and call `T.subscribe()` in setup.
- Action appears as a clip type on the RCP3 sequencer timeline.

### Custom Script Graph Nodes (`@Scriptable`)
- Apply `@Scriptable` macro (from `RealityKitScriptingMacros`) to a `Component` struct.
- Import `RealityKitScripting` and `RealityKitScriptingMacros`.
- Register a `RKS.Configuration` with a `Module` that includes `ComponentType.SchemaProvider.schema`.
- `RKS.addConfiguration(_:)` must be called on `@MainActor`.
- Generated nodes appear in Script Graph's node palette for artists to wire up without writing Swift.

## APIs & Frameworks

### RealityComposerPro (NEW framework)
- `RealityComposerProPlugin` protocol **[NEW]**: `setup(context: any RealityComposerProContext)`
- `RealityComposerProContext` **[NEW]**:
  - `registerComponent(_ type: Component.Type)`
  - `registerSystem(_ type: System.Type)`
  - `registerAction(_ type: EntityAction.Type)` **[NEW]**
- `@_cdecl("createRealityComposerProPlugin")` entry point convention

### RealityKit
- `Component` protocol + `Codable` — reflected in inspector when registered
- `System` protocol — `init(scene:)`, `update(context: SceneUpdateContext)`
- `EntityComponentQuery` — `context.entities(matching:)` in system update
- `EntityAction` protocol **[NEW]**: `animatedValueType`, `.subscribe(to:)` class method
  - Event: `playbackController.time`, `startTime`, `duration`, `action`, `targetEntity`
- `ShaderGraphMaterial.setParameter(name:value:)` — existing
- `ModelComponent`, `Entity.components`, `Entity.findEntity(named:)` — existing

### RealityKitScripting / RealityKitScriptingMacros (NEW)
- `@Scriptable` macro **[NEW]**: auto-generates Script Graph schema from a `Component`
- `RKS.Configuration(id:)` **[NEW]**: configures a scripting module
  - `.onInitialize { _ in [...] }` — returns `[Module(...)]`
- `Module(name:)` **[NEW]**: groups schema providers
- `ComponentType.SchemaProvider.schema` **[NEW]**: schema from `@Scriptable`-annotated component
- `RKS.addConfiguration(_:)` **[NEW]**: registers module; must be called `@MainActor`

## Code Highlights

Minimal plugin entry point:
```swift
@_cdecl("createRealityComposerProPlugin")
public func createRealityComposerProPlugin() -> UnsafeMutableRawPointer {
    return RCPCustomComponentsPlugin().passRetained()
}
```

Register component, system, and action:
```swift
final class RCPCustomComponentsPlugin: RealityComposerProPlugin {
    public func setup(context: any RealityComposerProContext) {
        context.registerComponent(Cauldron.self)
        context.registerSystem(CauldronSystem.self)
        context.registerAction(SetWaterLevelAction.self)
        SetWaterLevelAction.subscribe()
    }
}
```

Expose a component to Script Graph:
```swift
@Scriptable
public struct Cauldron: Component, Codable {
    public var waterLevel: Float
    public var rotationSpeed: Float
}
```

## Takeaways
- The RCP3 plugin system enables true artist–engineer collaboration in a shared repository: engineers write Swift components and systems that run live inside the editor, and artists see real-time results while adjusting values in the inspector.
- Custom `EntityAction` types let engineers author timeline-animatable behaviors that artists compose on the sequencer — bridging procedural code and keyframe editing.
- The `@Scriptable` macro is a single-line addition that makes any component's properties accessible in Script Graph, extending the no-code workflow to custom data without any additional UI code.
- Driving `ShaderGraphMaterial` parameters from a `System` enables rich shader-based visual effects (water surfaces, fire, vortices) that are fully artist-controllable through component inspector values.

---
_Source: WWDC26 Session 281 page (abstract, chapter summaries, code samples, and resource links)._
