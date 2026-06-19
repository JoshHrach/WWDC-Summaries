# Support Cinematic mode videos in your app
**WWDC23 · Session 10137** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10137/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, tvOS 17

## Overview
This session introduces the Cinematic framework (CN prefix), a new API that lets third-party apps read, play, edit, and re-render Cinematic mode videos captured on iPhone 13 and 14. Cinematic mode records a multi-track asset containing a video track, a disparity (depth) track, and a metadata track. The metadata track holds the Cinematic script — all on-device subject detections and focus decisions — which drives real-time shallow depth-of-field rendering. The new API exposes every layer of this pipeline so apps can replicate and extend what Photos, iMovie, Final Cut Pro, and Motion do.

The session walks through building a playback and editing app: fetching a Cinematic asset from Photos, setting up a custom AVFoundation video compositor that calls `CNRenderingSession`, reading and modifying `CNScript` focus decisions, overlaying detection boxes, and saving/loading script changes as a compact binary file.

## Key Topics

### Cinematic Asset Structure
A Cinematic mode capture produces two files: a baked rendered asset (shareable QuickTime movie) and the full Cinematic asset (multi-track). The Cinematic asset contains:
- **Video track** — original QuickTime movie; HDR/SDR, 1080p@30fps or 4K@24/25/30fps (iPhone 14)
- **Disparity track** — lower-resolution relative disparity map (pixel shift between two cameras)
- **Metadata track** — rendering attributes (focus disparity, aperture f-number) + Cinematic script (all detections and focus decisions)

### Rendering Pipeline
Data flows: Cinematic asset → optional nondestructive edits → `CNRenderingSession` (GPU-accelerated depth-of-field compositor) → rendered output. The rendering session takes focus disparity and aperture as inputs and applies the shallow-depth-of-field effect using the disparity map.

### Custom Video Compositor
To play back a Cinematic asset with real-time rendering, an `AVVideoCompositing`-conforming class is used. Inside `startRequest(_:)`:
1. Extract source buffers for video, disparity, and metadata tracks using `CNCompositionInfo` track IDs.
2. Get `CNRenderingFrameAttributes` from the metadata buffer via `CNRenderingSession`.
3. Optionally modify frame attributes (aperture, focus disparity).
4. Encode a render command on a Metal command queue and commit.

### CNScript — Detections and Decisions
`CNScript` holds all `CNDetection` objects (face, head, torso, cat, dog, ball) grouped by a persistent `detectionGroupID`, and `CNDecision` objects that assign focus to a detection at a point in time.

Three decision types:
- **Base decisions** — automatic, generated during capture
- **Weak user decisions** — override until the next base or user decision
- **Strong user decisions** — hold focus on a subject as long as the detection track exists, overriding any subsequent base decisions

`CNScript` supports adding user decisions via `addUserDecision(_:)`. Focus transitions (racking) between decisions are computed ahead of time by the Cinematic engine, accessible as focus disparity per frame.

### Saving and Loading Script Changes
Script changes are serialized to a compact binary format:
- `cnScript.changes()` → `CNScriptChanges`
- `cnScriptChanges.makeData()` → `Data` (write to file)
- `CNScriptChanges(data:)` → load from file
- `cnScript.reload(changes:)` → apply loaded changes

### CNObjectTracker
For objects not automatically detected, `CNObjectTracker` can be engaged by tapping a point on a frame. It produces a detection track that can be added to the script, enabling custom focus subjects.

## APIs & Frameworks

- **Cinematic** framework (new, `CN` prefix) **[NEW]**
  - `CNAssetInfo` — wraps a Cinematic AVAsset, provides track references
  - `CNCompositionInfo` — like CNAssetInfo but for AVComposition tracks
  - `CNRenderingSession` — GPU rendering session for depth-of-field effect **[NEW]**
    - `init(commandQueue:renderingAttributes:quality:)` — set up session
    - `CNRenderingQuality` — `.preview`, `.export`, etc.
    - `encodeRender(to:commandBuffer:sourceImage:sourceDisparity:frameAttributes:)` — encode render on GPU
    - `frameAttributes(for:)` — extract `CNRenderingFrameAttributes` from metadata buffer
  - `CNRenderingAttributes` — focus disparity and aperture from asset
    - `CNAssetInfo.renderingAttributes` — fetch from asset
  - `CNRenderingFrameAttributes` — per-frame rendering attributes (focus disparity, aperture)
    - `focusDisparity: Float` — mutable; change to redirect focus
    - `fNumber: Float` — mutable; change to alter bokeh amount
  - `CNScript` — holds all detections and focus decisions **[NEW]**
    - `CNAssetInfo.script` — load from asset
    - `frame(at:)` — get `CNScriptFrame` for a time
    - `addUserDecision(_:)` — add a weak or strong user focus decision
    - `changes()` → `CNScriptChanges` — extract delta for serialization
    - `reload(changes:)` — apply loaded changes
  - `CNScriptFrame` — detections and focus decision at a point in time
    - `detections: [CNDetection]` — all detected subjects
    - `focusDetection: CNDetection?` — currently focused subject
  - `CNDetection` — a detected subject
    - `detectionID`, `detectionGroupID` — tracking identifiers
    - `normalizedRect: CGRect` — bounding box in normalized coordinates
    - `detectionType` — face, head, torso, cat, dog, ball
  - `CNDecision` — assigns focus to a detection
    - `CNDecisionStrength` — `.weak`, `.strong`
    - `CNDecision(detectionID:at:strong:)` — create user decision
  - `CNScriptChanges` — serializable delta of script edits
    - `makeData()` → `Data`
    - `init(data:)`
  - `CNObjectTracker` — tracks arbitrary objects not auto-detected **[NEW]**
- **AVFoundation**
  - `AVAsset` — base video asset
  - `AVPlayerItem` — playback item
  - `AVVideoCompositing` protocol — custom compositor interface
    - `startRequest(_:)` — per-frame composition callback
  - `AVVideoCompositionRequest` — provides source pixel buffers; `finish(withComposedVideoFrame:)`
  - `AVMutableComposition` — composition for multi-track assets
  - `AVMutableVideoComposition` — video composition with custom compositor
  - `AVAssetExportSession` — offline export of rendered video
- **Photos / PhotosUI**
  - `PHPickerViewController` / `PHPickerFilter.cinematicVideos` — pick Cinematic mode assets from library
  - `PHAsset` — represents the asset in Photos library
  - `PHImageManager.requestAVAsset(forVideo:options:resultHandler:)` — fetch the `AVAsset`
  - `PHVideoRequestOptions` — set `.version = .original`, `isNetworkAccessAllowed = true`
- **Metal**
  - `MTLCommandQueue` — required by `CNRenderingSession` for GPU rendering

## Code Highlights

Setting up the rendering session:
```swift
let renderingAttributes = try CNAssetInfo.renderingAttributes(from: cinematicAsset)
let renderingSession = try CNRenderingSession(
    commandQueue: metalCommandQueue,
    renderingAttributes: renderingAttributes,
    quality: .export
)
```

Extracting and modifying frame attributes inside the custom compositor:
```swift
let frameAttributes = try renderingSession.frameAttributes(for: metadataBuffer)
frameAttributes.fNumber = userSelectedAperture     // change bokeh
// or:
frameAttributes.focusDisparity = scriptFrame.focusDetection?.disparity ?? frameAttributes.focusDisparity
```

Adding a user focus decision:
```swift
let decision = CNDecision(detectionID: tappedDetection.detectionID, at: currentTime, strong: isDoubleTap)
cnScript.addUserDecision(decision)
```

Saving and loading script changes:
```swift
// Save
let changes = cnScript.changes()
let data = try changes.makeData()
try data.write(to: changesFileURL)

// Load
let data = try Data(contentsOf: changesFileURL)
let changes = try CNScriptChanges(data: data)
cnScript.reload(changes: changes)
```

## Takeaways

- The Cinematic framework (`CN` prefix) is the complete API for playing, editing, and re-rendering Cinematic mode videos — previously only available to Photos, iMovie, and Final Cut Pro.
- Integration requires a custom `AVVideoCompositing` class that extracts the three Cinematic tracks per frame, calls `CNRenderingSession` on the GPU, and returns the rendered output — aperture and focus disparity can be modified per frame before rendering.
- `CNScript` contains all detections and decisions; add `CNDecision` objects (weak or strong) to redirect focus to any detection at any time, and the Cinematic engine computes smooth rack-focus transitions automatically.
- Script changes serialize to a compact binary format (`CNScriptChanges.makeData()`) for nondestructive storage — the original asset is never modified.

---
_Source: WWDC23 Session 10137 page (abstract, chapter summaries, and resource links)._
