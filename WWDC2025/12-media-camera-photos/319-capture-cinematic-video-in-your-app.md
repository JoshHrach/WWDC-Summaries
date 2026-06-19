# Capture Cinematic Video in Your App
**WWDC25 · Session 319** · [Watch](https://developer.apple.com/videos/play/wwdc2025/319/)

_Platforms:_ iOS 26

## Overview
This session introduces the Cinematic Video API for third-party apps, enabling any camera app to capture cinema-style video with automatic rack focus, tracking focus, and shallow depth of field — capabilities previously exclusive to the system Camera app's Cinematic mode (introduced with iPhone 13). The API handles the intelligence automatically, while exposing manual focus controls for advanced use cases.

The session walks through a complete implementation: selecting a compatible device and format, enabling Cinematic capture on `AVCaptureDeviceInput`, building a SwiftUI viewfinder, drawing focus detection overlays, and implementing three manual focus methods.

## Key Topics

### Enabling Cinematic Capture
Cinematic Video is enabled by setting `isCinematicVideoCaptureEnabled = true` on `AVCaptureDeviceInput`. This single flag configures the entire capture graph — movie file output, video data output, and preview layer all receive the Cinematic treatment automatically. A compatible format (queried via `isCinematicVideoCaptureSupported`) must be selected first.

**Supported cameras:** Dual Wide (rear), TrueDepth (front)
**Supported formats:** 1080p and 4K at 30fps; SDR (420v/420f) and 10-bit HDR (x420)

### Movie File Output
`AVCaptureMovieFileOutput` produces a Cinematic movie containing original video, disparity data, and metadata for non-destructive editing. The Cinematic framework (introduced WWDC23) handles playback and editing of bokeh. Still images captured via `AVCapturePhotoOutput` during recording automatically receive the bokeh effect.

### Simulated Aperture
`AVCaptureDeviceInput.simulatedAperture` adjusts bokeh strength in f-stops. Format exposes `minSimulatedAperture`, `maxSimulatedAperture`, and `defaultSimulatedAperture`. Lower f-number = stronger bokeh.

### Metadata and Focus Overlay
`AVCaptureMetadataOutput` with `requiredMetadataObjectTypesForCinematicVideoCapture` must be added. The delegate receives face/subject metadata objects including `objectID` and `cinematicVideoFocusMode` per frame. Metadata bounds must be converted from metadata output coordinates to preview layer coordinates using `layerRectConverted(fromMetadataOutputRect:)` before drawing.

### Manual Focus Control
Three focus methods on `AVCaptureDevice`:
1. `setCinematicVideoTrackingFocus(detectedObjectID:focusMode:)` — focus on a known subject by ID
2. `setCinematicVideoTrackingFocus(at:focusMode:)` — find salient subject at a CGPoint
3. `setCinematicVideoFixedFocus(at:focusMode:)` — lock focus at a fixed depth at a point

`CinematicVideoFocusMode`: `.none`, `.weak` (algorithm retains authority), `.strong` (locks to subject).

### Scene Monitoring
KVO on `cinematicVideoCaptureSceneMonitoringStatuses` notifies when conditions change (e.g., `AVCaptureSceneMonitoringStatus.notEnoughLight`). Empty set means conditions are normal.

## APIs & Frameworks

**AVFoundation (iOS 26)**
- **[NEW]** `AVCaptureDeviceInput.isCinematicVideoCaptureEnabled` — enable Cinematic capture
- **[NEW]** `AVCaptureDeviceInput.isCinematicVideoCaptureSupported`
- **[NEW]** `AVCaptureDeviceInput.simulatedAperture` — bokeh strength in f-stops
- **[NEW]** `AVCaptureDeviceFormat.isCinematicVideoCaptureSupported`
- **[NEW]** `AVCaptureDeviceFormat.minSimulatedAperture`
- **[NEW]** `AVCaptureDeviceFormat.maxSimulatedAperture`
- **[NEW]** `AVCaptureDeviceFormat.defaultSimulatedAperture`
- **[NEW]** `AVCaptureMetadataOutput.requiredMetadataObjectTypesForCinematicVideoCapture`
- **[NEW]** `AVMetadataObject.cinematicVideoFocusMode` — current focus mode on a detected object
- **[NEW]** `AVCaptureDevice.setCinematicVideoTrackingFocus(detectedObjectID:focusMode:)`
- **[NEW]** `AVCaptureDevice.setCinematicVideoTrackingFocus(at:focusMode:)`
- **[NEW]** `AVCaptureDevice.setCinematicVideoFixedFocus(at:focusMode:)`
- **[NEW]** `AVCaptureDevice.CinematicVideoFocusMode` — `.none`, `.weak`, `.strong`
- **[NEW]** `AVCaptureDevice.cinematicVideoCaptureSceneMonitoringStatuses` — KVO property
- **[NEW]** `AVCaptureSceneMonitoringStatus.notEnoughLight`
- `AVCaptureVideoPreviewLayer` — preview layer; Cinematic bokeh rendered in real time
- `AVCaptureMovieFileOutput` — records Cinematic movie with non-destructive metadata
- `AVCapturePhotoOutput` — still capture; automatically bakes in bokeh

**Cinematic framework**
- For playback and editing of Cinematic video files (introduced WWDC23)

## Code Highlights
Enable Cinematic capture:
```swift
if videoInput.isCinematicVideoCaptureSupported {
    videoInput.isCinematicVideoCaptureEnabled = true
}
```

Set metadata object types:
```swift
metadataOutput.metadataObjectTypes = metadataOutput.requiredMetadataObjectTypesForCinematicVideoCapture
```

Convert metadata bounds for UI overlay:
```swift
let layerRect = previewLayer.layerRectConverted(fromMetadataOutputRect: metadata.bounds)
```

Manual tracking focus:
```swift
camera.setCinematicVideoTrackingFocus(detectedObjectID: metadata.objectID, focusMode: .strong)
```

Fixed focus on long press:
```swift
let point = previewLayer.metadataOutputRectConverted(fromLayerRect: CGRect(origin: pressLocation, size: .zero)).origin
camera.setCinematicVideoFixedFocus(at: point, focusMode: .strong)
```

## Takeaways
- Set `isCinematicVideoCaptureEnabled = true` on `AVCaptureDeviceInput` to enable the full Cinematic pipeline with a single line.
- Always select a format where `isCinematicVideoCaptureSupported` is true before enabling Cinematic; use `requiredMetadataObjectTypesForCinematicVideoCapture` for the metadata output to avoid exceptions.
- Differentiate `.weak` focus (algorithm retains control, can auto-rack) vs. `.strong` focus (locked to subject) in your UI — users need visual feedback to know which mode is active.
- KVO `cinematicVideoCaptureSceneMonitoringStatuses` to surface low-light warnings before the user tries to record.

---
_Source: WWDC25 Session 319 page (abstract, chapter summaries, code samples, and resource links)._
