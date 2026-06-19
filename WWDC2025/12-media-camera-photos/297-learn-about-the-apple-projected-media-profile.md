# Learn about the Apple Projected Media Profile
**WWDC25 · Session 297** · [Watch](https://developer.apple.com/videos/play/wwdc2025/297/)

_Platforms:_ visionOS 26, macOS 26

## Overview
The Apple Projected Media Profile (APMP) is a new standard for packaging and delivering immersive, 360-degree spatial media. It defines how omnidirectional video and audio are encoded, containerized, and played back, giving developers a single authoritative format to target across Apple platforms.

APMP builds directly on AVFoundation, CoreMedia, and existing HLS tooling. Existing workflows for HLS-based streaming and AVAssetWriter-based recording can be extended with a handful of new APIs and compression property keys to produce fully compliant APMP assets.

The session walks through the full pipeline: encoding spatial video with Video Toolbox, packaging assets with AVAssetWriter, inspecting content with AVAssetPlaybackAssistant and the new ParametricImmersiveAssetInfo type, delivering via HLS, and rendering with AVPlayer plus the ImmersiveMediaSupport framework.

## Key Topics

### What APMP Contains
APMP wraps equirectangular or MV-HEVC omnidirectional video together with APAC (Apple Positional Audio Codec) multi-channel audio. A single `.movpkg` or HLS playlist describes projection metadata, stitching seams, and spatial audio configuration in a standardized way.

### Encoding and Packaging
New `CompressionPropertyKey` values on AVAssetWriter signal APMP-compliant output. Video Toolbox handles the low-level HEVC encoding while CoreMedia's `CMTaggedDynamicBuffer` carries the tagged, multi-view sample buffers through the pipeline.

### Playback Inspection
`AVAssetPlaybackAssistant` now surfaces `ParametricImmersiveAssetInfo`, allowing the app to query projection type, field-of-view parameters, and audio layout before playback begins. This enables adaptive UI (e.g., showing a spatial audio indicator) without starting AVPlayer.

### Delivery via HLS
The existing HLS tooling accepts APMP segments; media playlist tags describe the immersive track groups. The session covers the required `EXT-X-` tags and recommends bitrate ladder guidelines for visionOS streaming.

### Rendering
ImmersiveMediaSupport is the new framework that bridges AVPlayer output to the visionOS compositor. It handles reprojection, head-tracking synchronization, and the transition between windowed and fully immersive modes.

## APIs & Frameworks

- **AVFoundation** — extended for APMP asset creation and playback
- **ImmersiveMediaSupport** **[NEW]** — rendering layer between AVPlayer and the visionOS spatial compositor
- **AVAssetPlaybackAssistant** — updated to return `ParametricImmersiveAssetInfo` **[NEW]**
- **ParametricImmersiveAssetInfo** **[NEW]** — projection type, FOV, and audio layout query
- **CMTaggedDynamicBuffer** (CoreMedia) **[NEW]** — tagged multi-view sample buffer carrier
- **AVVideoComposition** — used to compose omnidirectional layers for APMP output
- **AVAssetWriter** + `CompressionPropertyKey` **[NEW values]** — APMP-compliant packaging
- **Video Toolbox** — HEVC encoding for omnidirectional content
- **APAC (Apple Positional Audio Codec)** — spatial audio encoding within APMP
- **HLS tooling** — updated to accept APMP track groups and playlist tags

## Code Highlights

```swift
// Query APMP asset info before playback
let assistant = AVAssetPlaybackAssistant(asset: asset)
let info = try await assistant.parametricImmersiveAssetInfo
if let info {
    print("Projection: \(info.projectionType), FOV: \(info.fieldOfView)")
}
```

```swift
// Write APMP-compliant output with AVAssetWriter
let writerInput = AVAssetWriterInput(mediaType: .video, outputSettings: [
    AVVideoCodecKey: AVVideoCodecType.hevc,
    // APMP-specific compression property
    AVVideoCompressionPropertiesKey: [
        AVVideoProfileLevelKey: "APMP_Projection_Equirectangular"
    ]
])
```

## Takeaways

- APMP is the canonical Apple format for immersive/360° media; adopt it now for visionOS 26 and macOS 26 compatibility.
- Use `AVAssetPlaybackAssistant` and `ParametricImmersiveAssetInfo` to gate UI and playback decisions on asset capabilities.
- `ImmersiveMediaSupport` abstracts compositor integration — do not bypass it with custom Metal renderers for standard playback.
- Existing HLS pipelines need only new playlist tags and the APMP track group to become APMP-compliant.

---
_Source: WWDC25 Session 297 page (abstract, chapter summaries, code samples, and resource links)._
