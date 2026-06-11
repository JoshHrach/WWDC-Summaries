# Build live production tools for Apple Immersive Video
**WWDC26 · Session 338** · [Watch](https://developer.apple.com/videos/play/wwdc2026/338/)

_Platforms:_ macOS 27, visionOS 27

## Overview
This session goes behind the scenes of live Apple Immersive Video (AIV) production, explaining the full technical pipeline from camera to Apple Vision Pro playback. The target audience is developers building broadcast/production tools — encode servers, media routers, monitoring apps, or playback applications — that need to handle AIV at live production scale.

The session covers what distinguishes live AIV from standard live video: massive resolution, high frame rates, and rich Apple Spatial Audio Format (ASAF) mixes that must be synchronized and transported together. It then walks through the three pillars of the live format — streaming ProRes video, uncompressed PCM audio, and per-frame JSON metadata — and how these are packaged and transported over IP networks using the SMPTE 2110 industry standard. Finally, it demonstrates how live streams are recorded to disk and played back using `AVAssetWriter` and the `ImmersiveMediaSupport` framework.

The session uses live LA Lakers courtside coverage on the Spectrum SportsNet / NBA app as a concrete production example.

## Key Topics

### What Makes Immersive Live Different
- Massive video resolution requirements (significantly higher than standard 4K/8K broadcast).
- High frame rates (matching Apple Vision Pro's rendering capabilities).
- Rich Apple Spatial Audio Format (ASAF) multi-channel audio mix — not just stereo or 5.1.
- All three streams (video, audio, metadata) must be synchronized frame-accurately.

### Immersive Live Format
- **Video**: streaming ProRes (not compressed-for-delivery codecs at this stage).
- **Audio**: uncompressed PCM, carrying the ASAF mix.
- **Metadata**: per-frame JSON containing scene metadata (orientation, projection parameters, etc.).
- The three-stream bundle is what gets transported and later muxed into a playable AIV file.

### Real-Time Media Transport — SMPTE 2110
- SMPTE 2110 is the industry-standard protocol for professional IP media transport.
- Each stream (video, audio, metadata) travels as a separate SMPTE 2110 flow over the IP network.
- Enables standard broadcast infrastructure (routers, monitors, switchers) to handle AIV feeds.
- Compatible with existing professional broadcast workflows.

### Recording and Playback
- **Recording**: `AVAssetWriter` writes the incoming SMPTE 2110 streams to an asset file on disk.
- **Video Toolbox**: `kVTCompressionPropertyKey_ProjectionKind = kVTProjectionKind_AppleImmersiveVideo` sets the vexu metadata required for the video track to be recognized as AIV.
- **Immersive Media Support framework**: reads and writes AIV-specific metadata; used for both recording (authoring metadata tracks) and playback (parsing scene metadata).
- `CMVideoCodecType` — identifies the codec used; ProRes variants supported.
- Playback via existing visionOS AVFoundation AIV playback stack.

## APIs & Frameworks

### VideoToolbox
- `kVTCompressionPropertyKey_ProjectionKind` — existing key
  - `kVTProjectionKind_AppleImmersiveVideo` **[NEW]**: value that marks video as AIV in vexu metadata
- Compression session setup for live ProRes encode

### AVFoundation
- `AVAssetWriter` — existing; used to write multi-track AIV assets from live streams
- `AVAssetWriterInput` — existing; separate inputs for video, audio, metadata tracks
- `CMSampleBuffer` — existing; carries per-frame media data from SMPTE 2110 input

### CoreMedia
- `CMVideoCodecType` — existing; identifies ProRes variants (`.appleProRes422`, `.appleProRes4444`, etc.)

### ImmersiveMediaSupport (NEW/updated framework)
- Authoring and reading AIV scene metadata **[NEW capabilities]**
- Per-frame JSON metadata encoding/decoding for the metadata track
- Integration with `AVAssetWriter` for muxing metadata alongside video/audio
- See: [Immersive Media Support](https://developer.apple.com/documentation/ImmersiveMediaSupport)

### External Standards
- **SMPTE 2110**: professional IP media transport standard
  - ST 2110-20: video
  - ST 2110-30: audio
  - ST 2110-40: ancillary/metadata
- **Apple Spatial Audio Format (ASAF)**: Apple's multi-object spatial audio format for AIV

### Reference Documentation
- [kVTCompressionPropertyKey_ProjectionKind](https://developer.apple.com/documentation/VideoToolbox/kVTCompressionPropertyKey_ProjectionKind)
- [CMVideoCodecType](https://developer.apple.com/documentation/CoreMedia/CMVideoCodecType)
- [Immersive Media Support](https://developer.apple.com/documentation/ImmersiveMediaSupport)
- [Apple ProRes White Paper](https://www.apple.com/final-cut-pro/docs/Apple_ProRes.pdf)
- [Apple ProRes RAW White Paper](https://www.apple.com/final-cut-pro/docs/Apple_ProRes_RAW.pdf)

## Code Highlights

Set the projection kind for AIV encoding:
```swift
import VideoToolbox

let compressionProperties: [String: Any] = [
    kVTCompressionPropertyKey_ProjectionKind as String:
        kVTProjectionKind_AppleImmersiveVideo
]
```

## Takeaways
- Live AIV production is built on SMPTE 2110 — the same infrastructure used for standard broadcast — making it approachable for developers already familiar with professional IP media workflows.
- The three-stream bundle (ProRes video + uncompressed PCM + JSON metadata) must be kept synchronized frame-accurately; this is the core engineering challenge for live production tools.
- `kVTCompressionPropertyKey_ProjectionKind = kVTProjectionKind_AppleImmersiveVideo` is a single API call that marks a video track as Apple Immersive Video — the rest of the metadata is authored via `ImmersiveMediaSupport`.
- `AVAssetWriter` is the recording path for live streams; playback flows through the same AVFoundation / visionOS pipeline used by on-demand AIV content.

---
_Source: WWDC26 Session 338 page (abstract, chapter summaries, code samples, and resource links)._
