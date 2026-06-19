# Improve Stream Authoring with HLS Tools
**WWDC20 · Session 10225** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10225/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, tvOS 14

## Overview
This session covers updates to the Apple HLS (HTTP Live Streaming) authoring tools for 2020, focusing on three areas: tool suite changes (Low-Latency tools merged back into the main distribution, Media Stream Validator immediate error reporting, HLS Report rewritten as a compiled binary), a practical walkthrough of setting up a Low-Latency HLS test stream using `TS recompressor` and `Media Stream Segmenter`, and guidelines for constructing master playlists with multiple audio codec groups including the new USAC (xHE-AAC), Apple Lossless, FLAC, and Dolby Digital Plus codecs.

A new `SCORE` attribute for `EXT-X-STREAM-INF` variants is introduced to control variant preference ordering when BANDWIDTH-based selection would choose the wrong codec group.

## Key Topics

**HLS Tool Suite Changes**
- Low-Latency HLS tools previously in a separate package are now merged back into the single tool distribution **[NEW]**
- `Media Stream Validator`: new `--immediate` option reports errors as they are detected rather than waiting until the end of validation — critical for long-running live stream validation **[NEW]**
- `HLS Report`: rewritten from Python script to compiled binary; one deprecated option dropped, all other functionality preserved **[NEW]**
- New audio codec support added to tools (USAC/xHE-AAC, Apple Lossless, FLAC)

**Low-Latency HLS Tool Flow**
- `TS recompressor` — generates and recompresses transport stream input; can use hardware encoder; outputs multiple bitrate variants to separate UDP ports
- `Media Stream Segmenter` — processes each variant's UDP stream; new `--part-target-duration-ms` option activates partial segment generation for Low-Latency HLS **[NEW]**; new `--date-time` option adds `EXT-X-PROGRAM-DATE-TIME` tag **[NEW]**
- Delivery Directives Interface — interprets HLS query parameters (`_HLS_msn`, `_HLS_part`, `_HLS_skip`) and serves synthesized playlists:
  - Go implementation: runs as its own HTTP server; given parent directory of all variants
  - PHP implementation: runs under existing HTTP server; one instance per variant; both expect variants in subdirectories of a common parent

**Audio Codec CODECS Tags**
- USAC / xHE-AAC: `mp4a.40.42`
- Apple Lossless: `alac`
- FLAC: `fLaC` (note the unusual mixed case — not a typo)
- AC-3 / Dolby Digital: existing `ac-3` / `ec-3`
- Standard AAC-LC: `mp4a.40.2`; HE-AAC: `mp4a.40.5`; HE-AACv2: `mp4a.40.29`

**Master Playlist Audio Group Best Practices**
- Always include a stereo AAC group — universal fallback for all devices
- For multichannel lossless (Apple Lossless, FLAC): include a multichannel AAC group as well (lossless bit rates may be too high for some conditions)
- Never switch the channel count — if a variant starts multichannel, keep it multichannel
- Switchable codec groups (freely switches between): AAC-LC, HE-AAC, HE-AACv2, USAC
- Non-switchable codec groups (requires explicit pairing): lossless ↔ AAC (allowed), AC-3 ↔ AAC (not switched automatically)
- For switchable codecs: spread audio groups across variants (low bit rate audio with low bit rate video, high with high)
- For non-switchable codecs: replicate the variant entry for every video bitrate level (new master playlist entries only; no new media playlists)

**SCORE Attribute (New)**
- Applied to `EXT-X-STREAM-INF` and `EXT-X-I-FRAME-STREAM-INF` tags **[NEW]**
- Floating-point value; already supported — no tool upgrade needed to use
- Must be set on ALL variants — scoring is all-or-nothing; partial scoring creates indeterminate behavior
- Selection logic: standard filtering (format compatibility, bandwidth) runs first → among the surviving variants, highest SCORE wins
- Use case: USAC variants should score higher than HE-AAC variants so USAC is preferred when playable; HE-AAC serves as fallback when device cannot play USAC

## APIs & Frameworks

### HLS Tools (Command Line)
- `TS recompressor` **[ADDED to main distribution]**
  - `--generate` — use the tool's own test signal generator
  - `--hardware-encoder` — prefer hardware encoding where available
  - `--output <bitrate> <udp-port>` — define a variant output; can specify multiple
- `Media Stream Segmenter` **[UPDATED]**
  - `--part-target-duration-ms <ms>` **[NEW]** — enables Low-Latency partial segment generation
  - `--target-duration <s>` — segment target duration
  - `--max-count <n>` — number of segments to retain in playlist
  - `--delete-segments` — remove expired segments
  - `--date-time` **[NEW]** — adds `EXT-X-PROGRAM-DATE-TIME` tag
  - `--output-dir <path>` — output directory for segments and playlist
- `Media Stream Validator` **[UPDATED]**
  - `--immediate` **[NEW]** — report errors immediately rather than at end of validation run
- `HLS Report` **[UPDATED]** — rewritten as compiled binary; same interface minus one deprecated flag

### HLS Playlist Tags and Attributes
- `EXT-X-STREAM-INF` — variant stream declaration in master playlist
  - `SCORE=<float>` **[NEW]** — variant preference score; higher wins after filtering
  - `BANDWIDTH=<bps>` — peak bandwidth
  - `AVERAGE-BANDWIDTH=<bps>` — average bandwidth
  - `CODECS="<codec-string>"` — required; must precisely match actual codec
  - `AUDIO="<group-id>"` — references an `EXT-X-MEDIA` audio group
- `EXT-X-MEDIA` — audio rendition declaration
  - `TYPE=AUDIO`
  - `GROUP-ID="<id>"` — links renditions into a switchable group
  - `LANGUAGE`, `NAME`, `DEFAULT`, `AUTOSELECT`
  - `CHANNELS="<count>"` — stereo="2", 5.1="6"
- `EXT-X-PROGRAM-DATE-TIME` — wall-clock time of first sample in segment **[added via --date-time option]**
- `EXT-X-RENDITION-REPORT` — required by Low-Latency HLS delivery directives; links to sibling variant playlist states

## Code Highlights

TS recompressor generating three bitrate variants:
```bash
ts_recompressor --generate --hardware-encoder \
  --output 500000   udp://127.0.0.1:50000 \
  --output 2000000  udp://127.0.0.1:50001 \
  --output 4000000  udp://127.0.0.1:50002
```

Media Stream Segmenter for a Low-Latency variant:
```bash
media_stream_segmenter \
  --part-target-duration-ms 250 \
  --target-duration 2 \
  --max-count 10 \
  --delete-segments \
  --date-time \
  --output-dir /var/www/hls/low/ \
  udp://127.0.0.1:50001
```

Master playlist excerpt with SCORE for USAC preference:
```
#EXTM3U
#EXT-X-MEDIA:TYPE=AUDIO,GROUP-ID="aac",LANGUAGE="en",NAME="English",DEFAULT=YES,URI="en_aac/playlist.m3u8"
#EXT-X-MEDIA:TYPE=AUDIO,GROUP-ID="usac",LANGUAGE="en",NAME="English",DEFAULT=YES,URI="en_usac/playlist.m3u8"

#EXT-X-STREAM-INF:BANDWIDTH=2500000,CODECS="avc1.640020,mp4a.40.5",AUDIO="aac",SCORE=1.0
low/playlist.m3u8
#EXT-X-STREAM-INF:BANDWIDTH=2700000,CODECS="avc1.640020,mp4a.40.42",AUDIO="usac",SCORE=2.0
low/playlist.m3u8
#EXT-X-STREAM-INF:BANDWIDTH=4500000,CODECS="avc1.640028,mp4a.40.5",AUDIO="aac",SCORE=1.0
high/playlist.m3u8
#EXT-X-STREAM-INF:BANDWIDTH=4700000,CODECS="avc1.640028,mp4a.40.42",AUDIO="usac",SCORE=2.0
high/playlist.m3u8
```

## Takeaways
- Always include a stereo AAC audio group as a universal fallback alongside any higher-quality codec group (USAC, Dolby, lossless); never design a master playlist with only a non-AAC codec group.
- Use the new `SCORE` attribute when BANDWIDTH-based selection would favor the wrong codec group — set higher SCORE on preferred variants (e.g., USAC) and lower on fallbacks (HE-AAC); the attribute must be set on every variant or it has no effect.
- For Low-Latency HLS test streams, `TS recompressor` + `Media Stream Segmenter` with `--part-target-duration-ms` is the simplest path to a working multi-variant setup; place all variant output directories under a shared parent for the Delivery Directives scripts to find sibling variants automatically.
- The `FLAC` CODECS attribute has unusual mixed-case capitalization (`fLaC`) — this is correct per the spec and not a typo.

---
_Source: WWDC20 Session 10225 page (abstract, transcript, and resource links)._
