# Explore Object Tracking for visionOS
**WWDC24 · Session 10101** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10101/)

_Platforms:_ visionOS 2

## Overview
visionOS 2 introduces object tracking — the ability to recognize specific real-world physical objects and use them as virtual anchors. A three-step workflow covers creating a 3D reference model (ideally via Object Capture), training a machine learning model in the Create ML Object Tracking template, and integrating the resulting `.referenceobject` file into an app via Reality Composer Pro, RealityKit, or ARKit APIs.

The session demonstrates a rich globe experience: virtual content (orbiting moon, space station, space shuttle) appears anchored precisely to a physical globe, with an occlusion material making objects disappear behind the real globe for a convincing mixed-reality effect. A coaching UI built with `RealityView` attachments and `SpatialTrackingSession` guides the user to the correct object before tracking begins.

## Key Topics

### Create Reference Object in Create ML
Create ML gains a new "Spatial" category with an **Object Tracking** template. The workflow: import a USDZ asset (photorealistic, ideally from Object Capture), choose a viewing angle (All Angles, Upright, or Front) to optimize tracking for how the object is typically oriented, and click Train. Training runs locally on Apple Silicon Macs (takes a few hours). The output is a `.referenceobject` file — a new file type introduced for object tracking.

### Anchor Virtual Content in Reality Composer Pro
Add an `AnchoringComponent` to a Transform entity and set its target to "Object", then associate the `.referenceobject`. A semi-transparent USDZ ghost of the object appears in the viewport for precise content placement. Child entities (labels, animations, occluder meshes) are placed relative to the anchor. A `ShaderGraph` occlusion material applied to a USDZ occluder entity makes virtual objects realistically hide behind the physical object.

### Coaching UI with RealityKit and ARKit APIs
A coaching UI displays a 50%-opacity preview of the target object before tracking starts. The `SceneEvents.AnchoredStateChanged` event on the `AnchorEntity` signals when tracking begins, triggering a transition animation from the preview position to the tracked object's world position. `SpatialTrackingSession` is used to request authorization and obtain the object anchor's world transform for that animation.

### ARKit Object Tracking API
A new ARKit API (released alongside this session) provides access to tracked objects' bounding boxes, refined state information (ready-to-track, error conditions), and the corresponding USDZ files. A sample app is available for download.

## APIs & Frameworks

**ARKit**
- `SpatialTrackingSession` **[NEW]** — requests authorization and provides transform data
  - `SpatialTrackingSession.Configuration` **[NEW]** with tracking options including `.object` and `.world`
  - `run(_:)` async method returning authorization result
- `ReferenceObject` **[NEW]** — loaded from a `.referenceobject` file URL
  - `ReferenceObject(from:)` async init **[NEW]**
  - `.usdzFile` property — URL to the embedded USDZ asset **[NEW]**
- `ObjectTrackingProvider` **[NEW]** — new ARKit data provider for object tracking
- Object anchor bounding box and state APIs **[NEW]** (see "Exploring object tracking with ARKit" sample)

**RealityKit**
- `AnchoringComponent` — enhanced with a new "Object" target type for `.referenceobject` files **[NEW target]**
- `Entity.init(contentsOf:)` — async load of USDZ model entities (existing)
- `OpacityComponent` — used to set preview entity to 50% opacity (existing)
- `SceneEvents.AnchoredStateChanged` — subscribe to detect when an object anchor becomes tracked **[NEW event use]**
- `Entity.isAnchored` flag — check in the update loop (existing)
- `Entity.transformMatrix(relativeTo:)` — get world transform of the anchor entity (existing)
- `RealityView` attachments — place SwiftUI elements on RealityKit anchor entities (existing)
- Occlusion material via `ShaderGraph` editor — makes virtual objects hide behind physical geometry (existing)

**Create ML**
- Object Tracking template (Spatial category) **[NEW]** — produces `.referenceobject` file
- Viewing angle options: All Angles, Upright, Front **[NEW]**
- Requires Apple Silicon Mac; training runs locally

**Reality Composer Pro**
- Object anchor entity with AnchoringComponent "Object" target **[NEW]**
- Semi-transparent USDZ ghost in viewport for placement reference **[NEW]**
- Behaviors component for tap-gesture event handling (existing)
- Timeline animations for orbital motion (existing)

## Code Highlights

Display a USDZ preview of the target object (coaching UI):
```swift
struct ImmersiveView: View {
    @State var globeAnchor: Entity? = nil
    var body: some View {
        RealityView { content in
            let refObjURL = Bundle.main.url(forResource: "globe", withExtension: ".referenceobject")
            let refObject = try? await ReferenceObject(from: refObjURL!)
            let globePreviewEntity = try? await Entity.init(contentsOf: (refObject?.usdzFile)!)
            globePreviewEntity!.components.set(OpacityComponent(opacity: 0.5))
            content.add(globePreviewEntity!)
        }
    }
}
```

Check anchor state to switch between coaching UI and live experience:
```swift
let updateSub = content.subscribe(to: SceneEvents.AnchoredStateChanged.self) { event in
    if let anchor = globeAnchor, event.anchor == anchor {
        if event.isAnchored {
            // Object anchor found, trigger transition animation
        } else {
            // Object anchor not found, display coaching UI
        }
    }
}
```

Use `SpatialTrackingSession` to get the anchor's world transform for a coaching animation:
```swift
let trackingSession = SpatialTrackingSession()
let config = SpatialTrackingSession.Configuration(tracking: [.object, .world])
if let result = await trackingSession.run(config) {
    if result.anchor.contains(.object) {
        // Tracking not authorized, adjust experience accordingly
    }
}
let objectTransform = globeAnchor?.transformMatrix(relativeTo: nil)
```

## Takeaways
- Use Create ML's Object Tracking template to train a `.referenceobject` from any USDZ — no ML expertise needed; training is fully local on Apple Silicon.
- Prefer the `AnchoringComponent` path in Reality Composer Pro for static content placement; use the new ARKit API when precise per-frame bounding box or state data is needed.
- Apply an occlusion material to a USDZ duplicate of the physical object to make virtual content realistically disappear behind it.
- Build a coaching UI using `SceneEvents.AnchoredStateChanged` and a semi-transparent preview entity so users know which physical object to look for before tracking begins.

---
_Source: WWDC24 Session 10101 page (abstract, chapter summaries, code samples, and resource links)._
