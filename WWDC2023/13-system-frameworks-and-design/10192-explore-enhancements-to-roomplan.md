# Explore Enhancements to RoomPlan
**WWDC23 · Session 10192** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10192/)

_Platforms:_ iOS 17, iPadOS 17

## Overview
This session presents major new capabilities in RoomPlan for iOS 17, centered on MultiRoom scanning support, custom ARSession integration, richer room representations, and enhanced export options with 3D model replacement. The biggest addition is `StructureBuilder`, which merges individual `CapturedRoom` scans into a unified `CapturedStructure` representing an entire floor plan.

Two strategies for achieving a common coordinate system across multiple scans are detailed: continuous ARSession (setting `pauseARSession: false` in `stop()`) and ARSession relocalization (saving and restoring `ARWorldMap`). The session also covers new room element types (sections, polygonal walls, floors, object attributes), VoiceOver support for `RoomCaptureView`, and a new `ModelProvider` API to replace bounding-box objects with matching 3D models during USDZ export.

## Key Topics

- **Custom ARSession support** — Pass a custom `ARSession` with `ARWorldTrackingConfiguration` to `RoomCaptureSession.init(arSession:)` to combine RoomPlan with existing AR experiences, high-quality image capture, or plane detection; new `pauseARSession` parameter in `stop()`.
- **MultiRoom with continuous ARSession** — Set `pauseARSession: false` to keep the ARSession running between scans; all scans share the same world coordinate system; reuse same `RoomCaptureSession` instance.
- **MultiRoom with ARSession relocalization** — Save `ARWorldMap` to disk after first scan; load it as `ARWorldTrackingConfiguration.initialWorldMap` before second scan; wait for relocalization before starting next scan; suitable for scanning at different times.
- **StructureBuilder** — New API merging multiple `CapturedRoom` objects into a `CapturedStructure`; call `structureBuilder.capturedStructure(from:)` async; export to USDZ directly from `CapturedStructure.export(to:)`.
- **New room representations** — Sections (livingRoom, bedroom, bathroom, kitchen, diningRoom) with position and floor; polygonal walls and floors via `polygonCorner`; curved walls now in final result from `RoomCaptureView`; object `attributes` (e.g., stool vs. dining chair vs. office chair); `parent` identifier on surfaces and objects; `floors` array in `CapturedRoom`.
- **VoiceOver support** — `RoomCaptureView` now provides audio guidance and object descriptions when VoiceOver is enabled.
- **Export enhancements** — `metadataURL` parameter in `export()` creates a mapping file (encoded `[String: UUID]`) linking USDZ node names to `CapturedRoom` element identifiers; `ModelProvider` maps categories+attributes to 3D model URLs; `.model` `USDExportOptions` value replaces bounding boxes with catalog models.

## APIs & Frameworks

**RoomPlan**
- `RoomCaptureSession.init(arSession:)` **[NEW]** — accepts optional custom `ARSession`
- `RoomCaptureSession.stop(pauseARSession:)` **[NEW parameter]** — `pauseARSession: false` keeps ARSession running between scans
- `StructureBuilder` **[NEW]** — merges multiple `CapturedRoom` into `CapturedStructure`
- `StructureBuilder.init(option:)` **[NEW]** — options include `.beautifyObjects`
- `StructureBuilder.capturedStructure(from:)` **[NEW]** — async method; accepts `[CapturedRoom]`; returns `CapturedStructure`
- `CapturedStructure` **[NEW]** — top-level merged result type; properties: `rooms`, `walls`, `doors`, `windows`, `openings`, `objects`, `floors`, `sections`; method: `export(to:metadataURL:modelProvider:exportOptions:)`
- `CapturedRoom.Section` **[NEW]** — room area element; `label` (`.livingRoom`, `.bedroom`, `.bathroom`, `.kitchen`, `.diningRoom`), `position`, `floor`
- `CapturedRoom.Surface.polygonCorner` **[NEW]** — polygon representation for non-uniform/slanted/curved walls and floors
- `CapturedRoom.Object.attributes` **[NEW]** — polymorphic array of `CapturedRoomAttribute` enums describing object sub-type
- `CapturedRoom.Object.Category.supportedCombinations` **[NEW]** — returns supported attribute combinations for a category
- `CapturedRoom.Surface.parent` **[NEW]** — identifier of parent element (e.g., wall ID for a window)
- `CapturedRoom.Object.parent` **[NEW]** — identifier of parent element (e.g., storage ID for dishwasher)
- `CapturedRoom.floors` **[NEW]** — array of floor `Surface` elements
- `ModelProvider` **[NEW]** — maps `CapturedRoom.Object.Category` and attribute sets to model file URLs
- `ModelProvider.setModelFileURL(_:for:)` **[NEW]** — associates a model URL to a category or attribute set
- `CapturedRoom.export(to:metadataURL:modelProvider:exportOptions:)` **[updated]** — new `metadataURL`, `modelProvider`, `exportOptions` parameters
- `USDExportOptions.model` **[NEW]** — exports with 3D models from `ModelProvider` instead of bounding boxes
- `RoomCaptureView` — VoiceOver audio feedback **[NEW]**

**ARKit**
- `ARSession` — passed to `RoomCaptureSession` for custom AR integration
- `ARWorldTrackingConfiguration` — required for custom ARSession with RoomPlan
- `ARWorldMap` — save/load for relocalization across scanning sessions
- `ARWorldTrackingConfiguration.initialWorldMap` — set loaded `ARWorldMap` to trigger relocalization

## Code Highlights

Continuous ARSession for multiple rooms:
```swift
roomCaptureSession.run(configuration: captureSessionConfig)
roomCaptureSession.stop(pauseARSession: false)  // keep ARSession running
roomCaptureSession.run(configuration: captureSessionConfig)  // same coordinate system
roomCaptureSession.stop()
```

StructureBuilder merge and export:
```swift
let structureBuilder = StructureBuilder(option: [.beautifyObjects])
var capturedRoomArray: [CapturedRoom] = [room1, room2, room3]
let capturedStructure = try await structureBuilder.capturedStructure(from: capturedRoomArray)
try capturedStructure.export(to: destinationURL)
```

ModelProvider setup and export with 3D models:
```swift
let modelProvider = ModelProvider()
try modelProvider.setModelFileURL(chairURL, for: .chair)
try modelProvider.setModelFileURL(stoolURL, for: [ChairAttribute.stool])
try capturedRoom.export(to: outputURL, modelProvider: modelProvider, exportOptions: .model)
```

ARSession relocalization for multi-session scanning:
```swift
let arWorldMap = try NSKeyedUnarchiver.unarchivedObject(ofClass: ARWorldMap.self, from: data)
let config = ARWorldTrackingConfiguration()
config.initialWorldMap = arWorldMap
roomCaptureSession.arSession.run(config, options: [])
// Wait for relocalization to complete, then run next scan
```

## Takeaways

- Use `pauseARSession: false` in `stop()` to scan multiple rooms in sequence while maintaining a single world coordinate system — the simplest approach for same-session multi-room capture.
- `StructureBuilder` handles merging `CapturedRoom` arrays into a unified `CapturedStructure`; optimal for residential spaces up to ~2,000 sq ft with 50+ lux lighting.
- The new `ModelProvider` API and `.model` export option let you replace bounding boxes with catalog 3D models during USDZ export — enabling photorealistic floor plans with minimal code.
- Object `attributes` and `parent` relationships significantly enrich the data available after scanning, enabling more faithful room representations and model matching.

---
_Source: WWDC23 Session 10192 page (abstract, chapter summaries, code samples, and resource links)._
