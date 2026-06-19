# Meet Object Capture for iOS
**WWDC23 · Session 10191** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10191/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14

## Overview
Object Capture, previously available only as a macOS reconstruction API, now comes to iOS with a complete end-to-end experience: guided image capture and on-device 3D model reconstruction all in the palm of your hand. The new `ObjectCaptureSession` manages a state machine through initializing, ready, detecting, capturing, finishing, and completed states, while `ObjectCaptureView` automatically adapts its camera UI to each state. Reconstruction runs locally using `PhotogrammetrySession`, outputting a USDZ model in the `reduced` detail level optimized for mobile display.

A key improvement this year is LiDAR-assisted scanning. By fusing RGB photos with LiDAR point cloud data, Object Capture can now reconstruct low-texture objects—such as plain furniture or ceramic objects—that previously could not be reliably captured. The accompanying guided capture feedback system alerts users about lighting, motion blur, distance, and field-of-view issues in real time.

Apple also provides full sample app source code showing a complete capture-and-reconstruct workflow that developers can download, run immediately, and use as a starting point for their own applications.

## Key Topics

### LiDAR Support for Low-Texture Objects
- RGB images alone cannot recover geometry for textureless surfaces; LiDAR point cloud data fills the gap
- Fused RGB + LiDAR produces complete, dense 3D geometry
- Supported devices: iPhone 12 Pro, iPad Pro 2021, and later; verify with `ObjectCaptureSession.isSupported`
- Avoid reflective, transparent, or very thin-structure objects

### Guided Capture
- Automatic image capture as users orbit the object; no manual shutter required
- Capture dial shows which angular regions have sufficient coverage
- Real-time feedback: lighting conditions, motion speed, camera distance, field-of-view warnings
- Flip detection API (`ObjectCaptureSession.FeedbackMessages`) guides users on whether to flip the object
- `isOverCaptureEnabled` configuration flag allows capturing more images than on-device reconstruction uses, storing them for later Mac reconstruction

### Scan Pass Workflow
- Three scan passes recommended for complete coverage
- `userCompletedScanPass` property signals when the capture dial is full
- `beginNewScanPass()` – continues in same orientation at different height
- `beginNewScanPassAfterFlip()` – returns to ready state for bounding box re-detection in new orientation
- `finish()` – transitions to finishing, then automatically to completed

### On-Device Reconstruction (iOS)
- `PhotogrammetrySession` now runs on iOS **[NEW]**
- Only `reduced` detail level supported on iOS (diffuse, ambient occlusion, and normal texture maps)
- Checkpoint directory can be shared between `ObjectCaptureSession` and `PhotogrammetrySession` to speed up reconstruction
- Higher detail levels (`medium`, `full`, `raw`, `custom`) require Mac reconstruction

### macOS Reconstruction Enhancements
- Mac reconstruction now uses LiDAR data saved in captured images
- Estimated reconstruction time added to progress output **[NEW]**
- New `custom` detail level **[NEW]**: control mesh decimation amount, texture map resolution, format, and which maps to include
- Pose output: `PhotogrammetrySession.Output.poses` returns high-quality estimated camera position and orientation for each image **[NEW]**
- Reality Composer Pro integrates Object Capture for Mac reconstruction without writing code

## APIs & Frameworks

- **RealityKit** – framework containing Object Capture and reconstruction APIs
- `ObjectCaptureSession` **[NEW on iOS]** – reference-type session managing capture state machine
  - `ObjectCaptureSession.Configuration` – `checkpointDirectory`, `isOverCaptureEnabled` **[NEW]**
  - `ObjectCaptureSession.State` – `.initializing`, `.ready`, `.detecting`, `.capturing`, `.finishing`, `.completed`, `.failed`
  - `start(imagesDirectory:configuration:)` – begins capture, moves to `.ready`
  - `startDetecting()` – moves to `.detecting`; shows bounding box
  - `startCapturing()` – moves to `.capturing`; begins auto-capture
  - `beginNewScanPass()` – resets capture dial, stays in `.capturing`
  - `beginNewScanPassAfterFlip()` – returns to `.ready` for new orientation
  - `finish()` – moves to `.finishing`, then `.completed`
  - `userCompletedScanPass` – `Bool` property; true when capture dial is full
  - `isSupported` – static property to check device LiDAR support **[NEW]**
- `ObjectCaptureView` **[NEW on iOS]** – SwiftUI view; adapts UI to session state automatically
- `ObjectCapturePointCloudView` **[NEW on iOS]** – SwiftUI view; displays captured point cloud for preview; pauses capture session
- `PhotogrammetrySession` – reconstruction session (previously Mac-only, now iOS)
  - `PhotogrammetrySession.Configuration` – `checkpointDirectory`
  - `process(requests:)` – initiates reconstruction
  - `PhotogrammetrySession.Request.modelFile(url:)` – request a USDZ model file
  - `PhotogrammetrySession.Request.poses` **[NEW]** – request camera pose data
  - `PhotogrammetrySession.Output.processingComplete` – signals reconstruction done
  - `PhotogrammetrySession.Output.poses(_:)` **[NEW]** – returns estimated camera poses
  - `PhotogrammetrySession.Detail.reduced` – mobile-optimized detail level (iOS supported)
  - `PhotogrammetrySession.Detail.custom` **[NEW on macOS]** – fully configurable mesh and texture settings

## Code Highlights

Session instantiation and start:
```swift
import RealityKit
import SwiftUI

var session = ObjectCaptureSession()

var configuration = ObjectCaptureSession.Configuration()
configuration.checkpointDirectory = getDocumentsDir().appendingPathComponent("Snapshots/")

session.start(imagesDirectory: getDocumentsDir().appendingPathComponent("Images/"),
              configuration: configuration)
```

Layering state-driven buttons over `ObjectCaptureView`:
```swift
struct CapturePrimaryView: View {
    var body: some View {
        ZStack {
            ObjectCaptureView(session: session)
            if case .ready = session.state {
                CreateButton(label: "Continue") { session.startDetecting() }
            } else if case .detecting = session.state {
                CreateButton(label: "Start Capture") { session.startCapturing() }
            }
        }
    }
}
```

On-device reconstruction:
```swift
.task {
    var configuration = PhotogrammetrySession.Configuration()
    configuration.checkpointDirectory = getDocumentsDir().appendingPathComponent("Snapshots/")
    let session = try PhotogrammetrySession(
        input: getDocumentsDir().appendingPathComponent("Images/"),
        configuration: configuration)
    try session.process(requests: [
        .modelFile(url: getDocumentsDir().appendingPathComponent("model.usdz"))
    ])
    for try await output in session.outputs {
        switch output {
        case .processingComplete: handleComplete()
        default: break
        }
    }
}
```

## Takeaways
- Object Capture is now a complete on-device workflow on iOS: capture with `ObjectCaptureSession` + `ObjectCaptureView`, then reconstruct locally with `PhotogrammetrySession`.
- LiDAR integration dramatically expands the range of scannable objects by supplementing RGB images with depth point cloud data.
- `ObjectCaptureView` is a UI-less SwiftUI component — add your own overlays on top to match your app's design.
- For high-fidelity or large-scale scans, use `isOverCaptureEnabled = true` to store extra images for later Mac reconstruction at higher detail levels.

---
_Source: WWDC23 Session 10191 page (abstract, chapter summaries, code samples, and resource links)._
