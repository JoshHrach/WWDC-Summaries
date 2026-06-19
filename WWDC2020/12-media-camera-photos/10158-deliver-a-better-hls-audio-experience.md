# Deliver a Better HLS Audio Experience
**WWDC20 · Session 10158** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10158/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, tvOS 14, watchOS 7

## Overview
This session introduces three new audio codecs supported in HLS as of the 2020 OS releases — xHE-AAC, FLAC, and Apple Lossless — and provides guidance on how to deploy them effectively in HLS master playlists for both low-bandwidth and high-fidelity streaming scenarios. The talk supplements the HLS Authoring Specification for Apple Devices and is specifically aimed at content authors and streaming engineers.

The first half focuses on xHE-AAC (Extended High-Efficiency AAC), a highly efficient codec suited for bit rates as low as 24 kbps with built-in Dynamic Range Control (DRC) support that is increasingly mandated by industry standards. The second half covers FLAC and Apple Lossless lossless codecs, which support up to 8-channel multichannel audio (5.1 and 7.1), and explains why multichannel AAC must be included as a stepping stone for adaptive bitrate scaling into lossless multichannel content.

## Key Topics

**New Audio Codecs in HLS (2020 OS Releases)**
- **xHE-AAC** (`mp4a.40.42`): Extended High-Efficiency AAC, also called USAC (Unified Speech and Audio Coding). Optimized for speech and general audio at 24–200 kbps. New to HLS in 2020; was available for file-based playback since iOS 13 / macOS Catalina. Stereo only via AVPlayer; fragmented MP4 container; common encryption only.
- **FLAC** (`fLaC`, case-sensitive): Free Lossless Audio Codec. New to HLS in 2020; available for file-based playback previously. Up to 8 channels (5.1/7.1). Channel order: L, R, C, LFE, Ls, Rs. Fragmented MP4; common encryption only.
- **Apple Lossless** (`alac`): Apple Lossless Audio Codec. New to HLS in 2020; available for file-based playback previously. Up to 8 channels (5.1/7.1). Channel order: C, L, R, LFE, Ls, Rs. Fragmented MP4; common encryption only.

**xHE-AAC vs. AAC Family**
The AAC family progression: AAC-LC (`mp4a.40.2`, min 96 kbps) → HE-AAC with SBR (`mp4a.40.5`, min 48 kbps) → HE-AAC v2 with Parametric Stereo (`mp4a.40.29`, min 32 kbps) → xHE-AAC (`mp4a.40.42`, min 24 kbps). xHE-AAC is not backwards-compatible with earlier AAC decoders. Must be correctly identified with `mp4a.40.42` in the CODECS attribute.

**Dynamic Range Control (DRC)**
DRC metadata allows continuous adjustment of audio signal levels to reduce the volume gap between loud and soft passages. The HLS Authoring Specification recommends including DRC metadata. CMAF mandates DRC inclusion for xHE-AAC (and recommends it for other AAC codecs). Industry standards (ANSI/CTA-2075) are moving toward DRC as a baseline requirement.

**Deploying xHE-AAC**
Add additional low-bitrate audio variants (down to 24 kbps) to reach users on constrained networks (e.g., Apple Watch) or low-throughput connections. Use the new `SCORE` attribute in `#EXT-X-MEDIA` tags to prefer an xHE-AAC variant over an equal-bitrate AAC-LC variant — AVPlayer will pick the higher-scored variant when supported.

**Lossless Audio Bitrate Reality**
Lossless codecs cannot be configured to a target bitrate; they consume what the content requires. Stereo lossless at 16-bit/48 kHz averages ~600–1000 kbps, peaking over 1 Mbps. Multichannel lossless at 24-bit/96 kHz averages ~5–6 Mbps, peaking at 8+ Mbps. This is orders of magnitude above AAC rates.

**Multichannel Lossless Strategy**
To enable adaptive scaling into multichannel lossless content, include multichannel AAC variants as intermediate rungs. Multichannel AAC is not universally supported across Apple devices; on unsupported devices it downgrades to stereo. Stereo AAC variants are still required for universal backwards compatibility. Omitting multichannel AAC forces a jump directly from stereo to multi-megabit lossless, causing stalls for users who can't sustain those rates.

## APIs & Frameworks

### HTTP Live Streaming (HLS)
- `#EXT-X-STREAM-INF` — variant stream tag; includes `CODECS` attribute
- `#EXT-X-MEDIA` — alternate rendition tag; includes `CHANNELS` and `SCORE` attributes
- `CODECS="mp4a.40.42"` — xHE-AAC codec string **[NEW]**
- `CODECS="fLaC"` — FLAC codec string (case-sensitive) **[NEW]**
- `CODECS="alac"` — Apple Lossless codec string **[NEW]**
- `SCORE` attribute on `#EXT-X-MEDIA` **[NEW]** — preference score; AVPlayer prefers higher-scored variant
- `CHANNELS` attribute on `#EXT-X-MEDIA` — number of audio channels

### AVFoundation
- `AVPlayer` — playback engine; handles xHE-AAC mono/stereo, FLAC up to 8ch, Apple Lossless up to 8ch
- HLS Authoring Specification for Apple Devices — the normative reference document for all HLS content

### Codecs Supported (New in 2020 OS Releases via HLS)
- xHE-AAC / USAC (`mp4a.40.42`) — mono/stereo; 24–200 kbps **[NEW in HLS]**
- FLAC (`fLaC`) — up to 8 channels; all sample rates/bit depths **[NEW in HLS]**
- Apple Lossless (`alac`) — up to 8 channels; all sample rates/bit depths **[NEW in HLS]**
- Multichannel AAC — limited Apple device support; stereo fallback on unsupported devices

## Code Highlights

HLS master playlist snippet adding xHE-AAC at 24 kbps alongside existing AAC variants:
```
#EXT-X-MEDIA:TYPE=AUDIO,GROUP-ID="audio",NAME="English xHE-AAC",
  LANGUAGE="en",DEFAULT=NO,AUTOSELECT=YES,
  CODECS="mp4a.40.42",BANDWIDTH=24000,URI="xhe-aac-24k/prog_index.m3u8"

#EXT-X-MEDIA:TYPE=AUDIO,GROUP-ID="audio",NAME="English HE-AAC",
  LANGUAGE="en",DEFAULT=YES,AUTOSELECT=YES,
  CODECS="mp4a.40.5",BANDWIDTH=48000,URI="he-aac-48k/prog_index.m3u8"
```

Using SCORE to prefer a high-fidelity xHE-AAC variant over AAC-LC at similar bitrate:
```
#EXT-X-MEDIA:TYPE=AUDIO,GROUP-ID="audio",NAME="xHE-AAC 94k",
  CODECS="mp4a.40.42",BANDWIDTH=94000,SCORE=2.0,URI="xhe-aac-94k/index.m3u8"

#EXT-X-MEDIA:TYPE=AUDIO,GROUP-ID="audio",NAME="AAC-LC 96k",
  CODECS="mp4a.40.2",BANDWIDTH=96000,SCORE=1.0,URI="aac-lc-96k/index.m3u8"
```

## Takeaways
- Add xHE-AAC (`mp4a.40.42`) variants at 24 kbps to reach users on bandwidth-constrained networks and devices like Apple Watch; use the new `SCORE` attribute to prefer xHE-AAC over equal-bitrate AAC-LC where supported.
- Include DRC metadata in all audio encodings — CMAF mandates it for xHE-AAC, and industry standards are moving toward it universally.
- When offering multichannel lossless (FLAC or Apple Lossless), always include multichannel AAC variants as intermediate bitrate rungs to enable adaptive streaming; omitting them forces a jump from stereo to multi-megabit lossless, causing stalls.
- FLAC channel order is L/R/C/LFE/Ls/Rs; Apple Lossless channel order is C/L/R/LFE/Ls/Rs — the layouts differ and must be followed correctly for proper speaker assignment.

---
_Source: WWDC20 Session 10158 page (abstract, chapter summaries, code samples, and resource links)._
