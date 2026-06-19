# Design no-code games with Reality Composer Pro 3
**WWDC26 · Session 252** · [Watch](https://developer.apple.com/videos/play/wwdc2026/252/)

_Platforms:_ visionOS 27

## Overview
This session demonstrates how Reality Composer Pro 3's Script Graph enables game designers and artists to build complete interactive 3D experiences without writing any Swift code. The presenter walks through building a small visionOS game — a squirrel-and-nut adventure — entirely within RCP3, from prototyping interactions to polished gameplay with custom events and SwiftUI-powered speech bubble UI.

The session opens with a conceptual overview of Script Graph as an event-driven visual scripting system, then builds up game logic incrementally: adding components to entities, connecting event and action nodes, iterating on behavior inside the RCP3 editor, and finally showing advanced techniques including subgraphs (reusable node groups), prototyped subgraphs (shared subgraphs instantiated across multiple Script Graphs), custom events for cross-graph communication, material swapping for visual feedback, and embedding SwiftUI Attachment views as in-world speech bubbles.

The session also previews Live Preview on Apple Vision Pro (coming later in 2026) as the final iteration step — seeing the game in headset without writing a line of code.

## Key Topics

### Script Graph Basics
- Script Graph is a visual, node-based event system in RCP3.
- Graphs are attached to entities as a `ScriptGraphComponent`.
- Nodes represent triggers (e.g., `On Initialize`, `On Tap`, `On Collision`) and actions (e.g., `Play Animation`, `Set Visibility`, `Set Material`, `Move To`).
- Connections between output and input ports define data and event flow.
- Iterate entirely inside the RCP3 editor; preview in the editor viewport.

### Build the Game
- Add a `ScriptGraphComponent` to each interactive entity.
- Wire `On Tap` → `Play Animation` to make the squirrel react when tapped.
- Add game-state variables as parameters; use `If/Else` nodes for conditional logic.
- `Set Transform` node for moving objects (nut pickup/placement).

### Advanced Techniques
- **Subgraphs**: group a set of nodes into a named subgraph to reduce visual clutter and enable reuse within a single Script Graph.
- **Prototyped subgraphs**: promote a subgraph to a Prototype — it can then be instantiated in other Script Graphs, sharing the same node logic. Editing the prototype updates all instances.
- **Custom events**: fire a named event from one Script Graph; other graphs on other entities subscribe via an event listener node. Used to coordinate squirrel/nut state across separate entity graphs.
- **Material swapping**: `Set Material` node changes the material on an entity at runtime — used to show character reaction states (e.g., squirrel going red when the nut is stolen).
- **SwiftUI Attachments**: subscribe to a custom event by name from Swift, read a `String` payload, and render a SwiftUI `Attachment(id:)` view as an in-world speech bubble.

### Live Preview
- Coming later in 2026: target Apple Vision Pro directly from RCP3 for live in-headset game testing.
- No app, no Xcode project, no signing — the full no-code workflow extends to device preview.

## APIs & Frameworks

### Reality Composer Pro 3 — Script Graph (all NEW)
- `ScriptGraphComponent` **[NEW]**: attached to entities to host a Script Graph
- Event nodes: `On Initialize`, `On Tap`, `On Collision`, custom event listener nodes **[NEW]**
- Action nodes: `Play Animation`, `Set Material`, `Set Visibility`, `Move To`, `Set Transform`, `Set Parameter` **[NEW]**
- Logic nodes: `If/Else`, `Gate`, variable getters/setters **[NEW]**
- **Subgraphs**: node groups within a Script Graph **[NEW]**
- **Prototyped subgraphs**: shared, reusable subgraph templates **[NEW]**
- **Custom events**: named events fired/subscribed across entity Script Graphs **[NEW]**
- `Live Preview` to Apple Vision Pro (coming 2026)

### RealityKit (runtime counterparts called from Script Graph)
- `AnimationResource` / `AnimationPlaybackController` — existing; driven by Play Animation node
- `ModelComponent` / `ShaderGraphMaterial` — existing; driven by Set Material node
- Entity transforms — driven by Set Transform / Move To nodes

### SwiftUI Integration
- `scene.subscribe(forEventName:on:)` **[NEW or updated]**: subscribe to a named Script Graph custom event from Swift
- `event.value(_:)` — read typed payload value from the event
- `Attachment(id:)` — existing SwiftUI RealityKit attachment; used for speech bubble UI
- `RealityView { ... } attachments: { ... }` — existing

## Code Highlights

Subscribe to a custom Script Graph event in Swift and show a SwiftUI attachment:
```swift
if let scene = entity.scene {
    scene.subscribe(forEventName: "squirrelTalk", on: { event in
        if let sayThis: String = try? event.value("sayThis") {
            self.sayThis = sayThis
        }
    }).store(in: &cancellables)
}
```

Render the speech bubble as a RealityKit attachment:
```swift
} attachments: {
    Attachment(id: "squirrelTalk") {
        SquirrelTalkAttachmentView(text: sayThis)
    }
}
```

## Takeaways
- Script Graph makes game prototyping accessible to designers and artists without any Swift knowledge — a complete game loop with animations, materials, and UI can be built entirely in RCP3.
- Prototyped subgraphs are the Script Graph equivalent of prefabs: edit once, update everywhere — essential for building games with many similar interactive objects.
- Custom events bridge the Script Graph / Swift boundary cleanly: Script Graph fires typed events, Swift subscribes and responds by updating SwiftUI state.
- The no-code path extends to device preview (coming later in 2026) — the full design-to-playtest cycle will eventually require zero Xcode involvement.

---
_Source: WWDC26 Session 252 page (abstract, chapter summaries, code samples, and resource links)._
