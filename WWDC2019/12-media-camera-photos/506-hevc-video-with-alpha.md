# HEVC Video with Alpha
**WWDC19 · Session 506** · [Watch](https://developer.apple.com/videos/play/wwdc2019/506/)

_Platforms:_ iOS 13, iPadOS 13, tvOS 13, macOS 10.15 Catalina

## Overview
iOS 13, tvOS 13, and macOS Catalina introduce support for HEVC video with an alpha channel, enabling non-rectangular video compositing at distribution-ready bitrates. Previously, alpha video required professional formats like Apple ProRes 4444 with very high data rates unsuitable for app delivery. By encoding both a base color layer and an alpha layer inside a single HEVC video track, the format achieves high compression efficiency while maintaining full transparency support.

The feature is deeply integrated into AVFoundation — it fits into existing `AVPlayer`, `AVAssetWriter`, `AVAssetExportSession`, `AVPlayerItemVideoOutput`, and `AVAssetImageGenerator` workflows with minimal code changes. Safari also supports HEVC with Alpha on iOS 13 and macOS Catalina, including the `mediaCapabilities` API for feature detection. Hardware acceleration is available on recent devices.

## Key Topics

**File Format**
- Single video track containing two HEVC layers per frame: a base color layer and an alpha layer
- Backwards compatible: players that do not understand the alpha layer display only the base layer
- Encoded using standard HEVC codec type; uses special HEVC layer syntax
- Also supports HEIF sequences with alpha (multiple still images with transparency in one file)

**Encoding**
- Use `AVVideoCodecType.hevcWithAlpha` when configuring `AVAssetWriterInput` or `VTCompressionSession`
- Supports premultiplied and straight (unassociated) alpha; premultiplied is the default and recommended for GPU-rendered content
- Alpha mode specified via compression session property or pixel buffer attachment; mismatches cause encoding failure
- Alpha channel quality set independently via `kVTCompressionPropertyKey_TargetQualityForAlpha` (range 0–1; 1 = near lossless)
- Bitrate parameter applies only to the base layer; alpha layer uses quality-based encoding
- New `AVAssetExportPreset` variants with "WithAlpha" suffix for transcoding from ProRes 4444 or other alpha sources

**Playback**
- `AVPlayer` + `AVPlayerLayer`: transparent background, composites with Core Animation layers beneath
- `AVPlayerItemVideoOutput`: extract individual pixel buffers for custom Metal/SpriteKit/SceneKit rendering
- `AVAssetImageGenerator`: extract individual frames as `CGImage` with alpha
- `AVAssetReader`: frame extraction for non-playback workflows
- `VTDecompressionSession`: direct decoder access

**Detection**
- `AVMediaCharacteristic.containsAlphaChannel` — test on an `AVAsset` or track
- Format description extension: `kCMFormatDescriptionExtension_ContainsAlphaChannel`
- Use `AVAssetExportSession.determineCompatibility(ofExportPreset:for:outputFileType:)` to validate alpha presence before export (skip if using video composition to generate alpha from non-alpha sources)

**Background Burn-in**
- `AVAssetExportSession` can burn in a solid background color to produce a standard video without alpha, for delivery to players that don't support the format

**Web Support**
- Safari on iOS 13 and macOS Catalina plays HEVC with Alpha on web pages
- `MediaCapabilities` API available for feature detection

## APIs & Frameworks

**AVFoundation**
- `AVVideoCodecType.hevcWithAlpha` **[NEW]** — codec type for encoding; triggers alpha layer encoding
- `AVAssetWriterInput` — use `hevcWithAlpha` in output settings
- `AVAssetExportSession` **[NEW exports]**:
  - `AVAssetExportPresetHEVC1920x1080WithAlpha` **[NEW]**
  - `AVAssetExportPresetHEVC3840x2160WithAlpha` **[NEW]**
  - `determineCompatibility(ofExportPreset:for:outputFileType:completionHandler:)` — validate alpha compatibility
- `AVMediaCharacteristic.containsAlphaChannel` **[NEW]** — detects alpha in asset/track
- `AVPlayer` — plays HEVC with Alpha transparently; no code change required
- `AVPlayerLayer` — composites video with transparent background over CA layer hierarchy
- `AVPlayerItemVideoOutput` — frame-level access for custom Metal/SpriteKit rendering
- `AVAssetImageGenerator` — `copyCGImage(at:actualTime:)` returns `CGImage` with alpha
- `AVAssetReader` + `AVAssetReaderTrackOutput` — extract video frames for offline processing

**VideoToolbox**
- `VTCompressionSession` — direct encoder access; use `kCMVideoCodecType_HEVCWithAlpha` codec
  - `kVTCompressionPropertyKey_TargetQualityForAlpha` **[NEW]** — Float in 0–1 range for alpha layer quality
  - `kVTCompressionPropertyKey_AlphaChannelMode` **[NEW]** — `.straightAlpha` or `.premultipliedAlpha`
- `VTDecompressionSession` — direct decoder access
- `kCMFormatDescriptionExtension_ContainsAlphaChannel` **[NEW]** — format description extension

**Core Media**
- `CMFormatDescription` — query `kCMFormatDescriptionExtension_ContainsAlphaChannel`

## Code Highlights

Encoding HEVC with Alpha using `AVAssetWriterInput`:
```swift
let videoSettings: [String: Any] = [
    AVVideoCodecKey: AVVideoCodecType.hevcWithAlpha,
    AVVideoWidthKey: width,
    AVVideoHeightKey: height,
    AVVideoCompressionPropertiesKey: [
        kVTCompressionPropertyKey_TargetQualityForAlpha: 0.75,
        kVTCompressionPropertyKey_AlphaChannelMode: kVTAlphaChannelMode_PremultipliedAlpha
    ]
]
let input = AVAssetWriterInput(mediaType: .video, outputSettings: videoSettings)
```

Detecting alpha channel in a source asset:
```swift
if asset.hasMediaCharacteristic(.containsAlphaChannel) {
    // use hevcWithAlpha export preset
}
```

## Takeaways
- HEVC with Alpha brings distribution-ready transparent video to iOS/macOS apps and Safari with no new player infrastructure — `AVPlayer` + `AVPlayerLayer` handles it transparently.
- Use `AVPlayerItemVideoOutput` when you need to composite video frames in Metal, SpriteKit, or SceneKit rather than the standard UIKit/Core Animation layer stack.
- Set alpha quality independently from base layer bitrate; quality near 1.0 is important for clean edges on foreground subjects.
- Provide a "burn in background" fallback export for devices and third-party players that do not yet support the format.

---
_Source: WWDC19 Session 506 page (abstract, chapter summaries, code samples, and resource links)._
