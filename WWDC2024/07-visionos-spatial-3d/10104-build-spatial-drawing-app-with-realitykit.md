# Build a Spatial Drawing App with RealityKit
**WWDC24 · Session 10104** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10104/)

_Platforms:_ visionOS 2

## Overview
This session walks through building a complete spatial drawing app for visionOS using new RealityKit APIs introduced at WWDC24. The app lets users draw 3D brush strokes by pinching their fingers in physical space, with a SwiftUI palette for configuring brush type and color, an ARKit-backed spatial tracking session for hand pose detection, and GPU-accelerated geometry generation via the new `LowLevelMesh` and `LowLevelTexture` APIs.

Two brush types are demonstrated: a solid tube brush whose geometry is updated incrementally on the CPU, and a sparkle particle brush whose geometry is regenerated every frame via a Metal compute shader. The session also builds a splash screen using `MeshResource(extruding:)` to generate 3D text and shapes from `AttributedString` and `SwiftUI.Path`, and `LowLevelTexture` for GPU-generated background textures.

This is one of the most technically deep RealityKit sessions at WWDC24, covering the full custom GPU pipeline path.

## Key Topics
- **`SpatialTrackingSession`** — new visionOS 2 API; replaces raw ARKit for hand tracking in RealityKit apps; required to access `AnchorEntity.transform` for hand joints
- **`AnchorEntity` for hand tracking** — anchor to `.hand(.right, .indexFingerTip)` etc.; position-tracks a hand joint
- **`LowLevelMesh`** — new API; bring custom vertex buffer layouts to RealityKit without format conversion; supports interleaved/non-interleaved layouts, Metal vertex formats, triangle strips, compressed formats, up to 8 UV channels
- **GPU mesh updates** — `LowLevelMesh.replace(bufferIndex:using:)` returns an `MTLBuffer`; compute shader writes vertices directly; minimal overhead
- **`LowLevelTexture`** — new API; GPU-updatable textures; `LowLevelTexture.replace(using:)` returns `MTLTexture` for compute shader write
- **`HoverEffectComponent`** — new `.highlight` and `.shader` types; shader-based hover drives `HoverState` node in ShaderGraph
- **`MeshResource(extruding:)`** — new API; converts `SwiftUI.Path` or `AttributedString` to a 3D mesh; `ShapeExtrusionOptions` for depth, chamfer, material assignment
- **Additive blend mode** — new in `UnlitMaterial` and `PhysicallyBasedMaterial`; `Program.Descriptor.blendMode = .add`
- **Foveated rendering awareness** — avoid thin high-contrast geometry on visionOS; artifacts visible in peripheral vision

## APIs & Frameworks
### RealityKit — visionOS 2 [NEW or CHANGED]
- **[NEW] `SpatialTrackingSession`** — `Configuration(tracking: [.hand])`; `run(_:)` async; returns unapproved capabilities; required for `AnchorEntity.transform` updates
- **[NEW] `LowLevelMesh`** — `Descriptor` (vertexCapacity, indexCapacity, vertexAttributes, vertexLayouts); `withUnsafeMutableBytes(bufferIndex:)` for CPU updates; `replace(bufferIndex:using:)` for GPU compute updates; `parts: [LowLevelMesh.Part]` (indexOffset, indexCount, topology, materialIndex, bounds)
- **[NEW] `LowLevelMesh.Attribute`** — semantic (`.position`, `.normal`, `.bitangent`, `.color`, `.uv0`–`.uv7`), MTLVertexFormat, layoutIndex, offset
- **[NEW] `LowLevelMesh.Layout`** — bufferIndex, bufferOffset, bufferStride
- **[NEW] `LowLevelTexture`** — `Descriptor` (pixelFormat, width, height, textureUsage); `replace(using:)` returns `MTLTexture`; `TextureResource(from: LowLevelTexture)`
- `MeshResource(from: LowLevelMesh)` — create renderable resource from `LowLevelMesh`
- **[NEW] `MeshResource(extruding: SwiftUI.Path, extrusionOptions:)`** — 2D path to 3D mesh
- **[NEW] `MeshResource(extruding: AttributedString, extrusionOptions:)`** — text to 3D mesh
- **[NEW] `MeshResource.ShapeExtrusionOptions`** — `.extrusionMethod`, `.boundaryResolution`, `.materialAssignment`, `.chamferRadius`
- **[NEW] `HoverEffectComponent(.highlight(color:strength:))`** — uniform color highlight on gaze
- **[NEW] `HoverEffectComponent(.shader(.default))`** — drives `HoverState` ShaderGraph node
- **[NEW] `UnlitMaterial.Program.Descriptor.blendMode = .add`** — additive blend for glowing UI
- `AnchorEntity(.hand(.right, .indexFingerTip))` — hand joint tracking anchor

### Metal (GPU pipeline)
- `MTLBuffer` — vertex/index data; written by compute shaders for sparkle brush
- `MTLComputeCommandEncoder` — dispatches particle simulation + mesh populate kernels
- `MTLCommandBuffer` — submitted to `lowLevelMesh.replace(bufferIndex:using:)`

### ARKit / SwiftUI
- `ImmersiveSpace` — full immersive visionOS space; required for hand tracking
- `RealityView` — embed RealityKit in SwiftUI; used for brush preset thumbnails too

## Code Highlights
```swift
// SpatialTrackingSession for hand tracking
let session = SpatialTrackingSession()
let config = SpatialTrackingSession.Configuration(tracking: [.hand])
let unapproved = await session.run(config)
// AnchorEntity.transform now updates with real hand data

// LowLevelMesh: custom interleaved vertex layout
var descriptor = LowLevelMesh.Descriptor()
descriptor.vertexCapacity = 4096
descriptor.indexCapacity = 8192
descriptor.vertexAttributes = SolidBrushVertex.vertexAttributes  // custom extension
let stride = MemoryLayout<SolidBrushVertex>.stride
descriptor.vertexLayouts = [LowLevelMesh.Layout(bufferIndex: 0, bufferOffset: 0, bufferStride: stride)]
let mesh = try LowLevelMesh(descriptor: descriptor)
mesh.parts.append(LowLevelMesh.Part(indexOffset: 0, indexCount: indexCount,
                                     topology: .triangleStrip, materialIndex: 0, bounds: bounds))
let resource = try MeshResource(from: mesh)
entity.components[ModelComponent.self] = ModelComponent(mesh: resource, materials: [material])

// CPU vertex update
mesh.withUnsafeMutableBytes(bufferIndex: 0) { rawBuffer in
    let vertices = rawBuffer.bindMemory(to: SolidBrushVertex.self)
    vertices[0] = SolidBrushVertex(position: ..., normal: ..., ...)
}

// GPU compute update for sparkle brush
let vertexBuffer = lowLevelMesh.replace(bufferIndex: 0, using: commandBuffer)
encoder.setBuffer(particleSimBuffer, offset: 0, index: 0)
encoder.setBuffer(vertexBuffer, offset: 0, index: 1)
encoder.dispatchThreadgroups(...)
commandBuffer.commit()

// MeshResource extrusion: 3D text
var textString = AttributedString("RealityKit")
textString.font = .systemFont(ofSize: 8.0)
var options = MeshResource.ShapeExtrusionOptions()
options.extrusionMethod = .linear(depth: 2)
options.chamferRadius = 0.1
let textMesh = try await MeshResource(extruding: textString, extrusionOptions: options)

// LowLevelTexture for GPU-generated background
let descriptor = LowLevelTexture.Descriptor(pixelFormat: .rg16Float,
                                             width: 512, height: 512,
                                             textureUsage: [.shaderWrite, .shaderRead])
let texture = try LowLevelTexture(descriptor: descriptor)
let writeTexture = texture.replace(using: commandBuffer)
encoder.setTexture(writeTexture, index: 0)
```

## Takeaways
- `LowLevelMesh` is the key API for high-performance custom geometry pipelines: bring any vertex buffer layout to RealityKit without format conversion overhead, and update vertices directly from Metal compute shaders
- `SpatialTrackingSession` is the correct visionOS 2 path for accessing hand tracking data in RealityKit apps—it replaces the need to interact with ARKit's session directly
- Use `MeshResource(extruding:)` to convert SwiftUI paths and attributed strings into 3D geometry; it's the fastest path from 2D vector design to RealityKit mesh
- Shader-based hover effects via `HoverEffectComponent(.shader(.default))` + ShaderGraph `HoverState` node enable arbitrarily complex gaze-responsive animations without CPU callbacks

---
_Source: WWDC24 Session 10104 page (abstract, chapter summaries, code samples, and resource links)._
