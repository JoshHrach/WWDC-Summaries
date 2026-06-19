# What's New in USD
**WWDC20 · Session 10613** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10613/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11

## Overview
The session introduced two major advances for the USD ecosystem on Apple platforms: USDZ export from Reality Composer (complementing the existing import and Reality File export), and a set of new preliminary AR USD schemas developed in collaboration with Pixar. Together, these changes close the content creation loop — Reality Composer content can now travel through digital content creation (DCC) tools and return with all AR-specific properties (behaviors, physics, anchoring, audio) intact.

The new schemas — covering scenes, anchoring, behaviors, physics, spatial audio, 3D text, and playback/scene-understanding metadata — are designed with backward compatibility in mind: DCC tools that don't yet understand a schema can still open and edit the file without destroying the AR-specific data.

## Key Topics

### USDZ Export from Reality Composer
Reality Composer now exports to USDZ in addition to Reality File format. This enables a full DCC interoperability loop:
- Import USDZ from online galleries, DCCs, or Reality Converter
- Add AR-specific properties (anchoring, behaviors, physics, audio) in Reality Composer
- Export to USDZ for further editing in any DCC that supports the format
- Re-import into Reality Composer or deploy to RealityKit-based apps

DCCs that haven't adopted the new AR schemas preserve but ignore unknown schema data, keeping the AR properties intact for round-trip workflows.

### Scene Library Structure
Multiple scenes can be defined in a single USDZ file using a `sceneLibrary` kind on the root prim. Each scene is an Xform under the root:
- Active scenes use `def`; inactive scenes use `over`
- Each scene can be given a `sceneName` for loading by name in RealityKit (same as Reality File)
- RealityKit, AR Quick Look, and Reality Composer support a single active scene per library

### AR Anchoring Schema
`Preliminary_AnchoringAPI` — applied schema specifying where content anchors in the real world:
- `preliminary:anchoring:type` — `"image"`, `"plane"` (horizontal/vertical), `"face"`
- Image anchoring requires a `Preliminary_ReferenceImage` prim with `image` (JPEG/PNG asset) and `physicalWidth` in centimeters

### Behavior Schema
`Preliminary_Behavior` — prim type containing trigger and action relationships:
- `triggers` — array of relationships to `Preliminary_Trigger` prims
- `actions` — array of relationships to `Preliminary_Action` prims
- Multiple triggers: any trigger fires all actions
- Multiple actions: run serially
- Group actions support serial or parallel execution
- `Preliminary_Trigger`: `info:id` (e.g., `"tap"`, `"proximity"`); `affectedObjects` relationship
- `Preliminary_Action`: `info:id` (e.g., `"emphasize"`); action-specific properties (e.g., `motionType = "bounce"`)
- Behaviors scoped to the scene they're defined in (multi-scene files)

### Physics Schemas
- `Preliminary_PhysicsColliderAPI` — applied schema; `preliminary:physics:collider:convexShape` relationship
- `Preliminary_PhysicsRigidBodyAPI` — applied schema; `preliminary:physics:rigidBody:mass` (in kg)
- `Preliminary_PhysicsMaterialAPI` — applied schema on a Material prim; restitution and friction properties
- `Preliminary_InfiniteColliderPlane` — infinite ground plane; marked as scene ground plane via `customData`
- `Preliminary_PhysicsGravitationalForce` — defines gravity vector (in stage units/sec²)

### Spatial Audio Schema
`SpatialAudio` — new prim type (inherits from Xform):
- `filePath` — audio asset reference
- `auralMode` — `"spatial"` (position-based) or non-spatial
- `startTime`, `mediaOffset` — playback timing
- Transform inherited from parent or set explicitly for positional audio
- Played alongside USD animation track; can be invoked via "USD animation" action in Reality Composer

### 3D Text Schema
`Preliminary_Text` — prim type for 3D text:
- `content` — text string
- `font` — ordered list of font names (fallback chain)
- `wrapMode`, `horizontalAlignment`, `verticalAlignment`
- Additional bounding volume, depth, and alignment properties

### Metadata Extensions
- `playbackMode` — `"loop"`, `"once"`, etc.; hint to viewers on playback behavior
- `autoPlay` — boolean; `false` means viewer shows play button
- `preliminary_collidesWithEnvironment` — scene-level; enables interaction with LiDAR-scanned real-world geometry (scene understanding)

### Reality Converter
New tool introduced alongside USD export that converts DCC-format files to USDZ for import into Reality Composer and other apps.

## APIs & Frameworks

### USD / USDZ Schemas (Preliminary — proposed to Pixar)
- `sceneLibrary` kind metadata **[NEW]** — multi-scene USDZ structure
- `sceneName` metadata **[NEW]** — human-readable scene name for RealityKit loading
- `Preliminary_AnchoringAPI` applied schema **[NEW]**
  - `preliminary:anchoring:type` token (image, plane, face)
  - `preliminary:imageAnchoring:referenceImage` relationship
- `Preliminary_ReferenceImage` prim **[NEW]**
  - `image` asset, `physicalWidth` double (cm)
- `Preliminary_Behavior` prim **[NEW]**
  - `triggers`, `actions` relationship arrays
- `Preliminary_Trigger` prim **[NEW]** — `info:id`, `affectedObjects`
- `Preliminary_Action` prim **[NEW]** — `info:id`, action-specific properties
- `Preliminary_PhysicsColliderAPI` applied schema **[NEW]**
- `Preliminary_PhysicsRigidBodyAPI` applied schema **[NEW]**
- `Preliminary_PhysicsMaterialAPI` applied schema **[NEW]**
- `Preliminary_InfiniteColliderPlane` prim **[NEW]** + `preliminary_isSceneGroundPlane` custom data
- `Preliminary_PhysicsGravitationalForce` prim **[NEW]**
- `SpatialAudio` prim **[NEW]** (inherits Xform) — `filePath`, `auralMode`, `startTime`, `mediaOffset`
- `Preliminary_Text` prim **[NEW]** — `content`, `font`, `wrapMode`, `horizontalAlignment`, `verticalAlignment`
- `playbackMode` stage metadata **[NEW]** — `"loop"`, `"once"`, etc.
- `autoPlay` stage metadata **[NEW]**
- `preliminary_collidesWithEnvironment` scene metadata **[NEW]** — LiDAR scene understanding interaction

### Tools
- **Reality Composer** — now supports USDZ export **[NEW]**
- **Reality Converter** **[NEW]** — converts DCC formats to USDZ
- **usdz Python tool** — adds spatial audio schema to existing USD files (available on developer.apple.com)

## Code Highlights

Scene library structure (USDA):
```usda
def Xform "Root" (kind = "sceneLibrary") {
    def Cube "MyCubeScene" (sceneName = "My Cube Scene") { ... }
    over Sphere "MySphereScene" (sceneName = "My Sphere Scene") { ... }
}
```

Image anchoring:
```usda
def Cube "ImageAnchoredCube" (prepend apiSchemas = ["Preliminary_AnchoringAPI"]) {
    uniform token preliminary:anchoring:type = "image"
    rel preliminary:imageAnchoring:referenceImage = <ImageReference>
    def Preliminary_ReferenceImage "ImageReference" {
        uniform asset image = @image.png@
        uniform double physicalWidth = 12
    }
}
```

Tap-and-bounce behavior:
```usda
def Preliminary_Behavior "TapAndBounce" {
    rel triggers = [<Tap>]; rel actions = [<Bounce>]
    def Preliminary_Trigger "Tap" { uniform token info:id = "tap"; rel affectedObjects = [</Cube>] }
    def Preliminary_Action "Bounce" { uniform token info:id = "emphasize"; uniform token motionType = "bounce"; rel affectedObjects = [</Cube>] }
}
```

Spatial audio:
```usda
def SpatialAudio "HorseNeigh" {
    uniform asset filePath = @Horse.m4a@
    uniform token auralMode = "spatial"
    uniform timeCode startTime = 65.0
    uniform double mediaOffset = 0.33333333333
    double3 xformOp:translate = (0, 0.5, 0.1)
    uniform token[] xformOpOrder = ["xformOp:translate"]
}
```

## Takeaways

- Reality Composer now exports to USDZ, completing a full round-trip workflow with DCCs like Houdini and Maya — AR behaviors and schemas are preserved even in editors that don't understand them.
- The new preliminary AR USD schemas (anchoring, behaviors, physics, audio, text) are backward-compatible and proposed to Pixar as contributions to the open USD standard.
- `preliminary_collidesWithEnvironment` on a scene enables virtual content to interact with LiDAR-scanned real-world geometry from RealityKit scene understanding.
- The `SpatialAudio` prim plays audio from a specific 3D position alongside USD animation, opening up positional audio experiences without requiring RealityKit behaviors.

---
_Source: WWDC20 Session 10613 page (abstract, transcript, code samples, and resource links)._
