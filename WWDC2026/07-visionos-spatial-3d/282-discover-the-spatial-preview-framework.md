# Discover the Spatial Preview framework
**WWDC26 · Session 282** · [Watch](https://developer.apple.com/videos/play/wwdc2026/282/)

_Platforms:_ macOS 27, visionOS 27

## Overview
The Spatial Preview framework is an entirely new macOS framework that creates a live link between a Mac application and Apple Vision Pro, allowing 3D content to be previewed and edited spatially without writing any visionOS code. The framework launches Quick Look on visionOS automatically; the Mac app simply selects an endpoint and starts a session.

The session covers two session types. `DocumentPreviewSession` sends arbitrary file types (USDZ, `.aivu`, spatial photos, etc.) to visionOS — useful for gallery or slideshow workflows. `USDPreviewSession` goes deeper: it accepts a live `USDKit` stage and creates a bidirectional live-edit channel, so changes made on the Mac appear in visionOS instantly, and edits made in visionOS (annotations, object placement) are reflected back in the Mac app via USD stage observation.

The editing features section covers USD layout variants (switching furniture arrangements live), AppleTextAnnotation for authoring 3D sticky notes, per-object manipulation via a `spatialEditable` metadata flag in the USD, session options for playback event forwarding, and a `ProgressReporter`-based API for tracking asset upload progress.

## Key Topics

### Core Architecture
- Mac app does all the work; no visionOS target or code needed.
- Select a device: `ConnectedSpatialEndpointObserver` discovers paired Apple Vision Pro headsets on the local network/USB.
- `SpatialPreviewDevicePicker` SwiftUI sheet for user-facing device selection.
- Start session → content sent → Quick Look opens on visionOS automatically.

### Document Preview Session
- `DocumentPreviewSession(name:contentType:)`: create once, reuse for multiple documents.
- `session.start(endpoint:)` then `session.updateContents(url:)` to push a new file.
- `contentType` supports `.aivu` (Apple Immersive Video), `.usdz`, spatial photos, and more.
- Monitor session lifetime with `session.state.isInvalidated` via async observation.
- `session.close()` ends the session and dismisses the Quick Look view.

### USD Preview Session
- `USDPreviewSession(stage:)`: wraps a `USDKit` `USDStage`.
- `session.start(endpoint:parameters:)` — `.unmodified` parameter opts out of automatic asset optimization.
- `USDPreviewSession.Error.assetUnshareable` thrown when an asset cannot be shared after opting out.
- Changes to the stage (via USDKit) are automatically transmitted to visionOS in real time.

### Editing Features
- **Layout Variants**: define `variantSet "Layout"` in USD; call `prim.variantSets?.setSelection(_:variantName:)` to switch layouts live.
- **Annotations**: `AppleTextAnnotation` USD schema; prims placed under `/__documentAnnotationGroup__`; fields: `text`, `author`, `identifier`.
- **Per-object manipulation**: set `customData.apple.spatialEditable = 1` on any prim to make it draggable in Quick Look.
- **Session options**: `.annotations`, `.perObjectManipulation`, `.export` passed to `session.start`.
- **Playback events**: `session.events` async sequence yields `.timeChanged(time:)` and `.playbackStateChanged(isPlaying:)`.
- **Progress**: `session.progress.fractionCompleted` observable property for upload progress bar.

## APIs & Frameworks

### SpatialPreview (NEW framework)
- `DocumentPreviewSession(name:contentType:)` **[NEW]**
  - `start(endpoint:)` async throws
  - `updateContents(url:)` async throws
  - `close()` async throws
  - `state.isInvalidated` observable
- `USDPreviewSession(stage: USDStage)` **[NEW]**
  - `start(endpoint:parameters:)` — `.unmodified` parameter
  - `USDPreviewSession.Error.assetUnshareable`
  - `events: AsyncSequence` — `.timeChanged`, `.playbackStateChanged`
  - `progress.fractionCompleted`
  - Session options: `.annotations`, `.perObjectManipulation`, `.export`
- `ConnectedSpatialEndpointObserver` **[NEW]**: discovers `SpatialPreviewEndpoint` instances
- `SpatialPreviewEndpoint` **[NEW]**
- `SpatialPreviewDevicePicker(isPresented:onSelect:)` **[NEW]**: SwiftUI device picker view

### USDKit (used together with SpatialPreview)
- `USDStage.open(_:)`, `stage.prim(at:)`
- `prim.variantSets?.setSelection(_:variantName:)`
- `stage.addObserver(for: UsdStage.ObjectsDidChange.self)` — `notice.resyncedPaths`, `notice.stage`
- `SdfPath`, `prim.isAnnotation`

### USD Authoring (schema)
- `AppleTextAnnotation` schema: `text`, `author`, `identifier` string attributes
- `/__documentAnnotationGroup__` hierarchy anchor
- `customData = { dictionary apple = { bool spatialEditable = 1 } }` prim metadata

## Code Highlights

Start a document preview session:
```swift
let previewSession = DocumentPreviewSession(name: "Immersive.aivu", contentType: .aivu)
let endpoint = try await deviceObserver.endpoint
try await previewSession.start(endpoint: endpoint)
try await previewSession.updateContents(url: contentURL)
```

Share and live-edit a USD stage:
```swift
let stage = try USDStage.open(stageURL)
usdSession = USDPreviewSession(stage: stage)
try await usdSession?.start(endpoint: endpoint)
```

Apply a layout variant live:
```swift
let prim = stage.prim(at: SdfPath("/root/furniture"))
try prim.variantSets?.setSelection("Layout", variantName: "LayoutB")
```

Observe and show upload progress:
```swift
for await fraction in Observations({ session.progress.fractionCompleted }) {
    sessionProgress = fraction
}
```

## Takeaways
- Spatial Preview requires zero visionOS code — the entire API lives on the Mac side, making it accessible to any macOS app developer.
- `USDPreviewSession` + USDKit enables a powerful bidirectional workflow: Mac artists can edit a scene and see changes live in the headset, and headset-side annotations feed back into the Mac USD stage.
- The `spatialEditable` USD metadata flag is a minimal one-line addition that unlocks per-object manipulation in Quick Look, useful for furniture/layout review apps.
- Pairing `DocumentPreviewSession.updateContents` with a list UI creates a spatial gallery experience with no additional visionOS code.

---
_Source: WWDC26 Session 282 page (abstract, chapter summaries, code samples, and resource links)._
