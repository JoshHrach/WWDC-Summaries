# Validating HTTP Live Streams
**WWDC16 · Session 510** · [Watch](https://developer.apple.com/videos/play/wwdc2016/510/)

_Platforms:_ iOS 9–10, macOS El Capitan / Sierra 10.12, tvOS 9–10

## Overview
This session by Apple media engineer Eryk covers the tools and specifications developers must use to validate their HTTP Live Streaming (HLS) content before shipping. Validation ensures playlists and media segments are structurally correct, meet Apple's authoring requirements, and follow established best practices — all in service of reliable playback across devices and network conditions.

Two key reference documents govern HLS content: the IETF HLS specification ("draft-pantos") covering playlist structure and tag syntax, and the newer **HLS Authoring Specification** (published in the Apple developer reference library and updated for WWDC 2016 to cover iOS and macOS in addition to tvOS). The session walks through the command-line validation toolchain, explaining each tool's purpose, options, and output format, with annotated real-world report examples.

## Key Topics

### HLS Specifications
- **IETF HLS Specification** ("draft-pantos"): defines playlist structure, tags, and client/server responsibilities. Updated 2–3 times per year for seven years.
- **HLS Authoring Specification** [updated WWDC 2016 to cover iOS/macOS]: covers correct stream structure, encoding requirements (H.264 video, ≤60 fps), I-Frame playlist requirements, trick-play support, delivery bitrate recommendations, and privacy/security guidance.
- Validation checks: (1) playlist and segment formatting (parseable?), (2) additional Apple requirements (I-Frame playlists required), (3) best practices (bitrate declarations, default variant selection, captions).

### HLS Tools Download
- Available at `developer.apple.com/streaming` (requires free Apple Developer account).
- Installer package (macOS only) provides 7 tools:
  - `mediastreamsegmenter` — segments live streams
  - `mediafilesegmenter` — segments file-based video; produces media playlist + segments; supports encryption, naming, playlist structure options
  - `segmentedsubtitler` — segments WebVTT subtitle files into per-segment WebVTT
  - `mediasubtitlesegmenter` — subtitle segmentation variant
  - `variantplaylistcreator` — creates master (multivariant) playlists
  - `id3taggenerator` — formats timed ID3 metadata for insertion by segmenters
  - `mediafilevalidator` (covered in session): validates file-based streams
  - `mediastreamvalidator` (primary tool): validates live and VOD streams

### mediastreamvalidator
- Usage: `mediastreamvalidator -s <output.json> <master_playlist_url>`
- Downloads playlists and segments; validates structure and authoring spec requirements.
- Output: terminal trace + JSON file containing all validation data.
- Default timeout: 5 minutes (sufficient for most VOD streams; increase for large files or extended live validation).
- Key options: `-s <path>` (JSON output path), `-t <seconds>` (timeout override).
- JSON output: top-level dictionary with a `variants` array; each entry keyed by `dataId`; can exceed 100,000 lines for large streams.

### hlsreport (Python script)
- Usage: `hlsreport <mediastreamvalidator_output.json>`
- Converts JSON validation output into a human-readable HTML report.
- Key options: `--id` (adds `dataId` column linking HTML rows to JSON entries), `-v` / `--verbose` (detailed per-variant output), `-p` / `--playlist` (adds declared vs. measured bitrate columns), output path override.
- Report sections:
  1. **Stream type** — VOD, LIVE, EVENT, UNKNOWN (parse failure), or MIXED (conflicting types across variants)
  2. **Variant overview table** — one row per video variant; columns include max/average bitrate, percent difference (declared vs. measured), percent processed (VOD), or average segment count + playlist duration (LIVE)
  3. **Rendition overview table** — audio/subtitle renditions
  4. **I-Frame variant overview** — includes scaled average data rate and multiplier (effective worst-case at trick-play speed relative to 1× rate)
  5. **Must Fix issues** — required for correct playback (e.g., declared bitrate >10% off measured, missing I-Frame playlist)
  6. **Should Fix issues** — best-practice violations (e.g., no captions, wrong default variant)
  7. **Report information** — tool versions for reproducible bug reports

### Common Must-Fix Issues
- **Declared bitrate mismatch**: `BANDWIDTH` or `AVERAGE-BANDWIDTH` in master playlist more than ±10% from measured value → incorrect variant selection by client.
  - Positive percent difference: declared value too low → increase it.
  - Negative percent difference: declared value too high → decrease it.
- **No I-Frame playlist**: Apple requires I-Frame playlists for trick play (fast-forward, rewind) even though the HLS spec makes them optional.
- **Wrong default variant**: the first variant in the master playlist is the one clients play initially — it should not be the highest bitrate; a mid-range variant is appropriate.
- **Mixed stream type**: some variants are EVENT, others LIVE → conflicting master playlist.

### Common Should-Fix Issues
- No captions or subtitles (subtitle rendition missing).
- I-Frame scaled average data rate exceeds corresponding normal variant's rate for same resolution → trick play may cause resolution downgrade.

### Multi-Channel Audio Example
- When multiple audio codecs (AAC stereo, AC-3/Dolby Digital, EC-3/Dolby Digital Plus) are included:
  - Each codec group gets its own default variant (not substitutable across codecs).
  - Combined variant bitrate includes the audio rendition bitrate.
  - AC-3 and EC-3 renditions appear in the rendition overview with their own group IDs.

### Submission Requirements
- Validate streams before App Store submission.
- Include a test stream URL in the App Store Connect review notes field so reviewers can test playback.

## APIs & Frameworks

This session covers command-line tools and specifications, not iOS/macOS SDK APIs.

- `mediastreamvalidator` — primary HLS validation tool (macOS CLI)
  - `-s <json_path>` — output JSON file path
  - `-t <seconds>` — validation timeout (default: 300 seconds)
- `hlsreport` — Python script; converts validation JSON to HTML report
  - `--id` — include dataId column for JSON cross-reference
  - `-p` / `--playlist` — add declared vs. measured bitrate columns
  - `-v` / `--verbose` — per-variant detailed output
- `mediafilesegmenter` — file-based HLS segmenter; accepts a movie file; produces media playlist + `.ts` segments
- `mediastreamsegmenter` — live stream segmenter
- `segmentedsubtitler` / `mediasubtitlesegmenter` — WebVTT subtitle segmenters
- `variantplaylistcreator` — generates master (multivariant) playlists
- `id3taggenerator` — formats timed ID3 metadata
- **HLS Authoring Specification** [NEW document coverage for iOS/macOS] — available at developer.apple.com reference library; defines requirements for video encoding (H.264, ≤60 fps), I-Frame playlists, trick-play, segment duration, delivery bitrates, privacy/security
- **IETF HLS Specification** ("draft-pantos") — normative playlist grammar and tag definitions; searchable as "draft pantos" on the web
- HLS playlist tags relevant to validation: `#EXT-X-STREAM-INF` (with `BANDWIDTH`, `AVERAGE-BANDWIDTH`), `#EXT-X-I-FRAME-STREAM-INF`, `#EXT-X-MEDIA`, `#EXT-X-PLAYLIST-TYPE` (VOD/LIVE/EVENT), `#EXT-X-TARGETDURATION`
- Audio codecs: AAC-LC (stereo), AC-3 (Dolby Digital), EC-3 (Dolby Digital Plus); separate rendition groups required when mixing

## Code Highlights

Command-line validation workflow:
```bash
# Step 1: Run mediastreamvalidator (may take several minutes for large VOD)
mediastreamvalidator -s output.json https://example.com/master.m3u8

# Step 2: Generate HTML report
hlsreport output.json
# Produces output.html

# Step 3: With extra options for bitrate columns and JSON cross-reference
hlsreport --id -p output.json
```

## Takeaways
- Validate every HLS stream before App Store submission using `mediastreamvalidator` + `hlsreport`; structural errors that escape validation cause hard-to-debug playback failures on user devices.
- The **HLS Authoring Specification** (updated WWDC 2016 for iOS/macOS) is the definitive Apple-specific requirements document — I-Frame playlists are required even though the IETF spec makes them optional, and declared bitrate must be within ±10% of measured bitrate.
- The first variant in a master playlist is the client's default starting point — it must not be the highest-bitrate variant; choose a mid-range bandwidth variant appropriate for typical network conditions.
- Always include a test stream URL in App Store Connect review notes so App Review can validate playback independently.

---
_Source: WWDC16 Session 510 page (abstract, transcript, and resource links)._
