# Explore video experiences for visionOS

**Session ID:** 304  
**WWDC Year:** 2025  
**Folder:** `07-visionos-spatial-3d`  
**URL:** https://developer.apple.com/videos/play/wwdc2025/304/

---

## Overview

This session is the primary orientation session for video on visionOS 26, covering the expanded set of video profiles and formats that visionOS now supports. It introduces the Apple Projected Media Profile (APMP) for 180°, 360°, and Wide Field-of-View video, details the Apple Immersive Video format and its parametric projection model, explains how video playback fits into the visionOS immersion system, and shows how to use `AVPlayer` and `AVPlayerViewController` to play back all these formats. The session serves as a prerequisite for the deeper-dive sessions on Apple Immersive Video (403) and APMP (297).

---

## Key Topics

- Overview of all visionOS 26 video profiles: standard, spatial (MV-HEVC), APMP, Apple Immersive Video
- Apple Projected Media Profile (APMP): equirectangular 180°/360°, fisheye, and Wide FOV video
- Apple Immersive Video: parametric projection, stereoscopic, 4320×4320 per eye at 90fps
- Using `AVPlayer` and `AVPlayerViewController` for all video types
- Controlling immersion level: `VideoPlayerConfiguration` immersion modes
- HLS streaming support for all new video profiles
- Spatial audio pairing: APAC codec required for immersive audio tracks
- Quick Look support for AIVU (Apple Immersive Video Universal) files

---

## APIs & Frameworks

- **AVFoundation** – Core framework for loading and playing all video types on visionOS.
- **AVKit** – `AVPlayerViewController` renders video in the appropriate visionOS visual context; automatically adapts UI for immersive vs. windowed playback.
- **`AVPlayer`** – Unchanged entry point; loads standard, MV-HEVC spatial, APMP, and Apple Immersive Video assets transparently via `AVURLAsset`.
- **Apple Projected Media Profile (APMP)** – **[NEW]** Container format for non-parametric immersive video: equirectangular 360°/180°, fisheye projection, and Wide FOV. Stored in `.mov`/`.mp4` with projection metadata.
- **`AVAssetTrack` video projection properties** – New metadata keys for reading the projection type (`equirectangular`, `fisheye`, `parametric`) and field-of-view parameters from an APMP or Apple Immersive Video track.
- **`VideoPlayerConfiguration`** – **[NEW]** (visionOS 26) Struct attached to `AVPlayerViewController` to control immersion level for spatial/immersive content. Properties: `preferredImmersionStyle` (`.mixed`, `.progressive`, `.full`).
- **`AVPlayerViewController.videoPlayerConfiguration`** – **[NEW]** Property for setting `VideoPlayerConfiguration`.
- **Apple Immersive Video Universal (AIVU)** file format – **[NEW]** `.aivu` file extension; a QuickTime container with embedded VenueDescriptor (AIMEData) and PresentationDescriptor timed metadata; playable via Quick Look on visionOS.
- **`ImmersiveMediaSupport`** framework – **[NEW]** (macOS 26, visionOS 26) For reading/writing Apple Immersive Video metadata (covered in depth in session 403).
- **APAC (Apple Positional Audio Codec)** – **[NEW]** Required audio codec for Apple Immersive Video and APMP audio tracks; delivered as an MP4 audio track in HLS or standalone files.
- **MV-HEVC** – Multi-View HEVC codec used for stereoscopic spatial video; unchanged from visionOS 1/2 but now part of the unified video profile taxonomy.

---

## Code Highlights

Playing an APMP or Apple Immersive Video file:
```swift
import AVKit

let url = Bundle.main.url(forResource: "my360video", withExtension: "mov")!
let player = AVPlayer(url: url)

let playerVC = AVPlayerViewController()
playerVC.player = player
// Configure immersion style for spatial/immersive content
playerVC.videoPlayerConfiguration = VideoPlayerConfiguration(
    preferredImmersionStyle: .progressive
)
present(playerVC, animated: true)
player.play()
```

Checking the projection type of an asset track:
```swift
let asset = AVURLAsset(url: videoURL)
let tracks = try await asset.loadTracks(withMediaType: .video)
if let videoTrack = tracks.first {
    let projectionProps = try await videoTrack.load(.videoProjectionProperties)
    print(projectionProps?.projectionKind ?? "standard")
}
```

---

## Takeaways

- visionOS 26 consolidates video into four clear profiles: standard flat, spatial (MV-HEVC), APMP (projected immersive), and Apple Immersive Video (parametric, highest fidelity).
- `AVPlayer` and `AVPlayerViewController` handle all four profiles without format-specific code; the framework selects the correct renderer automatically.
- `VideoPlayerConfiguration.preferredImmersionStyle` gives apps control over whether immersive video begins in a mixed, progressive, or fully immersive shell environment.
- APMP enables existing 360° and 180° camera rigs to deliver content on visionOS without Apple Immersive Video cameras.
- APAC spatial audio is required for the immersive audio experience; standard stereo tracks will fall back but without spatial externalization.
- Watch session 403 for low-level ImmersiveMediaSupport APIs needed when building production tools (non-linear editors, encoders, transcoders) for Apple Immersive Video.
