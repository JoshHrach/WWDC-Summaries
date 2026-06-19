# Enhance Your App's Audio Recording Capabilities
**WWDC25 · Session 251** · [Watch](https://developer.apple.com/videos/play/wwdc2025/251/)

_Platforms:_ iOS 26, iPadOS 26, macOS Tahoe 26

## Overview
This session covers three major audio recording advancements in iOS 26: a brand-new system input picker for selecting microphones and audio sources without building custom UI, the expansion of spatial audio capture and playback to third-party apps (previously limited to Apple apps), and Bluetooth High-Quality Recording — a new mode that simultaneously enables high-fidelity microphone input and audio output over a single Bluetooth connection.

Together, these features dramatically reduce the engineering effort required to add professional-grade audio recording to any app.

## Key Topics

### AVInputPickerInteraction (New System Input Picker)
- **`AVInputPickerInteraction`** — **[NEW]** a `UIInteraction` that apps add to any `UIView` to get a system-provided audio source picker. No custom UI required.
- The picker surfaces all available audio inputs: built-in mics, AirPods, Bluetooth headsets, USB audio interfaces, etc.
- Works with `AVAudioSession`; the selected input is applied to the session automatically.
- Ideal for podcast apps, voice memos, interview apps, and any app where users need to switch inputs.
- Add to a button view; tapping triggers the picker sheet.

### Spatial Audio Recording
- First-Order Ambisonic (FOA) spatial audio capture — **[NEW for third-party apps]** previously available only in Apple apps (Voice Memos, Camera).
- **`AVAssetWriter` with FOA layout** — write spatially-encoded audio using `spatialAudioChannelLayoutTag`.
- **`.qta` format** — new spatial audio container format used by the system. **[NEW]**
- **`AVCaptureSpatialAudioMetadataSampleGenerator`** — **[NEW]** generates metadata samples required alongside the audio track for correct spatial decoding at playback.
- Playback decoding:
  - `CNAssetSpatialAudioInfo` — **[NEW]** query spatial audio properties of a media asset.
  - `CNSpatialAudioRenderingStyle` — **[NEW]** control rendering style (e.g., headlocked vs. world-locked) at playback.
- `AUAudioMix` — existing API; compatible with spatial audio mixes; use to mix spatial tracks at export.
- Applications: immersive journalism, spatial podcast recording, VR video apps, accessibility recordings for hearing aid spatial processing.

### Bluetooth High-Quality Recording
- **`bluetoothHighQualityRecording`** — **[NEW]** `AVAudioSession` category option that enables simultaneous high-fidelity Bluetooth mic input and audio playback over a single Bluetooth connection.
- Standard Bluetooth profiles (HFP) drop audio output quality to enable mic input. The new mode uses the higher-quality codec profile while keeping mic access.
- Compatible with AirPods and supported Bluetooth headsets.
- Usage: set `.bluetoothHighQualityRecording` in `AVAudioSession` category options alongside the record or playAndRecord category.

## APIs & Frameworks

### AVKit (NEW)
- `AVInputPickerInteraction` — **[NEW]** `UIInteraction` for system audio input picker.
  - `init()` — creates interaction.
  - Add to a `UIView` via `addInteraction(_:)`.

### AVFoundation
- `AVAudioSession` category option: `.bluetoothHighQualityRecording` **[NEW]**
- `AVAssetWriter` — existing; used with new spatial audio channel layout.
- `AVCaptureSpatialAudioMetadataSampleGenerator` — **[NEW]** companion metadata generator for spatial audio capture.
- `spatialAudioChannelLayoutTag` — **[NEW]** channel layout tag for FOA recording.
- `.qta` container format — **[NEW]** spatial audio file format.
- `AUAudioMix` — existing; spatial-compatible audio mix.

### CoreMedia / CoreAudio
- `CNAssetSpatialAudioInfo` — **[NEW]** inspect spatial audio properties of assets.
- `CNSpatialAudioRenderingStyle` — **[NEW]** control spatial rendering at playback.

## Code Highlights

```swift
// Add system input picker to a button
let inputPicker = AVInputPickerInteraction()
microphoneButton.addInteraction(inputPicker)
// Tapping the button presents the system audio source sheet automatically
```

```swift
// Enable Bluetooth high-quality recording
let session = AVAudioSession.sharedInstance()
try session.setCategory(.playAndRecord,
    options: [.allowBluetooth, .bluetoothHighQualityRecording])
try session.setActive(true)
```

```swift
// Spatial audio capture with metadata
let writer = try AVAssetWriter(outputURL: url, fileType: .qta)
let audioInput = AVAssetWriterInput(mediaType: .audio,
    outputSettings: [AVFormatIDKey: kAudioFormatFOA,
                     AVChannelLayoutKey: spatialAudioChannelLayoutTag])
let metadataGenerator = AVCaptureSpatialAudioMetadataSampleGenerator()
writer.add(audioInput)
```

## Takeaways
- Adopt `AVInputPickerInteraction` immediately if your app records audio — it replaces custom mic-picker UI with a zero-effort system component that users already know.
- Use `.bluetoothHighQualityRecording` for any recording app where users are likely to use AirPods — it removes the quality degradation of HFP mic mode while keeping audio output.
- Spatial audio recording opens new categories: immersive podcast, spatial video narration, VR journalism. The full pipeline (capture → metadata → `.qta` → playback) is now available without special entitlements.
- `CNAssetSpatialAudioInfo` and `CNSpatialAudioRenderingStyle` give apps control over how they present already-captured spatial audio at playback time.

---
_Source: WWDC25 Session 251 page (abstract, chapter summaries, code samples, and resource links)._
