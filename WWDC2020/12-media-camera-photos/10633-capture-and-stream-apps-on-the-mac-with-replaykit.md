# Capture and Stream Apps on the Mac with ReplayKit
**WWDC20 · Session 10633** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10633/)

_Platforms:_ macOS Big Sur 11, iOS 14, tvOS 14

## Overview
ReplayKit comes to macOS in Big Sur, bringing the same three core capabilities available on iOS and tvOS — in-app screen recording, in-app screen capture, and in-app live broadcast — to Mac apps and games. The session demonstrates all three modes through a single Xcode project, shows macOS-specific API differences (particularly for broadcast, where `RPBroadcastActivityViewController` is replaced by `RPBroadcastActivityController` with a `showBroadcastPicker(at:from:preferredExtension:)` method), and covers two new cross-platform features: `stopRecording(withOutput:completionHandler:)` for direct file access, and Game Controller framework integration that triggers recording from physical controller buttons.

A menu bar indicator appears whenever a ReplayKit session is active on macOS, allowing users to terminate the session at any time. Apps must handle `RPScreenRecorderDelegate.screenRecorder(_:didStopRecordingWith:previewViewController:error:)` to update their state when the user stops recording from the menu bar.

## Key Topics

**In-App Screen Recording (macOS)**
`RPScreenRecorder.shared().startRecording(handler:)` begins capturing screen, audio, and microphone into a movie file managed by ReplayKit. `stopRecording(handler:)` saves the movie and delivers an `RPPreviewViewController` for editing/saving via the macOS file save sheet flow. No code change needed from iOS — the API is identical.

**New: `stopRecording(withOutput:completionHandler:)` — Direct File Access**
A new API across all platforms that writes the recorded movie directly to a developer-specified URL. The app gets read access to the file, enabling custom video replay, management, or editing flows inside the app. Replaces the `RPPreviewViewController` workflow when direct file access is desired.

**In-App Screen Capture (macOS)**
`RPScreenRecorder.shared().startCapture(handler:completionHandler:)` delivers raw `CMSampleBuffer` samples to the app's process rather than writing a movie file. The handler is called continuously for each video, audio, and microphone sample. `RPSampleBufferType` enum distinguishes between `.video`, `.audioApp`, and `.audioMic`. The app is responsible for consuming, encoding, or rendering these samples.

**In-App Live Broadcast — macOS Differences**
On macOS, `RPBroadcastActivityViewController` (iOS) is replaced by `RPBroadcastActivityController`. Presenting the service picker uses:
`RPBroadcastActivityController.showBroadcastPicker(at:from:preferredExtension:activityDelegate:)`
where the `CGPoint` is the picker's origin (bottom-left of the picker popover), and `from:` is the `NSWindow`. After the delegate callback delivers an `RPBroadcastController`, call `startBroadcast(handler:)` to begin streaming. `finishBroadcast(completionHandler:)` ends it.

**macOS Menu Bar Integration**
A new menu bar icon appears during any active ReplayKit session (recording, capture, or broadcast). Users can tap it to stop the session. Apps must observe `RPScreenRecorder.isRecording` via KVO and implement `RPScreenRecorderDelegate.screenRecorder(_:didStopRecordingWith:previewViewController:error:)` to update UI when the user terminates a session from the menu bar.

**Game Controller Integration (New, all platforms)**
The Game Controller framework now has built-in ReplayKit support. Double-tapping the Share button on a PS4 controller (or the Select button on an Xbox controller) starts in-app recording; double-tapping again stops it and saves to Photos (iOS) or the Desktop (macOS). Apps should KVO-observe `RPScreenRecorder.isAvailable` and `RPScreenRecorder.isRecording`, and implement `didStopRecordingWith:previewViewController:` to respond to controller-triggered stop events.

## APIs & Frameworks

### ReplayKit **[macOS NEW in Big Sur]**
- `RPScreenRecorder.shared()` — singleton; main entry point for all recording/capture
- `RPScreenRecorder.startRecording(handler:)` — starts in-app screen recording
- `RPScreenRecorder.stopRecording(handler:)` — stops recording; delivers `RPPreviewViewController`
- `RPScreenRecorder.stopRecording(withOutput url: URL, completionHandler:)` **[NEW]** — stops recording and writes movie to specified URL; gives app direct file access
- `RPScreenRecorder.startCapture(handler:completionHandler:)` — starts raw sample capture
  - `handler: (CMSampleBuffer, RPSampleBufferType, Error?) -> Void` — called per sample
  - `completionHandler: (Error?) -> Void` — called once on start
- `RPScreenRecorder.stopCapture(handler:)` — stops raw capture
- `RPSampleBufferType` — `.video`, `.audioApp`, `.audioMic`
- `RPPreviewViewController` — presents edit/save/share UI for recordings
- `RPPreviewViewControllerDelegate.previewControllerDidFinish(_:)` — called when user dismisses preview

### ReplayKit — Broadcast (iOS)
- `RPBroadcastActivityViewController` — iOS sheet for selecting broadcast service
- `RPBroadcastActivityViewControllerDelegate.broadcastActivityViewController(_:didFinishWith:error:)` — delivers `RPBroadcastController`

### ReplayKit — Broadcast (macOS) **[NEW]**
- `RPBroadcastActivityController` **[NEW]** — macOS replacement for `RPBroadcastActivityViewController`
- `RPBroadcastActivityController.showBroadcastPicker(at:from:preferredExtension:activityDelegate:)` **[NEW]** — shows broadcast service picker at a `CGPoint` origin relative to a given `NSWindow`
- `RPBroadcastActivityControllerDelegate.broadcastActivityController(_:didFinishWith:error:)` **[NEW]** — delivers `RPBroadcastController`
- `RPBroadcastController.startBroadcast(handler:)` — begins the broadcast
- `RPBroadcastController.finishBroadcast(completionHandler:)` — ends the broadcast
- `RPBroadcastController.pauseBroadcast()` / `.resumeBroadcast()` — pause/resume

### ReplayKit — Delegate
- `RPScreenRecorderDelegate` — protocol for handling unexpected session stops
- `screenRecorder(_:didStopRecordingWith:previewViewController:error:)` — called when session ends (including via menu bar icon or game controller); `previewViewController` is `nil` for capture/broadcast stops
- KVO properties: `RPScreenRecorder.isRecording`, `RPScreenRecorder.isAvailable`, `RPScreenRecorder.isCameraEnabled`, `RPScreenRecorder.isMicrophoneEnabled`

### Game Controller Framework (New Integration)
- `GCController` — game controller; Share (PS4) or Select (Xbox) button double-tap triggers ReplayKit recording start/stop
- No API required; behavior is automatic when ReplayKit is available
- Must observe `RPScreenRecorder.isRecording` via KVO to respond to controller-triggered events

## Code Highlights

Starting and stopping in-app recording:
```swift
let recorder = RPScreenRecorder.shared()

// Start
recorder.startRecording { error in
    if let error = error { print(error); return }
    self.updateRecordingState(isRecording: true)
}

// Stop with preview
recorder.stopRecording { previewVC, error in
    guard let previewVC = previewVC, error == nil else { return }
    previewVC.previewControllerDelegate = self
    NSApp.keyWindow?.beginSheet(previewVC)
}
```

Stopping with direct file output (new API):
```swift
let outputURL = FileManager.default.temporaryDirectory
    .appendingPathComponent("recording.mov")
recorder.stopRecording(withOutput: outputURL) { error in
    if let error = error { print(error); return }
    // outputURL now contains the recorded movie
    self.openCustomEditor(for: outputURL)
}
```

In-app capture handling samples:
```swift
recorder.startCapture(handler: { sampleBuffer, sampleType, error in
    guard error == nil else { return }
    switch sampleType {
    case .video:     self.processVideoSample(sampleBuffer)
    case .audioApp:  self.processAppAudioSample(sampleBuffer)
    case .audioMic:  self.processMicSample(sampleBuffer)
    @unknown default: break
    }
}, completionHandler: { error in
    if let error = error { print(error); return }
    self.updateCaptureState(isCapturing: true)
})
```

macOS live broadcast:
```swift
RPBroadcastActivityController.showBroadcastPicker(
    at: CGPoint(x: 0, y: window.frame.height),
    from: window,
    preferredExtension: nil,
    activityDelegate: self
)

// Delegate:
func broadcastActivityController(_ controller: RPBroadcastActivityController,
                                  didFinishWith broadcastController: RPBroadcastController?,
                                  error: Error?) {
    guard let broadcastController = broadcastController, error == nil else { return }
    self.broadcastController = broadcastController
    broadcastController.startBroadcast { error in
        if let error = error { print(error); return }
        self.updateBroadcastState(isBroadcasting: true)
    }
}
```

## Takeaways
- ReplayKit on macOS Big Sur brings all three modes (recording, capture, broadcast) from iOS/tvOS; most API is identical, with the key macOS difference being `RPBroadcastActivityController` replacing `RPBroadcastActivityViewController` and using `showBroadcastPicker(at:from:)` instead of a view controller presentation.
- Use `stopRecording(withOutput:completionHandler:)` when the app needs to incorporate recorded videos (custom editors, replay galleries) — the app gets direct filesystem access to the movie file rather than routing through the preview view controller.
- Always implement `RPScreenRecorderDelegate.screenRecorder(_:didStopRecordingWith:previewViewController:error:)` and KVO-observe `isRecording` on macOS to handle user-initiated session termination via the menu bar icon.
- On iOS, tvOS, and macOS, double-tapping the PS4 Share or Xbox Select controller button now automatically starts/stops in-app recording; apps must observe `RPScreenRecorder.isRecording` via KVO to stay in sync with controller-triggered events.

---
_Source: WWDC20 Session 10633 page (transcript and resource links)._
