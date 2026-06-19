# SceneKit in Swift Playgrounds
**WWDC17 · Session 605** · [Watch](https://developer.apple.com/videos/play/wwdc2017/605/)

_Platforms:_ iOS 11, macOS High Sierra 10.13

## Overview
The Swift Playgrounds Content team shares lessons learned building the Learn to Code 3D world using SceneKit — progressing from a 2D SpriteKit prototype through asset iteration to a fully optimized, accessible 3D experience. The session is structured around three phases: prototyping (keep fidelity low, test interaction models quickly), iterating (establish data-from-visuals separation, handle complex animations, support VoiceOver), and tuning (reduce draw calls through geometry flattening, texture atlasing, and light maps).

The performance optimization section is particularly concrete: the team took a scene with 877 draw calls and 29 fps down to 73 draw calls and 60 fps (2.3 ms render time) through three targeted techniques. These gains unlocked headroom for gameplay logic, networking, and richer geometry without sacrificing frame rate — including compatibility with 120 Hz ProMotion displays.

Accessibility in 3D is shown to be simpler than expected: the same `UIAccessibilityElement` technique used in UIKit works in SceneKit by converting 3D positions to 2D screen coordinates with `SCNView.projectPoint(_:)`.

## Key Topics
- **Prototyping phase** — start with placeholder assets (even emoji); test interaction model, not visuals; use SceneKit primitives for new mechanics; be willing to throw everything away; keep fidelity low so changes are cheap
- **Data / visuals separation** — build the world model independently of SceneKit nodes; allows asset swaps without rebuilding levels, network transmission of game state, and geometry optimization; enabled by a code-based world-construction API
- **Reference nodes (`SCNReferenceNode`)** — link SceneKit nodes to external `.scn` files; update one file to propagate changes across all scenes; prevents duplicated effort when an asset is updated
- **Baked-in displacement animations** — for complex movements like stair climbing, embed displacement in the animation rather than translating the node; use `SCNTransaction` to atomically synchronize node position and remove the animation in one frame
- **Geometry shader modifiers** — write custom GLSL geometry shaders to animate texture coordinates (moving water, swaying vines, blowing grass); accessible via the shader modifier tray in Xcode 9 Scene Editor
- **VoiceOver in 3D** — use `UIAccessibilityElement` with `SCNView.projectPoint(_:)` to convert 3D positions to 2D screen frame; add elements to the view; same pattern as UIKit; add audio richness (music, character sounds) as non-visual feedback
- **Geometry flattening (`SCNNode.flattenedClone()`)** — combine multiple static, same-material meshes into one; reduces draw calls dramatically (877 → <16 for flattened sections); caveat: do not flatten dynamic nodes or nodes with shader modifiers; group nodes by movement identity
- **Texture atlasing** — combine many materials into one atlas texture; reduces draw calls (one material = one shader per mesh); reduces shader compilation count at load time; reduces disk I/O
- **Light maps** — precompute static lighting as a baked material/texture; essentially free at runtime (no GPU cost); reduces dynamic light count from many to 1 (spotlight for moving character shadow); works multiplicatively with geometry flattening
- **Debug Statistics View** — `SCNView.showStatistics = true`; shows fps, render time (ms), draw call count (diamond indicator)
- **Performance targets** — 60 fps requires <16 ms render time; 120 Hz ProMotion requires <7 ms; team achieved 2.3 ms

## APIs & Frameworks

### SceneKit
- **`SCNNode`** — scene graph node; position, rotation, scale, geometry, actions, animations
- **`SCNNode.flattenedClone()`** — returns a new node with all descendant geometries merged; reduces draw calls for static geometry
- **`SCNReferenceNode`** — references an external `.scn` file; live updates propagate automatically
- **`SCNTransaction`** — `SCNTransaction.begin()` / `SCNTransaction.commit()`; `SCNTransaction.animationDuration = 0` for synchronous single-frame updates
- **`SCNView.showStatistics`** — `Bool`; enables debug statistics overlay (fps, render time, draw calls)
- **`SCNView.projectPoint(_:)`** — converts 3D `SCNVector3` to 2D screen `SCNVector3`; used for VoiceOver accessibility element frame computation
- **`SCNScene`** — root scene; loaded from `.scn` or `.scnassets`
- **`SCNGeometry`** — mesh data; `.materials` array
- **`SCNMaterial`** — surface properties; `.diffuse`, `.normal`, `.emission`, `.lightingModel`; replace many materials with one atlas material
- **`SCNLightingModelLightMap`** — lighting model that uses a baked light map texture; runtime cost ~zero
- **`SCNLight`** — scene lights; `.spot`, `.omni`, `.ambient`; each light adds draw call overhead; replace static lights with light maps
- **Shader modifiers** — `SCNShadable.shaderModifiers` dictionary; `SCNShaderModifierEntryPoint.geometry` for vertex-level displacement; new shader tray in Xcode 9 Scene Editor
- **`SCNAction`** — `moveBy`, `rotateTo`, `sequence`, `group`; built-in animation chaining
- **`SCNConstraint`** — `SCNIKConstraint` for inverse kinematics (considered but not used due to personality limitations)

### UIKit (Accessibility)
- **`UIAccessibilityElement`** — custom accessibility element; `label`, `accessibilityFrame`; add to `SCNView.accessibilityElements`
- **`UIAccessibilityElement.accessibilityFrame`** — set using `SCNView.projectPoint(_:)` to convert 3D world position to 2D screen coordinates

## Code Highlights

```swift
// SCNTransaction for synchronous node position/animation sync (stair animation)
SCNTransaction.begin()
SCNTransaction.animationDuration = 0
characterNode.position = newPosition
characterNode.removeAllAnimations()
SCNTransaction.commit()

// Geometry flattening (parent node contains all static grass tiles)
let flattenedGrass = grassParentNode.flattenedClone()
scene.rootNode.addChildNode(flattenedGrass)
grassParentNode.removeFromParentNode()

// Enable debug statistics
sceneView.showStatistics = true

// VoiceOver accessibility element for a 3D position
let element = UIAccessibilityElement(accessibilityContainer: sceneView)
let worldPos = SCNVector3(node.worldPosition)
let screenPos = sceneView.projectPoint(worldPos)
element.accessibilityFrame = CGRect(x: screenPos.x - 22, y: screenPos.y - 22,
                                     width: 44, height: 44)
element.accessibilityLabel = "Gem at column \(col), row \(row)"
sceneView.accessibilityElements = [element]
```

## Takeaways
- Separating game data from SceneKit node representation is the single most important architectural decision for a world-building app: it enables asset iteration, level editing, and geometry optimization independently.
- `SCNNode.flattenedClone()` is low-hanging fruit for performance: replacing hundreds of individual static mesh nodes with a single merged node can cut draw calls by over 90%.
- Baked light maps let artists create rich, complex lighting environments at zero runtime GPU cost; pair with geometry flattening and texture atlasing to hit 60+ fps even on complex scenes.
- VoiceOver support in 3D is no harder than in UIKit: `SCNView.projectPoint(_:)` maps any 3D scene position to a 2D screen frame for `UIAccessibilityElement` — plan for it early rather than bolting it on later.

---
_Source: WWDC17 Session 605 page (abstract, chapter summaries, code samples, and resource links)._
