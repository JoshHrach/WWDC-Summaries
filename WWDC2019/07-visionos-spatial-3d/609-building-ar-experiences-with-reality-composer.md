# Building AR Experiences with Reality Composer
**WWDC19 · Session 609** · [Watch](https://developer.apple.com/videos/play/wwdc2019/609/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15

## Overview
Reality Composer is a new tool (app on iOS/iPadOS/macOS) that lets developers quickly prototype and build AR scenes without writing code. It provides a library of built-in 3D objects, a drag-and-drop scene layout editor, a behavior system for no-code interactions and animations, a physics simulation, and deep Xcode integration through code generation. The resulting `.reality` file integrates directly into RealityKit apps.

This session covers three areas: building scenes in Reality Composer (anchors, content library, USDZ import), authoring behaviors with triggers and action sequences (including physics), and integrating the resulting Reality files into an app via Xcode's code generation and the RealityKit API. Key developer features include generated Swift types for scenes and named entities, `NotifyAction` closures for bridging RC behaviors to app code, and `NotificationTrigger` for posting trigger events programmatically — including post-with-overrides for dynamic entity targeting.

## Key Topics

**Scenes and Anchors**
- Every scene has exactly one anchor type: horizontal (table/floor), vertical (wall), image, or face
- Projects can contain multiple scenes; scene selector in Reality Composer's top-left corner
- Objects named in Reality Composer's Configure panel become strongly typed properties in generated Swift code

**Content and USDZ Import**
- Built-in content library: basic shapes, text, picture frames, and hundreds of ready-to-use 3D objects
- Custom USDZ files can be dragged in; the Replace action preserves existing behaviors on an object
- Picture Frame lets 2D photos/graphics appear as objects in the scene

**Behaviors**
- A behavior = one trigger + one action sequence
- Triggers: Scene Start, Tap, Proximity (distance threshold), Collision, Notification (programmatic) **[NEW]**
- Action sequences: sequential by default; drag-and-drop cards into groups to run in parallel; support looping; can be marked exclusive (stops any other running exclusive sequence)
- Built-in actions: visibility (show/hide with fade, scale, etc.), emphasis animations (BasicPop, bounce, spin), orbit, move by/to, look at camera, USD asset animation, play sound/ambient/music, force impulse
- Notify action: custom closure set in app code, called within an action sequence
- Notification trigger: programmatic trigger posted from app code via `.post()` or `.post(overrides:)`

**Physics Simulation**
- Materials: wood, metal, plastic, rubber, stone, ice — affect friction and restitution
- Objects must opt-in: Collide (participates in collision detection) and Simulate (moved by physics engine)
- Collision shapes: box, sphere, capsule
- Gravity configurable per scene; Force action applies an initial impulse with direction and magnitude
- Collision trigger: fires action sequence when two specified objects collide

**Xcode Integration and Code Generation**
- Drag a `.rcproject` file into an Xcode project or start from the RealityKit AR/Game template
- Xcode previews `.rcproject` inline; Open in Reality Composer button launches the editor
- At build time, Xcode automatically exports the project as an optimized `.reality` file
- Code generation produces a Swift source file matching the project name; scenes become typed structs with `AnchorEntity` conformance
- Named entities become typed properties; notify actions and notification triggers become nested types inside `Actions` and `Notifications` objects
- `allActions` and `allNotifications` arrays support Swift collection operations (filter, map, etc.)

**Loading API**
- Generated: `SolarSystem.loadSeasonsChapter()` / `SolarSystem.loadSeasonsChapterAsync()` — one-liner sync/async scene load
- Direct API: `Entity.loadAnchor(contentsOf:withName:)` / `Entity.loadAnchorAsync(contentsOf:withName:)` for downloaded `.reality` files
- `anchor.findEntity(named:)` — access named entities by string when not using code generation

## APIs & Frameworks

### Reality Composer (App/Tool, NEW)
- `.rcproject` file format **[NEW]** — Reality Composer project; previewed in Xcode
- `.reality` file format **[NEW]** — optimized runtime bundle exported from `.rcproject` at Xcode build time

### RealityKit — Generated API (NEW)
- Code-generated Swift struct per project (e.g., `SolarSystem`) **[NEW]**
- `<ProjectName>.load<SceneName>() -> <SceneName>` **[NEW]** — synchronous scene load returning typed anchor entity
- `<ProjectName>.load<SceneName>Async() -> LoadRequest<<SceneName>>` **[NEW]** — async via Combine
- Named entity properties on scene struct (e.g., `seasons.sun`, `seasons.earth`) **[NEW]**
- `<SceneName>.Actions` nested type with per-notify-action properties **[NEW]**
- `<SceneName>.Actions.allActions: [NotifyAction]` **[NEW]**
- `NotifyAction.onAction: ((Entity?) -> Void)?` **[NEW]** — set closure to handle notification
- `NotifyAction.identifier: String` **[NEW]**
- `<SceneName>.Notifications` nested type with per-trigger properties **[NEW]**
- `NotificationTrigger.post()` **[NEW]** — fires the trigger's action sequences
- `NotificationTrigger.post(overrides: [String: Entity])` **[NEW]** — dynamic entity targeting

### RealityKit — Direct File API (NEW)
- `Entity.loadAnchor(contentsOf url: URL, withName sceneName: String) throws -> AnchorEntity` **[NEW]**
- `Entity.loadAnchorAsync(contentsOf:withName:) -> LoadRequest<AnchorEntity>` **[NEW]**
- `Entity.load(contentsOf:withName:)` **[NEW]** — loads entity tree without anchor
- `anchor.findEntity(named:) -> Entity?` — string-based entity lookup

### Reality Composer Behavior Triggers (NEW)
- Scene Start trigger
- Tap trigger
- Proximity trigger (configurable distance threshold)
- Collision trigger (two objects)
- Notification trigger (programmatic via `NotificationTrigger.post()`)

### Reality Composer Actions (NEW)
- Visibility: fade in/out, scale up/down, hide, show
- Emphasis: BasicPop, bounce, jiggle, custom
- Spin, Orbit
- Move By, Move To, Look At Camera
- USD animation playback
- Play Sound (spatialized), Play Ambient, Play Music
- Force (physics impulse)
- Notify action (app code callback)

## Code Highlights

Synchronous scene load using code generation (2 lines to show AR):
```swift
let seasonsChapter = try! SolarSystem.loadSeasonsChapter()
arView.scene.addAnchor(seasonsChapter)
```

Async load with Combine:
```swift
SolarSystem.loadSeasonsChapterAsync()
    .sink(receiveCompletion: { _ in },
          receiveValue: { [weak self] anchor in
              self?.arView.scene.addAnchor(anchor)
          })
    .store(in: &cancellables)
```

Observing notify actions (filtered):
```swift
sizeChapter.actions.allActions
    .filter { $0.identifier.hasPrefix("display") }
    .forEach { action in
        action.onAction = { [weak self] entity in
            self?.showDetail(for: entity)
        }
    }
```

Posting a notification trigger:
```swift
sizeChapter.notifications.scaleToRelativeSizes.post()
```

Post with overrides for dynamic entity targeting:
```swift
let clonedStar = specialStar.clone(recursive: true)
clonedStar.position = SIMD3(0.1, 0.05, 0)
arView.scene.addAnchor(clonedStar)
sizeChapter.notifications.showGoldStar.post(overrides: ["special star": clonedStar])
```

## Takeaways
- Reality Composer's behavior system — triggers, action sequences, groups, loops, physics — enables non-trivial AR experiences with zero code, making rapid iteration in physical space practical.
- Xcode code generation eliminates string-based entity/scene lookups; named objects in Reality Composer become type-safe Swift properties.
- `NotifyAction` and `NotificationTrigger` are the clean bridge between designer-authored behaviors and app business logic — they can be set/posted from anywhere in Swift code.
- Post-with-overrides on `NotificationTrigger` enables reuse of a single RC action sequence across cloned entities, keeping scene authoring DRY.

---
_Source: WWDC19 Session 609 page (abstract, full transcript, and resource links)._
