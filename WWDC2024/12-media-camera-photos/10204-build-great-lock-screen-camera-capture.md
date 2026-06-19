# Build a Great Lock Screen Camera Capture Experience
**WWDC24 · Session 10204** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10204/)

_Platforms:_ iOS 18 (LockedCameraCapture framework; requires device with Face ID or Touch ID; does not run on Simulator)

## Overview
iOS 18 introduces the `LockedCameraCapture` framework, which enables a new type of app extension that runs directly from the iOS Lock Screen camera button—without requiring the user to unlock the device. This session explains the architecture of a Locked Camera Capture Extension, how it differs from traditional camera apps, the privacy and security constraints imposed by the OS, and how to build a great capture experience within those constraints.

A Locked Camera Capture Extension runs in a sandboxed process with access only to the camera and microphone, no network access, and no access to the Photos library while locked. The captured media is held in a temporary session store and transferred to the main app when the user unlocks the device. The session covers the full lifecycle: extension launch, camera session setup, capture, session data handoff, and main app processing.

## Key Topics
- **Locked Camera Capture Extension** — new iOS 18 extension point; launched from Lock Screen camera button; no unlock required
- **`LockedCameraCapture` framework** — core framework; `CameraCaptureIntent`, `CameraCaptureExtension`, `CameraCaptureSession`
- **Security constraints** — no network access, no Photos library access, no keychain, no persistent storage while locked; OS-enforced
- **Session data handoff** — captured media stored in `CameraCaptureSessionStore`; transferred to main app on unlock via `sessionContentURL`
- **SwiftUI + AVFoundation camera setup** — extension UI built in SwiftUI; camera access via `AVCaptureSession` (same as standard camera code)
- **Main app integration** — receives `CameraCaptureSession` data via `onCameraCaptureSessionActivated` scene modifier; processes and saves to Photos after unlock

## APIs & Frameworks
### LockedCameraCapture (iOS 18) — [NEW framework]
- **`CameraCaptureExtension`** — entry point protocol for the extension; `body: some View` returns the extension's SwiftUI view
- **`CameraCaptureIntent`** — `AppIntent`-based intent; system invokes this to launch the extension from the Lock Screen
- **`CameraCaptureSession`** — represents an active capture session; provides `sessionContentURL` for writing captured media
  - `sessionContentURL: URL` — temporary URL to write captured photos/videos; content survives until processed by the main app
  - `isLocked: Bool` — `true` if the device is currently locked
  - `invalidationReason: CameraCaptureSession.InvalidationReason?` — why the session ended (user dismissed, timeout, etc.)
- **`CameraCaptureSessionStore`** — manages active sessions; `sessions` property lists pending sessions awaiting main app processing
- **`onCameraCaptureSessionActivated(_:)`** — SwiftUI scene modifier on the main app; closure receives `CameraCaptureSession` when the user unlocks and a session is pending

### AVFoundation (within the extension)
- `AVCaptureSession` — standard camera session; configure as normal for photo/video capture
- `AVCapturePhotoOutput` / `AVCaptureMovieFileOutput` — capture outputs; write to `sessionContentURL`
- `AVCaptureDevice` — camera device selection; `.default(.builtInWideAngleCamera, for: .video, position: .back)`
- Note: `PHPhotoLibrary` is **not accessible** within the locked extension; media must be written to `sessionContentURL`

### Photos
- `PHPhotoLibrary.shared().performChanges` — called in the main app (after unlock) to save media from `sessionContentURL` to the Photos library

## Code Highlights
```swift
// Extension entry point
import LockedCameraCapture
import SwiftUI

struct MyCameraExtension: CameraCaptureExtension {
    var body: some View {
        MyCaptureView()
    }
}

// Capture view — standard SwiftUI + AVFoundation
struct MyCaptureView: View {
    @Environment(\.cameraCaptureSession) var session

    var body: some View {
        CameraPreview()
            .overlay(alignment: .bottom) {
                Button("Capture") { capturePhoto() }
                    .padding()
            }
    }

    func capturePhoto() {
        guard let session else { return }
        let outputURL = session.sessionContentURL.appending(component: "photo.jpg")
        // Write captured image data to outputURL
        // AVCapturePhotoOutput delegate writes to this URL
    }
}

// Main app — receive session on unlock
@main
struct MyApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .onCameraCaptureSessionActivated { session in
            // Process pending captured media
            let capturedFiles = try? FileManager.default
                .contentsOfDirectory(at: session.sessionContentURL, ...)
            // Save to Photos library
            PHPhotoLibrary.shared().performChanges { ... }
        }
    }
}
```

## Takeaways
- Locked Camera Capture Extensions run entirely on-device with strong OS-enforced privacy guarantees—no network, no Photos library, no keychain while locked; design your UX around these constraints from the start
- Media must be written to `CameraCaptureSession.sessionContentURL`, not the app's documents directory or Photos library, while the device is locked; the handoff to the main app happens automatically on unlock
- The `onCameraCaptureSessionActivated` scene modifier is the single integration point in the main app; process and save all pending session media there
- Test on a physical device—this extension type does not run in Simulator, and the Lock Screen camera button behavior requires real hardware

---
_Source: WWDC24 Session 10204 page (abstract, chapter summaries, code samples, and resource links)._
