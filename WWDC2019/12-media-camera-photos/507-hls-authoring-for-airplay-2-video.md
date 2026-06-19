# HLS Authoring for AirPlay 2 Video
**WWDC19 · Session 507** · [Watch](https://developer.apple.com/videos/play/wwdc2019/507/)

_Platforms:_ iOS 13, tvOS 13, macOS 10.15 Catalina (AirPlay 2 Video to third-party TVs)

## Overview
AirPlay 2 Video expands AirPlay support beyond Apple TV to third-party smart TVs with AirPlay built in. These TVs represent a new device class with stricter and different HLS authoring requirements than iOS or Apple TV. This session details the new requirements published in an updated HLS Authoring Specification and covers changes to the `HLSReport` validation tool that now checks all rule sets (general, iOS, tvOS, AirPlay 2) by default.

The key distinction from Apple TV is that AirPlay 2 TVs cannot switch codecs seamlessly mid-stream, cannot mix codec families across bitrate tiers, and require tighter segment synchronization. Content must be self-consistent within a single codec family at all quality levels.

## Key Topics

**Variant Synchronization**
- All variants must be synchronized on a shared timeline
- Use at least millisecond precision for timing
- All video segments must start with IDR (Instant Decoder Refresh) frames
- Proper alignment makes adaptive bitrate switching seamless on these TVs

**Codec Consistency (Critical for AirPlay 2 TVs)**
- No codec switching across variants: do not mix HEVC and H.264 tiers in the same stream intended for AirPlay 2 TVs
- These TVs stick with the codec chosen at stream start — they cannot switch codecs mid-playback
- Each codec (H.264 or HEVC) must cover the full range of bitrates from low to high on its own
- Do not use H.264 for low-bitrate variants and HEVC for high-bitrate variants

**Frame Rate Restrictions**
- Frame rate changes are allowed but only within the same cadence family
- No switching between 25 fps (PAL) and 30 fps (NTSC) cadences

**I-Frame Variants**
- I-frame-only playlists required for effective trick play (fast-forward, rewind, seek)
- I-frame variants must match the codec of the normal video variants they accompany

**Encryption Requirements (FairPlay)**
- Common Encryption (CENC) recommends 10% partial encryption; FairPlay **requires** it — other encryption patterns will not work
- For sample encryption, prefer the ISO BMFF format using `saio`+`saiz` boxes; the CMAF format using `senc` box is also accepted (providing both is acceptable)

**HDR Content**
- Provide multiple HDR formats (e.g., both Dolby Vision and HDR 10) since TVs may support only one format

**Subtitles**
- Use WebVTT for subtitles

**MIME Types**
- Recommend explicit MIME types for all content
- HLS playlist: `application/vnd.apple.mpegurl`
- Video: `video/mp4`
- Audio: `audio/mp4`
- WebVTT subtitles: `text/plain` (not `text/vtt` — that MIME type was not IANA-registered and is rejected by some clients)
- AC-3 audio: `audio/ac3`
- Enhanced AC-3: `audio/eac3`
- HEVC video: `video/mp4` with appropriate codec string
- Note: HEIF image and HEVC+Alpha are not applicable to AirPlay 2 content specifically

## APIs & Frameworks

**HLS Tooling (Command-Line)**
- `mediastreamvalidator` — validates streams against the HLS specification; unchanged
- `HLSReport` — validates against the HLS Authoring Specification **[UPDATED]**
  - Previously required `-os` flag to select iOS/tvOS rule sets separately
  - Now checks **all rule sets by default**: General, iOS, tvOS, and AirPlay 2 **[NEW behavior]**
  - `--rule-set` option available to customize which sets are checked, but not needed for most users
  - `-os` option still functional but deprecated for this purpose
  - Output now grouped by rule set with separate Must Fix / Should Fix subsections per set
  - A rule that is "Should Fix" in General requirements may be "Must Fix" in the AirPlay 2 section
  - Sections with no violations are omitted from output

**HLS Authoring Specification**
- Updated specification published with additional AirPlay 2 requirements section
- Available at `developer.apple.com/streaming/`

**AVFoundation (referenced)**
- `AVPlayer` — handles AirPlay 2 Video routing; no new API required for basic playback
- `AVRoutePickerView` — UI for AirPlay destination selection
- See companion session "Reaching the Big Screen with AirPlay 2" (Session 501) for app integration

## Code Highlights

No code samples; this is a content authoring/specification session. The primary deliverable is an updated HLS stream structure. Key authoring rules in summary:

```
# AirPlay 2 HLS Authoring Checklist
✓ All variants synchronized to millisecond precision, starting with IDR frames
✓ Single codec family (H.264 OR HEVC) covers all bitrate tiers
✓ No codec switching between variants
✓ No frame-rate cadence switching (no 25↔30 fps mix)
✓ I-frame variants present, matching codec of normal variants
✓ FairPlay encryption uses 10% partial encryption (required, not optional)
✓ Sample encryption uses saio+saiz boxes (ISO BMFF preferred)
✓ HDR: provide both Dolby Vision and HDR 10 where applicable
✓ Subtitles in WebVTT format
✓ MIME types declared for all content; use text/plain for WebVTT
```

Run both validation tools as a pair:
```bash
mediastreamvalidator https://example.com/master.m3u8
HLSReport https://example.com/master.m3u8
# HLSReport now checks General + iOS + tvOS + AirPlay 2 automatically
```

## Takeaways
- AirPlay 2 TVs cannot switch codecs; every bitrate tier must be in one codec family (H.264 or HEVC) — this is the most common authoring mistake to fix.
- FairPlay partial encryption at 10% is a hard requirement for AirPlay 2, not a recommendation — other patterns will cause playback failure.
- Run `HLSReport` without `-os` from now on; it checks all rule sets including the new AirPlay 2 rules and the output tells you per-section severity.
- Use `text/plain` for WebVTT MIME type, not `text/vtt`, to avoid client rejection.

---
_Source: WWDC19 Session 507 page (abstract, chapter summaries, code samples, and resource links)._
