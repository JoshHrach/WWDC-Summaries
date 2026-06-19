# Create a More Responsive Camera Experience
**WWDC23 · Session 10105** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10105/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, tvOS 17

## Overview
This session introduces four major improvements to AVFoundation capture and PhotoKit in iOS 17 that make camera apps faster, more reliable, and more expressive. The unifying theme is responsiveness: letting users take more photos with shorter delays, better capture timing, and richer UI feedback, all while maintaining or improving image quality.

The four features are: **Deferred Photo Processing** (deliver a proxy photo immediately, process to full quality later); **Zero Shutter Lag** (use a rolling ring buffer of past frames to capture what you see, not what happened after you tapped); **Responsive Capture** (overlap capture and processing phases for faster burst shooting); and **Video Effects / Reactions** (system-level real-time video effects including gesture-triggered confetti, balloons, hearts, and more).

## Key Topics

### Deferred Photo Processing
Apps opt in by setting `AVCapturePhotoOutput.autoDeferredPhotoDeliveryEnabled = true`. When the pipeline produces a Deep Fusion or similar high-quality photo, it delivers a lightweight proxy via `didFinishCapturingDeferredPhotoProxy` instead of a fully processed image. The proxy must be saved to the photo library immediately using `PHAssetCreationRequest.addResource(with: .photoProxy, data:, options:)`. The system processes the final photo later when conditions are favorable (idle device, plugged in).

PhotoKit additions: `PHAssetResourceType.photoProxy` **[NEW]** enables deferred processing when adding an asset. `PHImageRequestOptions.allowSecondaryDegradedImage` **[NEW]** adds an intermediate higher-resolution callback between the existing low-quality and final callbacks so apps can show a better placeholder while the final image is being computed.

Requirements: iPhone 11 Pro and later.

### Zero Shutter Lag
Enabled automatically for apps linked on iOS 17+ when `isHighestPhotoQualitySupported` is true for the configured format or preset. The camera keeps a rolling ring buffer of past frames; when the user taps, the pipeline reaches back in time to grab the optimal frame window. No opt-in required; opt out with `AVCapturePhotoOutput.isZeroShutterLagEnabled = false`. Not available for flash, manual exposure, bracketed, or constituent photo delivery captures.

### Responsive Capture
When enabled, the photo output overlaps the "capture frames from sensor," "process," and "encode" phases, so a new capture can begin while a previous one is still being processed. Requires Zero Shutter Lag to be on. Set `AVCapturePhotoOutput.isResponsiveCaptureEnabled = true` (check `isResponsiveCaptureSupported` first). Multiple in-flight photo requests produce interleaved delegate callbacks—apps must handle them correctly.

**Fast Capture Prioritization** (opt-in): `AVCapturePhotoOutput.isFastCapturePrioritizationEnabled`. Automatically downgrades photo quality from "quality" to "balanced" when rapid burst shooting is detected, maintaining consistent shot-to-shot timing. Off by default.

**Readiness Coordinator**: `AVCapturePhotoOutputReadinessCoordinator` **[NEW]** notifies a delegate when capture readiness changes. States: `.notRunning`, `.ready`, `.notReadyMomentarily`, `.notReadyWaitingForCapture`, `.notReadyWaitingForProcessing`. Use to disable/dim the shutter button appropriately. Call `startTrackingCaptureRequest(using:)` before each `capturePhoto(with:delegate:)`.

Requirements: iPhone A12 Bionic and later for Responsive Capture / Fast Capture; Readiness Coordinator works wherever `AVCapturePhotoOutput` is supported.

### Video Effects and Reactions
Reactions are system-level effects (balloons, confetti, hearts, fireworks, thumbs up/down, rain, lasers) that blend into the camera video feed. Available via:
1. System Video Effects menu (macOS).
2. Control Center gesture recognition (iOS; user controlled, apps cannot force on/off).
3. `AVCaptureDevice.performEffect(for: reactionType)` — programmatic trigger from app UI.

Key APIs: `AVCaptureReactionType` **[NEW]** enum, `AVCaptureDevice.availableReactionTypes`, `AVCaptureDevice.reactionEffectsInProgress` (KVO observable array of status objects), `AVCaptureDevice.canPerformReactionEffects`, `AVCaptureReactionType.systemImageName(_:)` **[NEW]** for built-in SF Symbol names. Frame rate may drop while effects are in progress; check `AVCaptureDeviceFormat.videoFrameRateRangeForReactionEffectsInProgress`.

Opt-in on iOS/iPadOS: add `NSCameraReactionEffectsEnabled = YES` in Info.plist, or use VoIP `UIBackgroundModes`.

Requirements: A14 Bionic and later (iPhone 12+), Apple Silicon Macs, Intel Macs and Apple TVs via Continuity Camera, Apple Studio Display.

## APIs & Frameworks

**AVFoundation — Photo Capture**
- `AVCapturePhotoOutput.autoDeferredPhotoDeliveryEnabled` **[NEW]** — opt into deferred photo delivery
- `AVCapturePhotoOutput.autoDeferredPhotoDeliverySupported` **[NEW]** — check support
- `AVCapturePhotoCaptureDelegate.didFinishCapturingDeferredPhotoProxy(_:deferredPhotoProxy:error:)` **[NEW]** — proxy delivery callback
- `AVCaptureDeferredPhotoProxy` **[NEW]** — lightweight proxy photo data
- `AVCaptureDeferredPhotoProxy.fileDataRepresentation()` — get data for library storage
- `AVCapturePhotoOutput.isZeroShutterLagEnabled` **[NEW]** — opt out of zero shutter lag
- `AVCapturePhotoOutput.isZeroShutterLagSupported` **[NEW]** — check support
- `AVCapturePhotoOutput.isResponsiveCaptureEnabled` **[NEW]** — enable overlapping captures
- `AVCapturePhotoOutput.isResponsiveCaptureSupported` **[NEW]** — check support
- `AVCapturePhotoOutput.isFastCapturePrioritizationEnabled` **[NEW]** — quality vs. speed adaptation
- `AVCapturePhotoOutput.isFastCapturePrioritizationSupported` **[NEW]** — check support
- `AVCapturePhotoOutputReadinessCoordinator` **[NEW]** — readiness state observer
- `AVCapturePhotoOutputReadinessCoordinatorDelegate` **[NEW]** — `captureReadinessDidChange` callback
- `AVCapturePhotoOutputCaptureReadiness` **[NEW]** — enum: `.notRunning`, `.ready`, `.notReadyMomentarily`, `.notReadyWaitingForCapture`, `.notReadyWaitingForProcessing`
- `AVCapturePhotoOutputReadinessCoordinator.startTrackingCaptureRequest(using:)` **[NEW]**

**AVFoundation — Video Effects**
- `AVCaptureReactionType` **[NEW]** — enum of reaction types (thumbsUp, thumbsDown, heart, balloons, fireworks, rain, confetti, lasers)
- `AVCaptureDevice.performEffect(for:)` **[NEW]** — trigger a reaction effect programmatically
- `AVCaptureDevice.canPerformReactionEffects` **[NEW]** — check if app can trigger effects
- `AVCaptureDevice.availableReactionTypes` **[NEW]** — set of supported reaction types
- `AVCaptureDevice.reactionEffectsInProgress` **[NEW]** — KVO array of in-progress effects
- `AVCaptureReactionType.systemImageName(_:)` **[NEW]** — SF Symbol name for a reaction type
- `AVCaptureDeviceFormat.reactionEffectsSupported` **[NEW]** — format-level support check
- `AVCaptureDeviceFormat.videoFrameRateRangeForReactionEffectsInProgress` **[NEW]** — frame rate during effects

**PhotoKit**
- `PHAssetResourceType.photoProxy` **[NEW]** — resource type for deferred proxy assets
- `PHAssetCreationRequest.addResource(with:data:options:)` — add proxy to library
- `PHImageRequestOptions.allowSecondaryDegradedImage` **[NEW]** — enable intermediate quality callback
- `PHPhotoLibrary.performChanges(_:completionHandler:)` — write proxy to library

## Code Highlights

Enabling deferred photo delivery:
```swift
if photoOutput.autoDeferredPhotoDeliverySupported {
    photoOutput.autoDeferredPhotoDeliveryEnabled = true
}
```

Saving proxy to photo library:
```swift
func photoOutput(_ output: AVCapturePhotoOutput,
                 didFinishCapturingDeferredPhotoProxy proxy: AVCaptureDeferredPhotoProxy?,
                 error: Error?) {
    guard error == nil, let proxy else { return }
    PHPhotoLibrary.shared().performChanges {
        let request = PHAssetCreationRequest.forAsset()
        request.addResource(with: .photoProxy, data: proxy.fileDataRepresentation()!, options: nil)
    }
}
```

Readiness coordinator usage:
```swift
let coordinator = AVCapturePhotoOutputReadinessCoordinator(photoOutput: photoOutput)
coordinator.delegate = self
// Before each capture:
coordinator.startTrackingCaptureRequest(using: photoSettings)
photoOutput.capturePhoto(with: photoSettings, delegate: self)
// In delegate:
func readinessCoordinator(_ coordinator: AVCapturePhotoOutputReadinessCoordinator,
                           captureReadinessDidChange captureReadiness: AVCapturePhotoOutputCaptureReadiness) {
    captureButton.isEnabled = captureReadiness == .ready
}
```

## Takeaways
- Deferred Photo Processing, Zero Shutter Lag, and Responsive Capture work together to maximize both image quality and shot-to-shot speed; they require no per-shot opt-in once enabled on the `AVCapturePhotoOutput`.
- `AVCapturePhotoOutputReadinessCoordinator` is the right tool for shutter button state management, regardless of whether the other features are enabled.
- Reaction effects are system-managed on macOS; on iOS, apps must add an Info.plist key and trigger effects programmatically via `AVCaptureDevice.performEffect(for:)`.
- Proxy photos must be stored to `PHPhotoLibrary` immediately—do not delay while the app may be backgrounded or memory-pressured.

---
_Source: WWDC23 Session 10105 page (abstract, chapter summaries, and resource links)._
