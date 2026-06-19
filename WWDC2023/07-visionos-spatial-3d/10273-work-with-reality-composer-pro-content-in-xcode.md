# Work with Reality Composer Pro content in Xcode
**WWDC23 · Session 10273** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10273/)

_Platforms:_ visionOS 1

## Overview
This session bridges Reality Composer Pro scene authoring with Xcode Swift code, covering the complete workflow of loading a `.rkassets`-backed Reality Composer Pro project as a RealityKit entity hierarchy, designing custom ECS components in the Swift package, wiring SwiftUI UI into 3D space with the new `RealityView` Attachments API, playing audio set up in Reality Composer Pro, and driving promoted Shader Graph material parameters at runtime. The example is an interactive topographical map of Yosemite / Catalina Island with a morphing slider, ambient audio crossfade, and floating SwiftUI point-of-interest buttons.

## Key Topics

**Loading 3D Content**
- Reality Composer Pro projects compile to Swift packages; use `Entity(named:in:)` async init inside a `RealityView` make closure
- `realityKitContentBundle` — auto-generated constant for the package bundle
- Scene tabs in Reality Composer Pro represent root entities; load by string name
- USD assets outside RCP: put them in a Swift Package with `.rkassets` directory; Xcode compiles it to a faster runtime format

**Entity Component System (ECS)**
- Entities hold no data; data lives in Components; Systems run once per frame querying for entities with specific components
- Add a component in code: `entity.components.set(component)` — or in RCP's Inspector Panel via "Add Component"
- Custom components: create a `Component & Codable` struct in the RCP Swift package; RCP auto-generates Inspector UI from Swift properties
- Design-time components: `Codable`, defined in the RCP package, visible in Inspector
- Runtime components: not `Codable`, defined anywhere (Xcode project or RCP package), invisible in Inspector — used for ephemeral runtime state
- Query for entities with a component: `EntityQuery(where: .has(MyComponent.self))`, then `scene?.performQuery(query)`

**RealityView and Attachments API (NEW)**
- `RealityView` **[NEW]** — SwiftUI view that bridges into RealityKit; three closure parameters:
  - `make`: called once; load initial scene; `content.add(entity)`
  - `update`: called on SwiftUI state changes; mutate entities, add/position attachment entities
  - `attachments` ViewBuilder: declare SwiftUI views with `.tag(hashable)` to become RealityKit entities
- `attachments.entity(for: tag)` in the update closure retrieves a SwiftUI view-turned-entity
- Adding the same entity twice is a no-op (set semantics)
- Attachment entities are safe to add in `update` (not duplicated); regular entities created in `update` must guard against duplication
- Data-driven pattern: query for design-time components → create SwiftUI views → store in `@Observable AttachmentsProvider` → `@State` triggers view update → ForEach in attachments ViewBuilder → retrieve as entities in update closure → position relative to marker entities

**Audio Playback**
- In RCP: add an Audio entity (Ambient Audio); attach an audio file as a resource prim
- At runtime: `entity.findEntity(named:)` to get the audio emitter; `AudioFileResource(named:from:in:)` to load the audio; `entity.prepareAudio(_:)` → `AudioPlaybackController`; call `.play()`
- Crossfade: query entities with `AmbientAudioComponent`; mutate `gain` property; store component back

**ShaderGraph Material Properties**
- In RCP Shader Graph: promote an input node (Cmd-click → Promote); name it (e.g., "Progress")
- At runtime: get `ModelComponent` from entity; cast first material to `ShaderGraphMaterial`; call `shaderGraphMaterial.setParameter(name:value:)`; write `ModelComponent` back

## APIs & Frameworks

**RealityKit**
- `Entity(named:in:)` async init **[NEW]** — load named entity from RCP bundle
- `realityKitContentBundle` **[NEW]** — auto-generated bundle constant in RCP Swift package
- `RealityView` **[NEW]** SwiftUI view
- `RealityView.make` closure, `RealityView.update` closure, `RealityView.attachments` ViewBuilder **[NEW]**
- `RealityViewContent.add(_:)` **[NEW]**
- `RealityViewAttachments.entity(for:)` **[NEW]**
- `EntityQuery(where:)` — query DSL for ECS
- `Scene.performQuery(_:)` — returns `QueryResult`
- `entity.components.set(_:)` — add/replace component
- `entity.components[ComponentType.self]` — retrieve component
- `Component` protocol — base for all RealityKit components
- `Entity.id` / `Identifiable` conformance — unique entity identifier usable as attachment tag
- `AmbientAudioComponent` — ambient audio component; `gain` property
- `AudioFileResource(named:from:in:)` **[NEW]** — load audio from RCP scene
- `entity.prepareAudio(_:)` → `AudioPlaybackController` **[NEW API surface]**
- `AudioPlaybackController.play()`, `.pause()`, `.stop()`
- `ModelComponent` — holds materials array
- `ShaderGraphMaterial` **[NEW]** — material type for RCP Shader Graph outputs
- `ShaderGraphMaterial.setParameter(name:value:)` **[NEW]**
- `MaterialParameters.Value.float(_:)` **[NEW]**

**SwiftUI (visionOS)**
- `RealityView` (same as above)
- `.tag(_:)` modifier — existing modifier, now used to tag attachment views
- `@Observable` — used for `AttachmentsProvider` to trigger SwiftUI updates

## Code Highlights

Loading a scene entity:
```swift
RealityView { content in
    let entity = try await Entity(named: "DioramaAssembled", in: realityKitContentBundle)
    content.add(entity)
}
```

Minimal attachment:
```swift
RealityView { _, _ in } update: { content, attachments in
    if let attachmentEntity = attachments.entity(for: "🐠") {
        content.add(attachmentEntity)
    }
} attachments: {
    Button("Learn More") { }
        .background(.green)
        .tag("🐠")
}
```

Design-time vs. runtime components:
```swift
// Design-time — Codable, visible in RCP Inspector
public struct PointOfInterestComponent: Component, Codable {
    public var region: Region = .yosemite
    public var name: String = "Ribbon Beach"
}

// Runtime — not Codable, invisible in RCP
public struct PointOfInterestRuntimeComponent: Component {
    public let attachmentTag: ObjectIdentifier
}
```

Driving ShaderGraph material from a slider:
```swift
try shaderGraphMaterial.setParameter(name: "Progress", value: .float(sliderValue))
modelComponent.materials = [shaderGraphMaterial]
terrain.components.set(modelComponent)
```

## Takeaways
- Separate design-time components (`Codable`, in RCP package) from runtime components (not `Codable`) to control what shows up in RCP's Inspector versus what lives purely in code.
- Use `@Observable AttachmentsProvider` with `@State` as the bridge: adding new entries to the provider triggers SwiftUI to re-evaluate the attachments ViewBuilder, which delivers new SwiftUI views as RealityKit entities on the next `update` call.
- Promote Shader Graph input nodes in Reality Composer Pro to expose them as named parameters settable at runtime via `ShaderGraphMaterial.setParameter(name:value:)` — no rebuild needed.
- Use `entity.id` (the `Identifiable` conformance) as the attachment tag instead of inventing custom identifiers; it's globally unique without extra effort in RCP.

---
_Source: WWDC23 Session 10273 page (abstract, chapters, transcript, and code samples)._
