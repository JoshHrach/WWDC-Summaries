# Discover Continuity Camera for tvOS
**WWDC23 · Session 10256** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10256/)

_Platforms:_ tvOS 17

## Overview
tvOS 17 brings Continuity Camera and Mic to Apple TV for the first time, enabling iPhone and iPad to serve as camera and microphone inputs on the big screen. Engineers Kevin Tulod (tvOS) and Somesh Ganesh (Core Audio) explain how to adopt the new Device Discovery API, use AVFoundation capture classes on tvOS, and handle the unique multi-user and storage characteristics of Apple TV.

This session opens new genres of tvOS apps — video conferencing, content creation, live streaming, and fitness experiences — that were previously impossible without a built-in camera. The key insight is that most iOS camera code "just works" on tvOS, with the primary addition being Device Discovery to handle the absence of a built-in camera.

## Key Topics

### Continuity Camera Overview
iPhone and iPad can be used as a wireless camera and microphone on Apple TV, similar to how Continuity Camera works on macOS (introduced in macOS Ventura). Only one Continuity Camera can be connected at a time. Apple TV is a communal device, so any user (or guest) can connect their compatible device.

### AVFoundation Capture APIs on tvOS **[NEW]**
`AVCaptureDevice`, `AVCaptureDeviceInput`, `AVCaptureSession`, `AVCaptureOutput` subclasses, `AVCaptureVideoPreviewLayer`, and `AVCaptureConnection` are all now available on tvOS 17. Existing iOS camera code largely runs unchanged on tvOS.

### Device Discovery API **[NEW]**
Since Apple TV has no built-in camera, apps must handle devices appearing and disappearing dynamically.

- **`AVContinuityDevicePickerViewController`** (UIKit) — presents a UI listing signed-in users and a guest pairing option. Delegate receives an `AVContinuityDevice` when a device is connected. **[NEW]**
- **`.continuityDevicePicker(_:isPresented:onDidConnect:)`** (SwiftUI modifier) — presents the picker when the bound state variable is true; callback receives an `AVContinuityDevice`. **[NEW]**
- **`AVContinuityDevice`** — represents a connected iPhone/iPad; has `captureDevices` and `audioSessionPorts` properties. **[NEW]**
- **`AVCaptureDevice.systemPreferredCamera`** — existing API, now available on tvOS; `nil` = no camera connected, non-nil = Continuity Camera connected. Monitor via KVO.
- On tvOS, all capture devices are of type `.continuityCamera`.

**Recommended flow:**
1. Check if `systemPreferredCamera` is non-nil → start session immediately.
2. If nil → present device picker.
3. KVO `systemPreferredCamera` to detect connect/disconnect at runtime.

### tvOS App Considerations
- **No touch input**: Users interact via remote with the focus engine (swipes, directional presses, select button).
- **Multi-user / communal device**: Handle personal information differently; guests can also connect their devices.
- **File storage**: `.documentDirectory` is **not available** on tvOS (causes runtime error). Use only `.cachesDirectory` for temporary storage. Offload recorded content to the cloud (e.g., CloudKit) promptly and delete local copies.

### Microphone APIs on tvOS **[NEW]**
Full AVFAudio and AudioToolbox recording APIs are now available on tvOS 17.

**Audio Session additions for tvOS:**
- `AVAudioSession` categories/modes supporting input: `.playAndRecord`, `.voiceChat`, `.videoChat` **[NEW on tvOS]**
- `AVAudioSessionPort` — new port type: `.continuityMicrophone` for Continuity Mic identification **[NEW]**
- `AVAudioSession.inputAvailable` — KVO-observable on tvOS to detect mic connect/disconnect **[NEW on tvOS]**
- `AVAudioSession.requestRecordPermission(_:)` — now available on tvOS **[NEW]**

**Recording API options:**
1. `AVAudioRecorder` — simplest; write to file, configurable format/sample rate.
2. `AVCapture` (AVCaptureAudioDataOutput) — for combined camera + mic workflows.
3. `AVAudioEngine` — complex processing, karaoke mixing, real-time callbacks via `AVAudioSinkNode` / `AVAudioSourceNode`.
4. `AudioQueue` — non-real-time, low-level.
5. `AudioUnit` — `AU RemoteIO` / `AU VoiceIO` for real-time I/O cycle access.

**Voice Processing and Echo Cancellation (tvOS) **[NEW]**:**
tvOS 17 introduces echo cancellation technology specifically designed for the Apple TV setup (mic on iPhone, playback on potentially loud external speakers many feet away). Enable it via:
- `AVAudioEngine.inputNode.setVoiceProcessingEnabled(true)` **[NEW on tvOS]**
- Or via `AU VoiceIO` audio unit (VoiceProcessingIO subtype)

## APIs & Frameworks

### AVFoundation (Camera — new on tvOS)
- `AVCaptureDevice` — represents a camera/mic device **[NEW on tvOS]**
- `AVCaptureDevice.systemPreferredCamera` — KVO-observable best available camera **[NEW on tvOS]**
- `AVCaptureDeviceInput` **[NEW on tvOS]**
- `AVCaptureSession` **[NEW on tvOS]**
- `AVCaptureVideoPreviewLayer` **[NEW on tvOS]**
- `AVCapturePhotoOutput` **[NEW on tvOS]**
- `AVCaptureMovieFileOutput` **[NEW on tvOS]**
- `AVCaptureMetadataOutput` — face/body detection metadata **[NEW on tvOS]**
- `AVCaptureConnection` **[NEW on tvOS]**

### AVKit (Device Discovery)
- `AVContinuityDevicePickerViewController` — UIKit device picker **[NEW]**
- `AVContinuityDevicePickerViewControllerDelegate` — callbacks for device selection **[NEW]**
- `.continuityDevicePicker(_:isPresented:onDidConnect:)` — SwiftUI view modifier **[NEW]**
- `AVContinuityDevice` — connected device reference with `captureDevices` and `audioSessionPorts` **[NEW]**

### AVFAudio (Microphone — new on tvOS)
- `AVAudioSession` — now supports recording categories/modes on tvOS **[NEW on tvOS]**
- `AVAudioSession.inputAvailable` — KVO-observable mic availability **[NEW on tvOS]**
- `AVAudioSessionPort.continuityMicrophone` — new port type **[NEW]**
- `AVAudioRecorder` **[NEW on tvOS]**
- `AVAudioEngine` with `AVAudioSinkNode`, `AVAudioSourceNode` **[NEW on tvOS]**
- `AVAudioEngine.inputNode.setVoiceProcessingEnabled(_:)` **[NEW on tvOS]**

### AudioToolbox (new on tvOS)
- `AudioQueue` — non-real-time recording **[NEW on tvOS]**
- `AU RemoteIO` audio unit — real-time I/O **[NEW on tvOS]**
- `AU VoiceIO` audio unit (VoiceProcessingIO subtype) — voice processing with echo cancellation **[NEW on tvOS]**

## Code Highlights
No full code samples in the transcript, but the key patterns shown in the session:
```swift
// SwiftUI: Present device picker when no camera available
.continuityDevicePicker($showPicker) { device in
    captureSession.setActiveVideoInput(from: device)
}

// KVO: Monitor camera availability
AVCaptureDevice.observe(\.systemPreferredCamera) { _, _ in
    if let camera = AVCaptureDevice.systemPreferredCamera {
        // Camera connected — start session
    } else {
        // Camera disconnected — teardown, show picker
    }
}

// Audio: Monitor mic availability
AVAudioSession.sharedInstance().observe(\.inputAvailable) { _, _ in
    // Start or stop I/O accordingly
}
```

## Takeaways
- iOS AVFoundation camera code runs largely unchanged on tvOS 17; the primary addition is Device Discovery via `AVContinuityDevicePickerViewController` or the SwiftUI `.continuityDevicePicker` modifier.
- KVO-observe `AVCaptureDevice.systemPreferredCamera` and `AVAudioSession.inputAvailable` to handle camera and mic connect/disconnect gracefully.
- Use only `.cachesDirectory` for file storage on tvOS — `.documentDirectory` is not available; upload recordings to the cloud promptly.
- Enable voice processing (`setVoiceProcessingEnabled`) for any conferencing or voice-capture use case to get tvOS's built-in echo cancellation against Apple TV audio output.

---
_Source: WWDC23 Session 10256 page (abstract, chapter summaries, code samples, and resource links)._
