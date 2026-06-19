# Discover rolling clips with ReplayKit
**WWDC21 · Session 10101** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10101/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12

## Overview
ReplayKit gains a new **clips recording** mode in iOS 15 and macOS Monterey: a rolling buffer that continuously stores the last 15 seconds of screen and audio samples. When an interesting moment occurs—whether triggered manually by the user or automatically by game logic—the app calls a single export API to save that window as a short video clip. This removes the need to record entire sessions and trim them afterward, and enables in-app highlight reels, automatic moment capture, and personalized clip experiences.

The session covers how clips recording works under the hood, the three-API lifecycle (start/stop/export), integration patterns (UI buttons, game controller support), and considerations for combining with other ReplayKit modes.

## Key Topics

### Clips Recording — Rolling Buffer Model **[NEW in iOS 15]**
- `startClipBuffering()` begins accumulating screen and audio samples in a rolling 15-second buffer.
- New samples continuously replace samples older than 15 seconds; storage overhead is bounded.
- `exportClip(to:duration:)` takes up to 15 seconds of buffered content ending at the call time and writes it to a file URL. Duration is app-specified (up to 15 seconds).
- `stopClipBuffering()` tears down the buffer session and must be called before switching to another ReplayKit mode (broadcast, in-app capture, standard recording).

### Triggering Clips
- **Manual (user-driven)**: show a "Capture Clip" button or gesture; call `exportClip` on tap. User controls exactly when to save.
- **Automatic (app-driven)**: trigger export on game events—perfect combo, boss defeated, personal best, level completed. The buffer always has the preceding 15 seconds ready.
- **Game Controller** (via Game Controller framework): the framework now has built-in clips support; clips saved via the controller go directly to Photos or the Desktop. To build custom in-app experiences, use the ReplayKit API directly alongside the controller.

### Building In-App Clip Experiences
- After export, the app has the clip at a local URL and can:
  - Save to Photos (`PHPhotoLibrary` / `UISaveVideoAtPathToSavedPhotosAlbum`).
  - Present a custom in-app clips viewer or highlight reel.
  - Overlay branding, filters, or metadata before saving.
- KVO on `RPScreenRecorder.isAvailable` and `RPScreenRecorder.isRecording` keeps UI in sync when the recording state changes externally (e.g., via game controller or system interruption).
- Conform to `RPScreenRecorderDelegate` to receive state change callbacks.

### Relationship to Other ReplayKit Modes
- Clips recording is a peer of in-app screen recording, in-app screen capture, and in-app broadcast—not a replacement.
- Only one ReplayKit mode can be active at a time; call `stopClipBuffering()` before switching.
- Same HD quality, low-performance-impact architecture, and built-in privacy safeguards as other ReplayKit modes.

## APIs & Frameworks

**ReplayKit**
- `RPScreenRecorder.shared().startClipBuffering(completionHandler:)` — start rolling buffer **[NEW]**
- `RPScreenRecorder.shared().stopClipBuffering(completionHandler:)` — stop rolling buffer **[NEW]**
- `RPScreenRecorder.shared().exportClip(to:duration:completionHandler:)` — export buffered clip to URL **[NEW]**
- `RPScreenRecorder.isAvailable` (KVO) — whether recording is supported on this device **[existing]**
- `RPScreenRecorder.isRecording` (KVO) — whether a recording session is active **[existing]**
- `RPScreenRecorderDelegate` — protocol for state change callbacks **[existing]**

**Game Controller framework**
- Built-in clips recording integration — game controller clips save to Photos/Desktop **[NEW]**
- Use `RPScreenRecorder` clips API in parallel to retain in-app clip access **[guidance]**

## Code Highlights

Start clip buffering:
```swift
func startClipBuffering() {
    RPScreenRecorder.shared().startClipBuffering { error in
        if error != nil {
            print("Error attempting to start Clip Buffering")
            self.setClipState(active: false)
        } else {
            self.setClipState(active: true)
            self.setupCameraView()
        }
    }
}
```

Stop clip buffering:
```swift
func stopClipBuffering() {
    RPScreenRecorder.shared().stopClipBuffering { error in
        if error != nil {
            print("Error attempting to stop clip buffering")
        }
        self.setClipState(active: false)
        self.tearDownCameraView()
    }
}
```

Export the last 5 seconds as a clip:
```swift
func exportClip() {
    let clipURL = getAppTempDirectory()
    let interval = TimeInterval(5)         // request up to 5 seconds (max 15)

    RPScreenRecorder.shared().exportClip(to: clipURL, duration: interval) { error in
        if error != nil {
            print("Error attempting to export clip")
        } else {
            // Clip is at clipURL — save, display, or upload
            self.saveToPhotos(tempURL: clipURL)
        }
    }
}
```

Guarded export triggered by a UI button:
```swift
@IBAction func exportClipButtonTapped(_ sender: Any) {
    if self.isActive && self.getClipButton.isEnabled {
        exportClip()
    }
}
```

## Takeaways
- Clips recording solves the "I didn't know to start recording" problem: the buffer is always running, so exporting retroactively captures moments that just happened.
- The API footprint is minimal — three calls. All buffering and encoding complexity is handled by ReplayKit internally.
- Automatic in-game triggers (boss defeated, record broken) are the highest-value pattern: they surface clips at the exact moment they're most relevant without user action.
- When using the Game Controller framework's built-in clips feature alongside ReplayKit, implement KVO on `RPScreenRecorder` and `RPScreenRecorderDelegate` to keep the app's recording state synchronized.

---
_Source: WWDC21 Session 10101 page (abstract, full transcript, code samples, and resource links)._
