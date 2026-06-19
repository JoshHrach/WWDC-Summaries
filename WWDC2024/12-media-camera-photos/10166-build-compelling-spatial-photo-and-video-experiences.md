# Build compelling spatial photo and video experiences
**WWDC24 · Session 10166** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10166/)

_Platforms:_ iOS 18 (iPhone 15 Pro capture), visionOS 2

## Overview
This session shows how to adopt spatial photos and videos in your apps: how to capture spatial video on iPhone 15 Pro, how to detect and load spatial media, and the various ways to present it on visionOS — including the new QuickLook `PreviewApplication` API. It closes with a deep dive into the stereo concepts and metadata that make a photo or video "spatial."

A key theme is that there is **no new framework** for spatial media — capabilities are integrated into the frameworks you already use (AVFoundation, PhotoKit, QuickLook, AVKit, WebKit). With a few lines of code you can record, detect, and present spatial media the same way you handle other media.

## Key Topics

### Types of stereoscopic experiences (1:07)
Covers 3D video (flat-screen stereo, e.g. TV/Disney+ titles), spatial video (captured on Vision Pro and iPhone 15 Pro; rendered through a window with a glow, expandable to immersive), and Apple Immersive Video (180° 8K with Spatial Audio). Each renders differently in the shared space vs. full immersion (docking, expansion, native wrap-around).

### Tour of the new APIs (4:13)
- **Capture:** iPhone 15 Pro's wide + ultra-wide cameras are aligned horizontally to enable spatial capture. Switch the `AVCaptureDevice` to `.builtInDualWideCamera`, pick a format where `isSpatialVideoCaptureSupported` is true, and set `isSpatialVideoCaptureEnabled = true` on the movie file output. Use `cinematicExtendedEnhanced` stabilization and observe `spatialCaptureDiscomfortReasons` (`.subjectTooClose`, `.notEnoughLight`) to show guidance UI.
- **Detect/load:** `PhotosPicker` with `matching: .spatialMedia`; `PHAsset` fetch with the `.spatialMedia` media subtype; or `AVAssetPlaybackAssistant.playbackConfigurationOptions` containing `.spatialVideo`.
- **Present:** `PreviewApplication` (QuickLook) for full spatial presentation of photos and videos; JavaScript Element FullScreen API for spatial photos on the web; `AVPlayerViewController` for 3D video playback (renders 3D only in fullscreen, treats spatial video as MV-HEVC 3D).

### Deep dive into spatial media formats (13:14)
Spatial video is a stereo **MV-HEVC** file with spatial metadata; a spatial photo is a stereo **HEIC** with a stereo image group plus metadata. Metadata comprises the (rectilinear) projection, **baseline** (inter-axial distance; ~64mm ≈ human vision), **field of view** (≤90° recommended), and **disparity adjustment** (horizontal offset, as % of width, controlling the zero-parallax plane). Optimal image characteristics: stereo rectified, optical-axis aligned image centers, and no vertical disparity.

## APIs & Frameworks

**AVFoundation (capture)**
- `AVCaptureSession`, `AVCaptureDeviceInput`, `AVCaptureMovieFileOutput`
- `AVCaptureDevice.default(.builtInDualWideCamera, …)` — required for spatial capture
- `AVCaptureDevice.Format.isSpatialVideoCaptureSupported` — **[NEW]**
- `AVCaptureMovieFileOutput.isSpatialVideoCaptureSupported` / `isSpatialVideoCaptureEnabled` — **[NEW]**
- `AVCaptureConnection.preferredVideoStabilizationMode = .cinematicExtendedEnhanced` — **[NEW]**
- `AVCaptureDevice.spatialCaptureDiscomfortReasons` (`.subjectTooClose`, `.notEnoughLight`) — **[NEW]**, KVO-observable
- `AVCaptureVideoPreviewLayer` (monocular preview only)

**AVFoundation / AVKit (detect & present)**
- `AVAssetPlaybackAssistant.playbackConfigurationOptions` containing `.spatialVideo` — **[NEW]**
- `AVPlayerViewController` — 2D/3D playback, HLS for spatial video

**PhotoKit / PhotosUI**
- `PhotosPicker(selection:matching: .spatialMedia)` — **[NEW]** filter
- `PHAsset` fetch with `PHAssetMediaSubtype.spatialMedia` — **[NEW]**

**QuickLook**
- `PreviewApplication` API — **[NEW]**; spawn a QuickLook scene with full spatial presentation (see "What's new in Quick Look for spatial computing")

**WebKit / JavaScript**
- Element FullScreen API — spatial photos from a webpage (see "Optimize for the spatial web")

**Formats & metadata**
- MV-HEVC (stereo video), stereo HEIC (spatial photo), ImageIO spatial metadata
- Spatial metadata: rectilinear projection, baseline, field of view, disparity adjustment

## Code Highlights

Enable spatial video capture (the three key changes):
```swift
guard let videoDevice = AVCaptureDevice.default(.builtInDualWideCamera, for: .video, position: .back) else { return false }
for format in videoDevice.formats where format.isSpatialVideoCaptureSupported {
    try videoDevice.lockForConfiguration()
    videoDevice.activeFormat = format
    videoDevice.unlockForConfiguration()
    break
}
movieFileOutput.isSpatialVideoCaptureEnabled = true
```

Filter the picker to spatial media:
```swift
PhotosPicker(selection: $selectedItem, matching: .spatialMedia) {
    Text("Choose a spatial photo or video")
}
```

## Takeaways
- Spatial capture needs only three changes to a standard AVCapture pipeline: switch to `.builtInDualWideCamera`, select a `isSpatialVideoCaptureSupported` format, and enable `isSpatialVideoCaptureEnabled`.
- Observe `spatialCaptureDiscomfortReasons` and surface guidance UI — the monocular iPhone preview won't reveal low-light noise or focus mismatches.
- For full spatial presentation use `PreviewApplication` (QuickLook); `AVPlayerViewController` plays spatial video only as flat 3D.
- When authoring custom spatial media, get baseline, field of view, and disparity adjustment right, and ensure images are stereo-rectified with no vertical disparity.

---
_Source: WWDC24 Session 10166 page (abstract, chapter summaries, code samples, and resource links)._
