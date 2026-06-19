# Learn about Apple Immersive Video technologies

**Session ID:** 403  
**WWDC Year:** 2025  
**Folder:** `07-visionos-spatial-3d`  
**URL:** https://developer.apple.com/videos/play/wwdc2025/403/

---

## Overview

This session is the deep-dive technical companion to "Explore video experiences for visionOS" (session 304), focused on the tools and APIs needed to build production workflows for Apple Immersive Video. It introduces the new ImmersiveMediaSupport framework (macOS 26, visionOS 26), which enables reading and writing the metadata that makes Apple Immersive Video work: VenueDescriptors (camera calibrations), PresentationDescriptors (dynamic per-frame commands), and AIME (Apple Immersive Media Embedded) data files. It also covers HLS publishing guidelines for Apple Immersive Video and the new Apple Spatial Audio Format (ASAF) and Apple Positional Audio Codec (APAC) for immersive audio.

---

## Key Topics

- Apple Immersive Video overview: parametric projection, stereoscopic 4320×4320/eye at 90fps, P3-D65-PQ
- ImmersiveMediaSupport framework: `VenueDescriptor`, `ImmersiveCamera`, AIMEData
- Timed `PresentationDescriptor` commands: shot flop, fades, dynamic camera selection
- AIVU (Apple Immersive Video Universal) file format: `.aivu` container with embedded metadata
- Reading AIVU metadata via AVFoundation: `quickTimeMetadataAIMEData`, `quickTimeMetadataPresentationImmersiveMedia`
- Writing AIVU files with `AVAssetWriter` and `AIVUValidator`
- HLS publishing: MV-HEVC at 4320×4320/eye, 90fps, recommended bitrates 25–150Mbps
- Remote preview: `ImmersiveMediaRemotePreviewSender`/`Receiver` for Mac-to-Vision-Pro editorial preview
- Apple Spatial Audio Format (ASAF) and Apple Positional Audio Codec (APAC)

---

## APIs & Frameworks

- **ImmersiveMediaSupport** framework (`import ImmersiveMediaSupport`) – **[NEW]** (macOS 26, visionOS 26) Provides types and APIs for reading/writing Apple Immersive Video metadata and previewing content.
- **`VenueDescriptor`** – **[NEW]** Represents the combination of cameras used in an Apple Immersive Video shoot; holds `ImmersiveCamera` array, reference to `AIMEData`, and `save(to:)` for writing `.aime` files.
- **`ImmersiveCamera`** – **[NEW]** Per-camera calibration data including lens geometry, edge-blend mask points, and origin position; part of `VenueDescriptor`.
- **`PresentationDescriptor`** – **[NEW]** Timed metadata type wrapping a set of `PresentationCommand` values for a video frame; JSON-codable.
- **`PresentationCommand`** – **[NEW]** Enum of per-frame commands: camera selection, shot flop (eye/image swap), fade in/out.
- **`PresentationDescriptorReader`** – **[NEW]** Reads `PresentationCommand` values at a specific `CMTime`; use during `AVAssetWriter` encoding to get commands for each frame.
- **`AVMetadataIdentifier.quickTimeMetadataAIMEData`** – **[NEW]** AVFoundation metadata identifier for the `VenueDescriptor` AIMEData track in an AIVU asset.
- **`AVMetadataIdentifier.quickTimeMetadataPresentationImmersiveMedia`** – **[NEW]** AVFoundation metadata identifier for timed `PresentationDescriptor` items.
- **`AVURLAsset` + `.load(.metadata)`** – Read static VenueDescriptor from an AIVU file using the standard AVFoundation metadata loading API.
- **`AVTimedMetadataGroup`** – Used with `AVPlayerItemMetadataOutput` to read per-frame `PresentationDescriptor` during playback or export.
- **`AIVUValidator.validate(url:)`** – **[NEW]** Async function that validates an AIVU file; throws on invalid structure, returns `true` if valid.
- **`ImmersiveMediaRemotePreviewSender`** / **`ImmersiveMediaRemotePreviewReceiver`** – **[NEW]** APIs for sending Apple Immersive Video frames from a Mac editorial tool to Apple Vision Pro for live preview at lower bitrate.
- **ASAF (Apple Spatial Audio Format)** – **[NEW]** Production audio format; linear PCM + spatial metadata in Broadcast Wave files; rendered adaptively by Apple silicon spatial renderer.
- **APAC (Apple Positional Audio Codec)** – **[NEW]** MP4 codec for distributing ASAF audio via HLS or standalone files; required for all Apple Immersive Video experiences. Available on all Apple platforms except watchOS. Minimum bitrate: 64 kbps.
- **`AVAssetWriter`** – Existing API; extended with Apple Immersive Video projection kind (`.appleImmersiveVideo`) for `AVAssetTrack` video projection.

---

## Code Highlights

Reading VenueDescriptor from an AIVU file:
```swift
import ImmersiveMediaSupport
import AVFoundation

func readAIMEData(from aivuFile: URL) async throws -> VenueDescriptor? {
    let avAsset = AVURLAsset(url: aivuFile)
    let metadata = try await avAsset.load(.metadata)
    guard let aimeItem = metadata.first(where: { $0.identifier == .quickTimeMetadataAIMEData }),
          let dataValue = try await aimeItem.load(.value) as? NSData else { return nil }
    return try await VenueDescriptor(aimeData: dataValue as Data)
}
```

Reading per-frame PresentationDescriptors:
```swift
func presentations(from timedMetadata: [AVTimedMetadataGroup]) async throws -> [PresentationDescriptor] {
    var results: [PresentationDescriptor] = []
    for group in timedMetadata {
        for item in group.items where item.identifier == .quickTimeMetadataPresentationImmersiveMedia {
            if let data = try await item.load(.dataValue) {
                results.append(try JSONDecoder().decode(PresentationDescriptor.self, from: data))
            }
        }
    }
    return results
}
```

Writing VenueDescriptor metadata item:
```swift
func makeAIMEMetadataItem(from venue: VenueDescriptor) async throws -> AVMetadataItem {
    let aimeData = try await venue.aimeData
    let item = AVMutableMetadataItem()
    item.identifier = .quickTimeMetadataAIMEData
    item.dataType = String(kCMMetadataBaseDataType_RawData)
    item.value = aimeData as NSData
    return item
}
```

Validating an AIVU file:
```swift
let isValid = try await AIVUValidator.validate(url: aivuFileURL) // true or throws
```

Saving a VenueDescriptor as an AIME file for HLS:
```swift
let aimeFile = FileManager.default.temporaryDirectory
    .appendingPathComponent("primary.aime")
try await venueDescriptor.save(to: aimeFile)
// Copy aimeFile into your HLS playlist directory
```

---

## Takeaways

- `ImmersiveMediaSupport` is aimed at tool developers (non-linear editors, transcoders, streaming pipelines), not end-user app developers; most visionOS app developers should use `AVPlayer`/`AVPlayerViewController` instead.
- AIVU files package everything needed for Apple Immersive Video (video track + VenueDescriptor + PresentationDescriptors) into a single QuickTime container; they are the recommended exchange format between production tools.
- `AIVUValidator.validate(url:)` should be called before publishing AIVU files to catch metadata errors early.
- HLS streaming requires version 12+, the AIME file linked in the multivariant playlist, and APAC audio; the recommended bitrate range is 25–150 Mbps average bandwidth for Apple Vision Pro playback.
- `ImmersiveMediaRemotePreviewSender`/`Receiver` enable real-time editorial preview on Apple Vision Pro from a Mac workstation without producing full-quality files first.
- APAC audio is required for immersive audio delivery and is available on all Apple platforms (except watchOS) at bitrates as low as 64 kbps, making it viable for streaming scenarios.
