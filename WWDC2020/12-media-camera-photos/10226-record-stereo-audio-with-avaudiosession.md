# Record stereo audio with AVAudioSession
**WWDC20 · Session 10226** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10226/)

_Platforms:_ iOS 14, iPadOS 14

## Overview
New in iOS 14 and iPadOS 14, developers can record stereo audio from the built-in microphones on supported iPhone and iPad hardware using `AVAudioSession`. Stereo recording produces a binaural experience by capturing audio from all device microphones simultaneously and applying a spatial modeling process, giving listeners the ability to perceive direction of sound sources.

The session introduces `inputOrientation` — a new `AVAudioSession` property that tells the system how the device is held relative to its microphones, ensuring that the left and right audio channels match the user's spatial expectations. Developers must coordinate data source selection (front vs. back camera direction) with input orientation to produce correct stereo results.

## Key Topics

### AVAudioSession Microphone and Polar Patterns
- Apple devices have multiple physical microphones; `AVAudioSession` exposes these as **data sources** (not individual mics), each combinable with a **polar pattern**
- Polar patterns: **omni** (single mic, all directions), **cardioid** (beamformer focused forward), **subcardioid** (beamformer focused rearward), and new **stereo** **[NEW in iOS 14]**
- Cardioid uses two mics in tandem for directional response (louder forward, quieter rear)
- Stereo polar pattern uses **all device microphones simultaneously** with special spatial modeling to produce binaural left/right separation

### Stereo Data Sources
- Two stereo data sources available on supported devices:
  - **Upper back** (stereo): forward = rear camera direction; use when recording video with back camera
  - **Upper front** (stereo): forward = front camera direction; use when recording video with front camera
- Left/right directions are relative to device edges and vary by orientation — this is why `inputOrientation` is required

### inputOrientation (NEW)
- `AVAudioSession.setPreferredInputOrientation(_:)` — four possible values: `.portrait`, `.portraitUpsideDown`, `.landscapeLeft`, `.landscapeRight`
- Combined with data source selection, determines all 8 distinct stereo configurations (2 data sources × 4 orientations)
- If app records **video**: set orientation to match left/right in audio with left/right in the video frame (see documentation for camera/rotation mapping)
- If app does **not** record video: match `inputOrientation` to current UI/window orientation so stereo directions match user's expectation
- Keep orientation **fixed during a recording** to avoid sudden direction changes
- Use non-mixable audio session option to get the best chance of receiving the preferred orientation

### Channel Count and Buffer Sizing
- After activating the audio session, check `AVAudioSession.inputNumberOfChannels` — stereo yields 2, mono yields 1
- **Size audio buffers dynamically** based on the actual channel count after activation; do not hardcode mono/stereo assumptions
- Fall back gracefully to mono when stereo is not supported by the hardware

### When to Update Orientation and Data Source
- Update data source and `inputOrientation` in response to UI trait changes (`traitCollectionDidChange`) or user input (e.g., camera selection picker)
- Do **not** change data source or orientation while recording is in progress
- For video recording apps: set orientation before starting the video recording

## APIs & Frameworks

- **AVFoundation** / **AVAudioSession**
- `AVAudioSession.availableInputs` — array of `AVAudioSessionPortDescription`
- `AVAudioSessionPortDescription.portType` — `.builtInMic`
- `AVAudioSession.setPreferredInput(_:)` — select built-in mic as input port
- `AVAudioSessionDataSourceDescription` — represents a microphone data source
- `AVAudioSessionDataSourceDescription.setPreferredPolarPattern(_:)` **[NEW stereo value]**
- `AVAudioSessionPolarPattern.stereo` **[NEW]** — binaural stereo polar pattern
- `AVAudioSessionPolarPattern.cardioid` — focused forward mono beam
- `AVAudioSessionPolarPattern.subcardioid` — focused rearward mono beam
- `AVAudioSessionPolarPattern.omnidirectional` — single-mic omnidirectional
- `AVAudioSessionPortDescription.setPreferredDataSource(_:)` — select data source on an input port
- `AVAudioSession.setPreferredInputOrientation(_:)` **[NEW]** — set stereo left/right orientation
- `AVAudioSessionStereoOrientation` **[NEW]** — `.portrait`, `.portraitUpsideDown`, `.landscapeLeft`, `.landscapeRight`
- `AVAudioSession.inputNumberOfChannels` — actual channel count after session activation (check dynamically)
- `AVAudioSession.setCategory(_:mode:options:)` — use non-mixable options for routing control
- `AVAudioRecorder` — record to file using the configured session
- `AVAudioEngine` — process audio using the configured session
- `UITraitCollection.traitCollectionDidChange(_:)` — listen for orientation changes

## Code Highlights

Select built-in mic as preferred input:
```swift
guard let builtInMic = session.availableInputs?.first(where: { $0.portType == .builtInMic }) else { return }
try session.setPreferredInput(builtInMic)
```

Configure stereo polar pattern and input orientation:
```swift
try newDataSource.setPreferredPolarPattern(.stereo)
try preferredInput.setPreferredDataSource(newDataSource)
try session.setPreferredInputOrientation(orientation.inputOrientation)
```

Guard against changing orientation during recording; update on trait changes:
```swift
override func traitCollectionDidChange(_ previousTraitCollection: UITraitCollection?) {
    updateDataSource()
}

private func updateDataSource() {
    guard controller.state != .recording else { return }
    controller.selectDataSource(named: dataSourceName, orientation: Orientation(windowOrientation)) { layout in
        self.layoutView.layout = layout
    }
}
```

## Takeaways

- Enable stereo recording on supported devices by setting `AVAudioSessionPolarPattern.stereo` on the data source — this is the single biggest improvement for immersive audio capture.
- Always coordinate data source (upper front vs. upper back) with `inputOrientation` so that left/right audio channels match what the listener expects to hear.
- Dynamically size audio buffers based on `AVAudioSession.inputNumberOfChannels` after session activation — do not assume mono or stereo will be available.
- Never change data source or `inputOrientation` during an active recording; update them only in `traitCollectionDidChange` or equivalent orientation/user-selection callbacks.

---
_Source: WWDC20 Session 10226 page (abstract, chapter summaries, code samples, and resource links)._
