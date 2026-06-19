# Supercharge your spatial workflows with Reality Composer Pro 3
**WWDC26 · Session 393** · [Watch](https://developer.apple.com/videos/play/wwdc2026/393/)

_Platforms:_ visionOS 27, macOS 27

## Overview
This session is a deep tour of the visual, node-based authoring tools in Reality Composer Pro 3 (RCP3) that enable rich interactivity and complex visual effects without leaving the editor. The session covers six major graph-based tools — Animation Graph, Behavior Tree, Script Graph, Navigation Mesh, Compute Graph, and Shader Graph enhancements — and shows how they compose together to build a complete, interactive spatial scene (the "Alchemist's Lab") from within RCP3 alone.

The session is aimed at developers and artists building visionOS apps or games who want to understand what each tool does and when to use it. It positions these tools as complementary layers: Behavior Trees drive NPC logic, Animation Graph blends character animations, Script Graph wires up events, Navigation Mesh handles pathfinding, Compute Graph runs GPU particle effects, and Shader Graph controls appearance.

Shader Graph enhancements are highlighted at the end: `RealityKit PBR Surface 2` adds sheen and subsurface scattering for more realistic organic materials; `Hair Surface` provides physically-based strand lighting; and portal-rendering support is now available directly in Shader Graph.

## Key Topics

### Animation Graph
- Visual state machine for blending skeletal animations at runtime.
- States (e.g., Idle, Walk) contain `Animation Clip` nodes; transitions are triggered by runtime parameters.
- Parameters (floats, booleans) can be set from Script Graph or from Swift code at runtime.
- Produces smooth blended character movement without per-frame code.

### Behavior Tree
- Visual tree for authoring autonomous NPC routines.
- Composite nodes: `Sequence` (all children must succeed), `Selector` (first succeeding child wins), `Parallel` (all children run simultaneously).
- Built-in Action nodes: `Move To`, `Rotate To Face`, `Wait`, `Parameter Setter`.
- Compose reusable subtrees for patrol, react, and investigate behaviors.

### Script Graph
- Event-driven visual scripting; connects to Behavior Trees via parameter nodes.
- Key nodes: `On Initialize`, `On Tap`, Subgraph references.
- Live Preview on Apple Vision Pro (coming later 2026): iterate behavior in headset without rebuilding.
- Fires custom events that Behavior Trees and Animation Graph can listen to.

### Navigation Mesh
- Bakes a walkable surface mesh from scene geometry directly in RCP3.
- Configure: bounding box for mesh generation region, off-mesh connections (ladders, jumps), cell size for accuracy.
- At runtime, entities with `NavigationComponent` and a `NavigationController` can pathfind automatically.
- Pairs with Behavior Tree `Move To` action node for fully visual NPC movement authoring.

### Compute Graph
- GPU-driven particle simulation backed by Metal, authored visually in RCP3.
- Four phases: `Emitter` (spawn rate, shape), `Initialize` (set initial position/velocity/color/lifetime), `Simulate` (per-tick forces, turbulence, color over lifetime), `Output` (mesh + Shader Graph material).
- Example: cauldron smoke with custom spawn sphere, upward velocity, color shift from white to grey.
- Replaces CPU particle systems; scales to millions of particles.

### Shader Graph Enhancements
- `RealityKit PBR Surface 2` **[NEW]**: adds sheen parameter (fabric-like highlight) and subsurface scattering (wax/skin translucency).
- `Hair Surface` shader **[NEW]**: physically-based strand-level light scattering for hair and fur.
- Portal rendering support in Shader Graph **[NEW]**: author portal-capable materials without code.

## APIs & Frameworks

### Reality Composer Pro 3 Visual Tools (all NEW)
- **Animation Graph**: State Machine, Animation Clip nodes, transition conditions, runtime parameters
- **Behavior Tree**: Sequence, Selector, Parallel composites; Move To, Rotate To Face, Wait, Parameter Setter actions
- **Script Graph**: On Initialize, On Tap, Subgraph, custom event nodes; Live Preview support
- **Navigation Mesh Component**: bounding box, off-mesh connections, cell size, agent radius
- **Compute Graph**: Emitter, Initialize, Simulate, Output phases; Metal-backed GPU simulation
- **Shader Graph — RealityKit PBR Surface 2** **[NEW]**: sheen, subsurface scattering
- **Shader Graph — Hair Surface** **[NEW]**
- **Shader Graph — Portal rendering** **[NEW]**

### RealityKit (runtime counterparts)
- `NavigationComponent` **[NEW]**: attach to NPC entity
- `NavigationController(entity:)` **[NEW]**: `computePath(from:to:)` async
- `NavigationMeshResource` **[NEW]**: generated from RCP3 or runtime
- `AnimationResource` / `AnimationPlaybackController` — existing, driven by Animation Graph output
- `ShaderGraphMaterial.setParameter(name:value:)` — runtime override of Compute/Shader Graph parameters

### RCP3 Plugin System (for Xcode integration)
- `RealityComposerProPlugin` protocol — `setup(context:)` registers components, systems, actions
- `RealityComposerProContext.registerComponent/registerSystem/registerAction`
- See Session 281 for full plugin authoring details

## Code Highlights

_Note: The primary authoring for these tools is visual (node graph). Runtime Swift code interacts via parameter setting and component APIs from Session 279/281. No dedicated code samples appear in this session's page._

Set a Behavior Tree parameter from Swift (to trigger an animation state change):
```swift
// Set via ManipulationComponent or custom component parameter
entity.components[BehaviorTreeComponent.self]?.setParameter("isWalking", value: true)
```

## Takeaways
- Animation Graph + Behavior Tree + Navigation Mesh together provide a complete, no-code pipeline for NPCs: the Behavior Tree issues `Move To` commands, Navigation Mesh pathfinds, and Animation Graph blends between Idle and Walk states.
- Compute Graph brings Metal-level particle simulation to artists with no shader code — a significant leap beyond CPU particle systems.
- `RealityKit PBR Surface 2` with sheen and subsurface scattering closes a major visual gap for organic materials like fabric, skin, and wax in spatial experiences.
- All of these tools are additive: a scene can use any combination of them, and they communicate through shared runtime parameters.

---
_Source: WWDC26 Session 393 page (abstract, chapter summaries, and resource links)._
