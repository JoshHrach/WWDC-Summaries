# Explore Materials in Reality Composer Pro
**WWDC23 · Session 10202** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10202/)

_Platforms:_ visionOS 1

## Overview
This session covers how to create custom 3D materials for visionOS using Reality Composer Pro's Shader Graph editor. The key new material type is `ShaderGraphMaterial`, which replaces `CustomMaterial` (Metal-coded shaders) for visionOS and is based on the open MaterialX standard. Materials are node graph networks built visually in the Shader Graph editor rather than written in Metal code, enabling both artists and developers to design dynamic, physically based materials.

The session walks through building a real-world topographical map material applied to a Yosemite Valley diorama model: computing contour lines using Position, Separate, Modulo, and IfGreater nodes; organizing reusable parts into node graphs (with instances); and finally building geometry modifiers that reshape flat geometry from height map data and animate between two terrains (Yosemite Valley and Catalina Island). It also explains how custom material inputs (promoted properties) can be read and set from Swift code at runtime.

## Key Topics

- **Materials overview** — Materials define object appearance using physically based rendering (PBR) properties: base color, metalness, roughness. Can use constant values, images, animation, or geometry modification.
- **ShaderGraphMaterial (new for visionOS)** — Replaces `CustomMaterial` (Metal-based) for visionOS; based on MaterialX open standard; built visually in Shader Graph editor; supports two shader types: Physically Based (simple PBR) and Custom (advanced, animated, with geometry modifiers).
- **Shader Graph editor in Reality Composer Pro** — Visual node-based editor; double-click to add nodes from node picker; drag connections between node outputs and inputs; inspector for constant values and node properties; real-time preview in viewport.
- **Core Shader Graph nodes** — `Position` (3D render position), `Separate3` (extract X/Y/Z components), `Modulo` (remainder for repeating bands), `IfGreater` (comparison/branch returning one of two values), `Combine3` (assemble 3D vector), `Image` (sample texture/EXR), `Mix` (blend two values by a factor), `Remap` (remap value range), `Multiply`.
- **Node graphs** — Group nodes into a named, reusable node graph via right-click Compose Node Graph; create instances (live copies that inherit changes) via Create Instance; add typed inputs/outputs in inspector to parameterize node graphs (e.g., Spacing: Float, Color: Color3).
- **Geometry modifiers** — Custom shader feature that displaces model vertices at render time rather than modifying mesh data; add Geometry Modifier surface alongside PBR surface in Outputs node; use EXR height map via Image node; Combine3 for directional offset (Y only); connect precomputed normals via Remap for correct lighting after displacement.
- **Dynamic material animation** — Mix between two sets of terrain images (heights, colors, normals) using a Mix node and a 0–1 progress constant; promote the constant to a material input so it can be set from Swift code.
- **Promoted inputs / Swift integration** — Use Promote command to convert a constant node into a named material input; input becomes a property accessible via Swift code at runtime to drive animations.
- **Material binding** — Assign materials to models in the Material Bindings section of the inspector in Reality Composer Pro.

## APIs & Frameworks

**RealityKit**
- `ShaderGraphMaterial` **[NEW]** — new material type for visionOS; replaces `CustomMaterial` for this platform; based on MaterialX
- `CustomMaterial` — previous Metal-shader-based material (iOS/iPadOS only); not used on visionOS
- Geometry Modifier surface **[NEW for visionOS]** — custom shader feature displacing model vertices at render time
- Material inputs (promoted properties) **[NEW workflow]** — constants promoted to named inputs; settable from Swift code at runtime

**Reality Composer Pro (tool, not SDK)**
- Shader Graph editor **[NEW]** — visual node graph editor for `ShaderGraphMaterial`
- Node graphs **[NEW]** — reusable, nestable shader sub-graphs with typed inputs/outputs
- Node graph instances **[NEW]** — live copies of a node graph; inherit changes from original
- Compose Node Graph command **[NEW]** — creates a named node graph from selected nodes
- Create Instance command **[NEW]** — creates a live instance of a node graph
- Promote command **[NEW]** — converts a constant into a named material input

**Shader Graph Nodes (MaterialX-based)**
- `Position` node — 3D world/object-space position at the current render point
- `Separate3` node — splits a 3D vector into scalar X, Y, Z outputs
- `Modulo` node — floating-point remainder; used for repeating band patterns
- `IfGreater` node — returns TrueResult or FalseResult based on a comparison; supports Float and Color3 output types
- `Combine3` node — assembles 3 scalars into a 3D vector
- `Image` node — samples a texture (PNG, EXR, etc.) at UV coordinates
- `Mix` node — linear blend between two values by a 0–1 factor
- `Remap` node — maps a value from one range to another (used to convert normals 0–1 → -1–1)
- `Multiply` node — multiplies two values

**MaterialX (open standard)**
- Open standard originally created by Industrial Light & Magic (2012)
- Basis for `ShaderGraphMaterial` on visionOS

## Code Highlights

Accessing a promoted material input from Swift to drive terrain animation:
```swift
// After loading the Reality Composer Pro scene:
if var material = model.model?.materials.first as? ShaderGraphMaterial {
    try? material.setParameter(name: "Progress", value: .float(animationProgress))
    model.model?.materials = [material]
}
```

Setting a material input value for `ShaderGraphMaterial`:
```swift
// ShaderGraphMaterial parameter set pattern
var mat = entity.components[ModelComponent.self]!.materials[0] as! ShaderGraphMaterial
try mat.setParameter(name: "Spacing", value: .float(0.1))
```

## Takeaways

- Use `ShaderGraphMaterial` (not `CustomMaterial`) for all custom materials on visionOS — it is the only supported custom material type on this platform and requires no Metal code.
- The Shader Graph editor in Reality Composer Pro provides a visual, artist-friendly workflow for building complex materials; node graphs allow reusable sub-graphs with parameterized inputs.
- Geometry modifiers in `ShaderGraphMaterial` can reshape flat base geometry at render time from height map data — enabling dynamic, data-driven terrain and morphable shapes without baking geometry.
- Promote a Shader Graph constant to a named material input to expose it as a settable Swift property, enabling runtime animations and interactivity driven from your app's code.

---
_Source: WWDC23 Session 10202 page (abstract, chapter summaries, transcript, and resource links)._
