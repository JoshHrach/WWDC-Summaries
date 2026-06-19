# Create Immersive Unity Apps
**WWDC23 · Session 10088** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10088/)

_Platforms:_ visionOS 1

## Overview
Unity and Apple have worked together for two years to bring Unity's full development experience to visionOS. This session introduces Unity PolySpatial, the technology layer that translates Unity content into RealityKit for rendering in the Shared Space, and the Volume Camera concept that determines how Unity scenes are presented in the platform's windowing model.

The session is structured around five topics: achieving the right visual look with Unity's materials and Shader Graph, the new Play to Device workflow for rapid iteration, the Volume Camera concept (bounded vs. unbounded volumes), building interactions with look-and-tap, hand tracking, and AR Foundation, and preparing existing Unity projects for the platform. Unity support requires Unity 2022 and is delivered through a beta program.

## Key Topics

### Unity PolySpatial
All content in the Shared Space is rendered using RealityKit. Unity PolySpatial translates Unity content into RealityKit equivalents:
- **MeshRenderer / SkinnedMeshRenderer** → RealityKit mesh entities
- **Physically Based Materials** (URP Lit/Simple Lit/Complex Lit; Built-in Standard Shader) → `RealityKit.PhysicallyBasedMaterial`
- **Unity Shader Graph** → MaterialX → `RealityKit.ShaderGraphMaterial`
- **Unlit Shader** → unlit RealityKit material
- **Occlusion Shader** → RealityKit occlusion material (lets passthrough show through)
- **Particle effects (Shuriken)** → RealityKit particle system (if compatible) or baked mesh
- **Sprites** → 3D meshes
- **Unity Physics, Animation/Timeline, NavMesh, MonoBehaviours** → run natively in Unity simulation, not affected by PolySpatial

Handwritten HLSL shaders are not supported for RealityKit rendering. Use Shader Graph or render to a `RenderTexture` first, then use it as a Shader Graph texture input.

### Play to Device (New)
Play to Device eliminates the build step during development. The Unity editor streams the scene to the visionOS Simulator or a real device in real-time:
- Live changes to materials, textures, and Shader Graphs appear instantly.
- Input events are sent back to the editor for debugging.
- Simulation continues running; attach the debugger to the editor process.
- Currently available only for Shared Space (bounded volume) content.

### Volume Camera
The Volume Camera is the Unity-side mechanism that maps a Unity scene into the visionOS windowing model:

**Bounded Volume**:
- Appears in the Shared Space alongside other apps.
- Has explicit dimensions (scene units) and a real-world size.
- Users can reposition but not resize it.
- The volume camera's transform and dimensions define the visible region of the scene.
- Content outside the bounds is clipped by RealityKit; add back-facing meshes for visible clip planes.

**Unbounded Volume**:
- Fills the entire Full Space (immersive mode).
- Selects the entire Unity scene; no dimensional clipping.
- The volume camera transform maps scene units to real-world meters.
- Only one unbounded volume camera can be active at a time.
- Required for hand tracking, head pose data, and ARKit data.

### Input and Interaction
| Input type | Available in | API |
|---|---|---|
| Look + Tap (WorldTouch) | Bounded and Unbounded | Unity WorldTouch events |
| Full hand tracking (joint data) | Unbounded only | Unity Hands package |
| Head pose | Unbounded only | Unity Input System |
| ARKit (planes, world mesh, image markers) | Unbounded only | AR Foundation + ARKit |
| Bluetooth (keyboard, game controller) | Both | Unity Input System |

Objects must have input colliders to receive tap events. Direct touch (finger reaching out) and distant tap (look + pinch) are both supported. Up to two simultaneous tap actions are in progress at once.

Interaction migration guidance:
- Touch → add input colliders, use WorldTouch.
- VR controllers → redesign around tap or hand tracking.
- Existing hand input → compatible as-is.
- Unity UI (uGUI, UI Toolkit) → supported directly.
- Other UI → works if using `MeshRenderer` or `RenderTexture` on a mesh.

### Preparing Existing Projects
- Use Unity 2022 or later (start upgrading now).
- Convert handwritten shaders to Shader Graph.
- Adopt Universal Render Pipeline (built-in pipeline supported but not future-focused).
- Switch to the Input System package (mixed-mode supported; platform events require Input System).
- Decide between Shared Space (bounded) vs. Full Space (unbounded) based on interaction needs.

## APIs & Frameworks

**Unity PolySpatial (Unity-side, all NEW for visionOS)**
- `PolySpatial` — Unity package translating content to RealityKit
- `VolumeCamera` component **[NEW]** — scene-to-world mapping; bounded or unbounded mode
- `VolumeCamera.Dimensions` — scene-space dimensions of the bounded volume
- `WorldTouch` events — 3D tap events with position and entity reference
- Unity Hands package — low-level hand joint data (unbounded volumes only)
- Unity Input System — head pose, Bluetooth device input
- AR Foundation + ARKit — plane detection, world mesh, image markers (unbounded only)

**RealityKit (Apple-side, used by PolySpatial)**
- `PhysicallyBasedMaterial` — target for Unity PBR materials
- `ShaderGraphMaterial` — target for Unity Shader Graph (via MaterialX)
- Particle system — target for compatible Shuriken particle effects
- Occlusion material — lets passthrough show through Unity objects

**Unity Shader Graph**
- Shader Graph — primary shader authoring tool for visionOS
- MaterialX — intermediate format for Shader Graph to RealityKit conversion
- Supported: most standard Shader Graph nodes
- Unsupported: handwritten HLSL/GLSL shaders for direct RealityKit rendering

**Tools**
- Play to Device **[NEW]** — Unity editor → visionOS Simulator/device live streaming

## Code Highlights
No code samples in this session (Unity/editor workflow focused). Key workflow:
1. Add `VolumeCamera` component to a GameObject to define the visible scene region.
2. Add `Collider` components to objects for tap input.
3. Handle `WorldTouch` events in `MonoBehaviour` scripts.
4. For hand/AR features, use an unbounded `VolumeCamera` and request appropriate permissions.

## Takeaways
- Unity PolySpatial handles material translation (PBR → `PhysicallyBasedMaterial`; Shader Graph → MaterialX → `ShaderGraphMaterial`) automatically; no changes needed for standard materials.
- The Volume Camera is the key new concept: bounded for Shared Space, unbounded for Full Space with hand tracking and ARKit access.
- Play to Device dramatically improves iteration time by streaming the Unity scene directly to device without building.
- Handwritten shaders must be converted to Shader Graph; the built-in pipeline works but URP is the forward path; use the Input System package for all platform input.

---
_Source: WWDC23 Session 10088 page (abstract, chapter summaries, and resource links)._
