# HLS Authoring Update
**WWDC17 · Session 515** · [Watch](https://developer.apple.com/videos/play/wwdc2017/515/)

_Platforms:_ iOS 11, macOS High Sierra 10.13, tvOS 11

## Overview
This session covers the 2017 updates to Apple's HLS Authoring Specification for Apple Devices and the companion command-line tools distributed at developer.apple.com/streaming. The biggest change is the addition of HEVC (H.265) support in HLS, including new bit-rate guidelines, codec interoperability rules for mixed H.264/HEVC playlists, and tooling updates to validate HEVC streams. A second new feature is IMSC1 subtitle support, an alternative to WebVTT based on TTML and compatible with EBU and SMPTE standards.

The HLS Authoring Specification sits above the IETF HLS internet draft: it includes Apple-player-specific requirements and best-practice recommendations that go beyond what the standard mandates. Developers authoring HLS content for any Apple platform are expected to follow it. The session walks through how the new HEVC rules integrate with existing H.264 delivery, emphasizing backward compatibility for older players via the `CODECS` playlist attribute and the requirement to always ship at least one H.264 variant alongside HEVC variants.

The HLS toolset received several significant improvements: `MediaStreamValidator` can now validate local files (no HTTP server needed), both it and `HLSReport` check more things about streams (including codec usage), and new CLI options like `--description` and `--columns` improve workflow and reporting clarity.

## Key Topics
- **HEVC in HLS** — HEVC Main 10 Profile, Level 5.0, High Tier support; new bit-rate guidelines (preliminary, subject to revision); player behavior in mixed playlists
- **Mixed H.264 + HEVC playlists** — always include `CODECS` attribute; always ship at least one H.264 variant; player prefers HEVC when bit rates are similar; trick play (I-frame-only) variants follow the same rules
- **IMSC1 subtitle support** — text profile of IMSC1; must use fragmented MP4 containers (not plain text); IMSC1 `CODECS` value required for older-client compatibility; can mix freely with WebVTT and H.264/HEVC
- **WebVTT** — unchanged; do not include `CODECS` value for WebVTT for maximum backward compatibility
- **MediaStreamValidator improvements** — local file validation via relative/absolute paths or `file://` URLs; HEVC support; more stream correctness checks (codec usage, ordering); better error handling during validation
- **HLSReport improvements** — new `--columns` option with column identifiers (replaces deprecated `--id` / `--pl` flags); CODECS column now visible in reports; renditions grouped consistently (audio, CC, subtitle kept separate)
- **MediaFileSegmenter** — `--hdcp-level` option (added in 2016, newly documented) sets `HDCP-LEVEL` attribute in generated master playlist via Plist pipeline
- **New `--description` option** — adds human-readable stream description text to the top of validator reports

## APIs & Frameworks

### HLS Playlist Attributes & Tags
- **`CODECS` attribute** **[Required for HEVC]** — must be present on all HEVC variants to prevent older players from attempting playback
- **`EXT-X-STREAM-INF`** — carries `CODECS`, `BANDWIDTH`, `RESOLUTION`, `HDCP-LEVEL`
- **`HDCP-LEVEL` attribute** — set via `MediaFileSegmenter --hdcp-level`; ensures DRM-constrained variants are skipped by non-HDCP clients
- **HEVC codec string** — `hvc1` / `hev1` in the `CODECS` attribute (Main 10 Profile, Level 5.0, High Tier)
- **IMSC1 codec string** — required in `CODECS` for IMSC1 subtitle renditions
- **Fragmented MP4 (fMP4)** — required container format for IMSC1 subtitle segments; optional (but recommended) for H.264 video

### HLS Command-Line Tools
- **`MediaStreamValidator`** — validates HLS master/media playlists; new local file support; `--description` option; improved codec, ordering, and error checks
- **`HLSReport`** — generates human-readable stream analysis reports; new `--columns <id-list>` option; CODECS column; deprecated `--id` and `--pl` in favor of column identifiers
- **`MediaFileSegmenter`** — segments video/audio for HLS; `--hdcp-level` option
- **Variant Playlist Creator** — consumes Plist from `MediaFileSegmenter` to produce master playlists with correct attributes
- **`hlsreport`** — column identifiers: `id`, `pl` (retained for backward compat), and new identifiers

### Codec Support Matrix (iOS 11)
- **H.264 (AVC)** — transport stream (`.ts`) preferred for maximum compatibility; fMP4 also supported
- **HEVC (H.265)** — Main 10 Profile, Level 5.0, High Tier; fMP4 container
- **AAC / AC-3 / EC-3** — audio codecs; unchanged
- **WebVTT** — plain text subtitle files (`.vtt` / `.webvtt`); no `CODECS` value
- **IMSC1** **[NEW]** — TTML-based subtitles; fragmented MP4 container; `CODECS` attribute required

## Code Highlights
No code samples in this session. Key playlist authoring rules:

```
# Mixed H.264 + HEVC master playlist (required structure)
#EXTM3U

# Always include H.264 variants for backward compat
#EXT-X-STREAM-INF:BANDWIDTH=2000000,CODECS="avc1.640028,mp4a.40.2"
h264_high.m3u8

# HEVC variants with CODECS attribute (required)
#EXT-X-STREAM-INF:BANDWIDTH=1500000,CODECS="hvc1.2.4.L150.90,mp4a.40.2"
hevc_main10.m3u8

# IMSC1 subtitle rendition
#EXT-X-MEDIA:TYPE=SUBTITLES,GROUP-ID="subs",NAME="English",
  DEFAULT=YES,URI="subs_en.m3u8",CODECS="im1t"
```

## Takeaways
- Always use the `CODECS` attribute on every HEVC variant and always include at least one H.264 variant; without these, older players will break.
- `MediaStreamValidator` now validates local files directly — no HTTP server required — significantly simplifying offline development and CI/CD validation workflows.
- IMSC1 is a production-ready alternative to WebVTT for international subtitle delivery; it must use fMP4 containers and requires the `CODECS` attribute.
- HEVC bit-rate guidelines in the Authoring Specification are preliminary; encoders are still improving and the guidelines will be revised.

---
_Source: WWDC17 Session 515 page (abstract, chapter summaries, code samples, and resource links)._
