# Deliver Video Content for Spatial Experiences
**WWDC23 · Session 10071** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10071/)

_Platforms:_ visionOS 1

## Overview
This session is the content-pipeline companion to "Create a great spatial playback experience" (10070). It covers how to prepare and deliver both 2D and stereoscopic 3D video content for visionOS using HTTP Live Streaming (HLS), focusing on what changes in the encoding, packaging, and delivery pipeline when moving from 2D to spatial video.

The core principle is that 2D content delivery to visionOS is identical to other Apple platforms—no changes required. For 3D, the pipeline extends naturally: use Multiview HEVC (MV-HEVC) for video encoding, add a new timed parallax metadata track for caption depth adaptation, use updated HLS tools and a new `REQ-VIDEO-LAYOUT` playlist tag, while reusing existing audio and caption assets unchanged.

## Key Topics

### 2D Content Pipeline (Unchanged for visionOS)
The existing HLS pipeline for 2D content works as-is on visionOS:
- **Video**: encode with HEVC up to 4K; the platform's display runs at 90 Hz (96 Hz mode for 24 fps content); supports standard and high dynamic range.
- **Audio**: produce per-language and per-role audio streams; consider including a Spatial Audio track alongside a stereo fallback.
- **Captions**: produce WebVTT (subtitles, closed captions, SDH); same assets work on visionOS.
- **Packaging**: use Apple HLS tools or equivalent to produce fragmented MP4 segments, media playlists, and a multivariant playlist.

### 3D Video: Multiview HEVC (MV-HEVC)
Stereoscopic 3D video for visionOS uses **MV-HEVC** (an extension of HEVC):
- Each compressed video frame carries both the left-eye and right-eye views (a stereo pair) in a single track.
- Uses a "2D Plus Delta" encoding: the base 2D view (e.g., left eye) is stored as standard HEVC; the right eye is encoded as a delta from the base. This means 2D-only decoders can still decode the base view.
- Supported by Apple silicon hardware decode.
- A new **Video Extended Usage (VEXU) box** is added to the video format description as a lightweight signal that the video is stereoscopic and identifies which views are present (left and right for HLS delivery). The VEXU specification is available in the SDK.

### Parallax Contour Metadata for Captions
The key challenge with 3D video and captions is depth conflict: captions rendered at screen-plane depth can clash with video objects that appear to be in front of the screen (negative parallax), causing viewer discomfort. The solution avoids changing caption formats:

**Parallax contour timed metadata track**: For each video frame, a metadata item describes the parallax across a 2D tiling of the frame. The platform uses this to automatically adjust caption depth at playback time, placing captions behind any foreground video elements.

- **Format**: each metadata item covers a 2D tile grid (recommended: 10×10 tiles, good balance of precision vs. storage) with the minimum parallax value for each tile.
- **Production**: perform parallax/disparity analysis on synchronized left+right view pairs (can use two tracks before MV-HEVC encode; MV-HEVC not required for this analysis step).
- **Packaging**: the metadata track is multiplexed with the video track into the same HLS segments.
- **Result**: existing 2D caption assets (any language, horizontal or vertical layout, any accessibility sizing) are reused unchanged; the platform adapts caption depth automatically.

### Audio for 3D
The same audio streams prepared for 2D delivery can be used with 3D video, provided the video edit timing is identical. If 2D and 3D assets have different edits, separate audio tracks are needed. Spatial Audio is recommended for immersion where the device supports head tracking.

### HLS Packaging Changes for 3D
- `EXT-X-VERSION:12` — new HLS version required for 3D video playlist tags.
- `REQ-VIDEO-LAYOUT` **[NEW]** — attribute on `#EXT-X-STREAM-INF` to indicate stereoscopic video (`STEREO` value). 2D streams are unchanged and can coexist in the same multivariant playlist without this tag.
- Include a 2D iFrame stream in the multivariant playlist for thumbnail scrubbing navigation.
- Delivery infrastructure (CDN, HTTP server) is unchanged.

### Visual Comfort Considerations
3D content design should prioritize viewer comfort for extended viewing:
- Avoid extreme parallax (both negative/foreground and positive/background extremes).
- Minimize high-motion content that causes focusing difficulty.
- Avoid "window violations" (depth conflicts between the 3D video edge and foreground objects).
- Users can affect apparent screen size by repositioning the window, so design for a range of viewing distances.

## APIs & Frameworks

**AVFoundation / HLS (Content Pipeline — New for visionOS)**
- MV-HEVC (Multiview HEVC) **[NEW for visionOS delivery]** — stereo video codec; 2D Plus Delta encoding; Apple silicon hardware decode
- Video Extended Usage box (VEXU) **[NEW]** — MPEG-4 box signaling stereoscopic format; specification in SDK
- Parallax contour timed metadata track **[NEW]** — per-frame parallax tile map for automatic caption depth adaptation; metadata payload specification in SDK
- `EXT-X-VERSION:12` **[NEW]** — HLS playlist version for 3D video
- `REQ-VIDEO-LAYOUT=STEREO` **[NEW]** — HLS `#EXT-X-STREAM-INF` attribute indicating stereoscopic video stream

**HLS Tools**
- Updated Apple HLS tools — handle MV-HEVC video and parallax metadata track packaging automatically

## Code Highlights
This session has no code samples (content-pipeline/encoding/HLS tooling focus). The key practical changes are in HLS multivariant playlist authoring:

Multivariant playlist snippet for 3D stream:
```
#EXTM3U
#EXT-X-VERSION:12

# 3D stereo stream
#EXT-X-STREAM-INF:BANDWIDTH=8000000,CODECS="hvc1",REQ-VIDEO-LAYOUT="CH-STEREO"
stereo/playlist.m3u8

# 2D stream (unchanged, no REQ-VIDEO-LAYOUT)
#EXT-X-STREAM-INF:BANDWIDTH=5000000,CODECS="hvc1"
2d/playlist.m3u8

# iFrame stream for thumbnail scrubbing
#EXT-X-I-FRAME-STREAM-INF:BANDWIDTH=500000,URI="iframe/playlist.m3u8"
```

## Resources
- [Apple HEVC Stereo Video Interoperability Profile](https://developer.apple.com/av-foundation/HEVC-Stereo-Video-Profile.pdf)
- [Video Contour Map Payload Metadata within the QuickTime Movie File Format](https://developer.apple.com/av-foundation/Video-Contour-Map-Metadata.pdf)
- Related: "Create a great spatial playback experience" (WWDC23 10070)
- Related: "Enhance your spatial computing app with RealityKit" (WWDC23 10081)

## Takeaways
- Existing 2D HLS content works on visionOS with zero pipeline changes; focus effort on 3D only if you have stereoscopic source material.
- MV-HEVC is the required codec for 3D video delivery; its 2D Plus Delta approach allows existing non-3D workflows to still preview the base view.
- Adding a parallax contour timed metadata track is the key enabler for reusing all existing 2D caption assets with 3D video; without it, captions may conflict with foreground 3D elements.
- The only required HLS playlist change for 3D is `EXT-X-VERSION:12` and the `REQ-VIDEO-LAYOUT=STEREO` attribute; 2D and 3D streams coexist in the same multivariant playlist.

---
_Source: WWDC23 Session 10071 page (abstract, transcript, chapter summaries, and resource links)._
