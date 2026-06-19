# Going Beyond 2D with SpriteKit
**WWDC17 · Session 609** · [Watch](https://developer.apple.com/videos/play/wwdc2017/609/)

_Platforms:_ iOS 11, macOS High Sierra 10.13, tvOS 11, watchOS 4

## Overview
SpriteKit has traditionally been Apple's go-to 2D game framework, but iOS 11 breaks it out of the 2D mold entirely. This session demonstrates three major integration paths: using SpriteKit content inside ARKit for augmented-reality apps, embedding SpriteKit scenes as textures on SceneKit geometry for mixed 2D/3D rendering, and using the new `SKRenderer` to take direct control of how and when SpriteKit updates and renders into a Metal command buffer.

ARKit handles all device tracking, plane detection, and anchor management automatically, letting developers focus entirely on creating SpriteKit content rather than the hard parts of AR. Sprites appear in the real world via billboarding — they always face the camera — enabling classic 2D content to feel naturally positioned in 3D space. Tapping the screen fires a hit-test ray through ARSKView, finds the nearest real-world feature point, and creates an ARAnchor for content placement.

The session concludes with `SKRenderer`, a new class that replaces `SKView` for scenarios where you need precise control over update/render timing, want to render into an off-screen Metal texture, or need to mix SpriteKit content directly with custom Metal rendering in the same command encoder.

## Key Topics
- **SpriteKit + ARKit integration** — ARSKView, ARAnchor, ARSKViewDelegate; billboarding sprites; hit-testing for anchor creation; Xcode 9 AR app template
- **ARSKViewDelegate event cycle** — `node(for:)`, `didAdd(_:for:)`, `willUpdate(_:for:)`, `didUpdate(_:for:)`, `didRemove(_:for:)`
- **SpriteKit as SceneKit material** — setting an SKScene as the `diffuse.contents` of a SceneKit material, enabling full 3D transforms and perspective on 2D content
- **Mixing 2D and 3D in ARKit** — using ARSCNView + SceneKit for perspective-correct placement of SpriteKit scene layers
- **SKRenderer** — low-level control over update/render timing; rendering into off-screen Metal textures; encoding into an existing MTLRenderCommandEncoder for efficient mixing
- **New SpriteKit features** — `SKLabelNode` attributed text support (`attributedText` property); `SKTransformNode` for full 3D rotations (x/y/z axes, Euler angles, quaternions, rotation matrices)
- **Xcode View Debugger** — SpriteKit scene graph visible in exploded 3D view; node properties inspectable at pause time

## APIs & Frameworks

### ARKit (New in iOS 11)
- **`ARSession`** **[NEW]** — core session; `run(_:)` with `ARWorldTrackingSessionConfiguration`
- **`ARWorldTrackingSessionConfiguration`** **[NEW]** — enables full 6DOF world tracking
- **`ARAnchor`** **[NEW]** — 3D point in the real world with a `transform` and unique identifier
- **`ARSKView`** **[NEW]** — `SKView` subclass that owns the `ARSession`; `hitTest(_:types:)` for anchor creation; `node(for:)` / `anchor(for:)` helpers
- **`ARSKViewDelegate`** **[NEW]** — protocol extending `SKViewDelegate`; methods: `view(_:nodeFor:)`, `view(_:didAdd:for:)`, `view(_:willUpdate:for:)`, `view(_:didUpdate:for:)`, `view(_:didRemove:for:)`
- **`ARSCNView`** / **`ARSCNViewDelegate`** **[NEW]** — SceneKit equivalents for the ARKit integration

### SpriteKit
- **`SKRenderer`** **[NEW]** — `init(device:)`; `scene` property; `update(atTime:)`; `render(withViewport:commandBuffer:renderPassDescriptor:)`; `render(withViewport:renderCommandEncoder:renderPassDescriptor:projectionMatrix:)`
- **`SKTransformNode`** **[NEW]** — subclass of `SKNode`; adds `xRotation`, `yRotation` in addition to `zRotation`; setters for Euler angles, rotation matrices, and quaternions; orthographic projection (no perspective skew)
- **`SKLabelNode.attributedText`** **[NEW]** — `NSAttributedString` property enabling per-character color, font, and other attributes in a single label
- **`SKScene`** — set as `SCNMaterial.diffuse.contents` to texture-map onto SceneKit geometry
- **`SKNode`** — children of anchor-mapped nodes are not modified by ARKit; Z position ≥ 0 draws over AR content
- **`SKView`** — standard rendering path; View Debugger now supported

### Metal
- **`MTLDevice`** — passed to `SKRenderer(device:)` for initialization
- **`MTLCommandBuffer`** — used with `SKRenderer.render(withViewport:commandBuffer:renderPassDescriptor:)`
- **`MTLRenderCommandEncoder`** — used with `SKRenderer.render(withViewport:renderCommandEncoder:renderPassDescriptor:projectionMatrix:)` for in-place mixing

### SceneKit
- **`SCNMaterial.diffuse.contents`** — set to an `SKScene` instance to render SpriteKit as a live texture
- **`SCNPlane`**, **`SCNBox`**, **`SCNSphere`** — example geometries that accept SpriteKit scene materials

## Code Highlights

Creating an ARAnchor from a tap:
```swift
func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
    let location = touches.first!.location(in: sceneView)
    let hits = sceneView.hitTest(location, types: .featurePoint)
    if let hit = hits.first {
        let anchor = ARAnchor(transform: hit.worldTransform)
        sceneView.session.add(anchor: anchor)
    }
}
```

Attaching SpriteKit content to an anchor:
```swift
func view(_ view: ARSKView, didAdd node: SKNode, for anchor: ARAnchor) {
    let label = SKLabelNode(text: "Hello AR")
    node.addChild(label)
}
```

Using SpriteKit as a SceneKit material:
```swift
let spriteScene = SKScene(size: CGSize(width: 512, height: 512))
let plane = SCNPlane(width: 1.0, height: 1.0)
plane.firstMaterial?.diffuse.contents = spriteScene
plane.firstMaterial?.isDoubleSided = true
let planeNode = SCNNode(geometry: plane)
scnScene.rootNode.addChildNode(planeNode)
```

Initializing and driving SKRenderer:
```swift
let renderer = SKRenderer(device: MTLCreateSystemDefaultDevice()!)
renderer.scene = myScene
// Each frame:
renderer.update(atTime: currentTime)
renderer.render(withViewport: viewport, commandBuffer: commandBuffer,
                renderPassDescriptor: renderPassDescriptor)
```

## Takeaways
- ARKit + SpriteKit together make AR app creation extremely low-friction; the Xcode 9 template generates a working AR app in seconds, and ARSKView handles all positioning automatically.
- `SKRenderer` unlocks use cases impossible with `SKView`: fixed-timestep updates, off-screen Metal texture rendering, and mixing SpriteKit with custom Metal rendering in the same encoder.
- SpriteKit scenes can be used as live, interactive textures on SceneKit geometry, enabling full perspective-correct 3D rendering of 2D content within both ARKit and regular 3D scenes.
- New `SKTransformNode` and `SKLabelNode.attributedText` significantly expand what is expressible with SpriteKit alone before touching the 3D integration points.

---
_Source: WWDC17 Session 609 page (abstract, chapter summaries, code samples, and resource links)._
