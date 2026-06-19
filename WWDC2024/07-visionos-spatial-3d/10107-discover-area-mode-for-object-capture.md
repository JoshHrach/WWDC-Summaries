# Discover Area Mode for Object Capture
**WWDC24 · Session 10107** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10107/)

_Platforms:_ iOS 18 (iPhone 15 Pro / iPhone 15 Pro Max and later), macOS 15

## Overview
Object Capture gains a new capture mode in iOS 18: Area Mode. Where the existing Object Mode captures a single discrete object placed on a surface, Area Mode scans an entire spatial region — a room corner, a tabletop scene, a piece of furniture in context — and produces a high-fidelity 3D mesh of the whole area. This unlocks scene-level 3D capture directly from an iPhone camera.

The session walks through the Area Mode data flow: capturing with the on-device `ObjectCaptureSession` API, transferring the resulting image set to a Mac, and running reconstruction with `PhotogrammetrySession`. It also explains new iOS 18 APIs for LiDAR-informed capture guidance and the updated `ObjectCaptureView` SwiftUI interface.

## Key Topics
- **Area Mode vs. Object Mode** — Object Mode targets a single object with turntable-style coverage; Area Mode targets a region of space and guides the user to walk through the area systematically.
- **`ObjectCaptureSession` Area Mode** — a new `.area` capture type enum value on `ObjectCaptureSession.CaptureMode`; the session uses LiDAR depth to compute coverage and guides the user with an in-app overlay.
- **`ObjectCaptureView`** — the SwiftUI view that renders the live camera feed plus the AR guidance overlays (coverage map, uncaptured region highlights); updated for Area Mode.
- **LiDAR-informed coverage** — the session uses iPhone Pro's LiDAR scanner to build a real-time coverage map, shown as a colored mesh overlay; green = captured, blue = in progress, gray = uncaptured.
- **Reconstruction pipeline** — the image set is transferred to a Mac (via Files, AirDrop, or network) and reconstructed with `PhotogrammetrySession`; the reconstructed mesh can be loaded into Reality Composer Pro for staging.
- **Output formats** — `.usdz`, `.obj`, and `.ply` outputs are supported; Area Mode output tends to be larger than Object Mode; use `PhotogrammetrySession.Request.modelFile(url:detail:)` with `.full` detail.

## APIs & Frameworks

**RealityKit (Object Capture — iOS)**
- `ObjectCaptureSession` — existing on-device capture session type
  - **[NEW]** `ObjectCaptureSession.CaptureMode.area` — new Area Mode; requires iPhone 15 Pro or later (LiDAR required)
  - `ObjectCaptureSession.CaptureMode.object` — existing single-object mode; unchanged
  - `ObjectCaptureSession.startDetecting()` — begin object/area detection; unchanged
  - `ObjectCaptureSession.startCapturing()` — begin image acquisition; unchanged
  - `ObjectCaptureSession.finish()` — end capture and finalize the image set
  - `ObjectCaptureSession.state` — `AsyncStream`; observe `.capturing`, `.finishing`, `.completed`, `.failed`
- **[NEW]** `ObjectCaptureView(session:cameraFeedVisibility:)` — SwiftUI view rendering camera + AR overlays; updated for Area Mode with coverage map visualization
- `ObjectCapturePointCloudView` — existing supplementary point cloud preview; unchanged

**RealityKit (Photogrammetry — macOS)**
- `PhotogrammetrySession` — existing macOS reconstruction engine
  - `PhotogrammetrySession(input:configuration:)` — input is the folder of captured images
  - `PhotogrammetrySession.Configuration` — set `sampleOrdering`, `featureSensitivity`
  - **[NEW]** `PhotogrammetrySession.Request.modelFile(url:detail:)` — `.full` detail recommended for Area Mode outputs
  - `PhotogrammetrySession.process([.modelFile(…)])` — run reconstruction
  - `PhotogrammetrySession.Outputs` — async sequence of `.processingComplete`, `.requestComplete`, `.requestError`

## Code Highlights
Start an Area Mode capture session:

```swift
let session = ObjectCaptureSession()
session.start(imagesDirectory: captureURL,
              configuration: .init(checkpointDirectory: checkpointURL))

// Select Area Mode
await session.startDetecting()
// When the user is ready:
await session.startCapturing()
// … user walks through the area guided by ObjectCaptureView …
session.finish()
```

Display the Area Mode AR guidance overlay:

```swift
ObjectCaptureView(session: session)
    .frame(maxWidth: .infinity, maxHeight: .infinity)
```

Reconstruct on Mac:

```swift
let pgSession = try PhotogrammetrySession(input: captureFolder)
let request = PhotogrammetrySession.Request.modelFile(url: outputURL, detail: .full)
try pgSession.process(requests: [request])
for try await output in pgSession.outputs {
    if case .requestComplete(_, let result) = output { print("Done: \(result)") }
}
```

## Takeaways
- Area Mode requires a LiDAR-equipped iPhone (iPhone 15 Pro or later); gate the `.area` mode behind a hardware capability check with `ObjectCaptureSession.isSupported`.
- The LiDAR coverage overlay is the primary UX for guiding non-expert users through the capture; do not hide or replace it — it is the key quality driver for Area Mode.
- Area Mode output meshes are significantly larger than Object Mode meshes; set `detail: .full` only when downstream use requires it, and offer `.reduced` or `.preview` outputs for in-app previews.
- Use Reality Composer Pro to stage and optimize the reconstructed area mesh before shipping it in a visionOS or AR Quick Look experience.

---
_Source: WWDC24 Session 10107 page (abstract, chapter summaries, code samples, and resource links)._
