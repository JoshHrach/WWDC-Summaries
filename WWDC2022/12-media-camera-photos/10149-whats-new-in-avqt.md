# What's New in AVQT
**WWDC22 · Session 10149** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10149/)

_Platforms:_ macOS Ventura 13, Linux

## Overview
The Advanced Video Quality Tool (AVQT) is a command-line perceptual video quality metric that assesses compression and scaling artifacts, attempting to mimic human quality ratings. First introduced at WWDC21, AVQT supports all AVFoundation-based video formats including SDR and HDR (HDR10, HLG, Dolby Vision), and leverages AVFoundation and Metal for fast video decoding and processing.

WWDC22 brings several major enhancements: interactive HTML-based reports for visualizing quality scores, a time window feature for evaluating specific scenes, extended raw YUV format support, and a new Linux beta release. On the latest M1 Ultra hardware, AVQT can process a 2-hour 4K HEVC movie in just 20 minutes (6x faster than real time).

The new Linux version enables cloud and server-based video quality analysis without moving files, has the same command-line interface as the macOS version, and requires no external dependencies.

## Key Topics

### Interactive HTML Reports
A new `--visualize` flag generates HTML-based interactive reports viewable in Safari. Reports include frame-level and segment-level AVQT score plots, zoom/hover interactivity, and a pie chart showing distribution across five quality categories (Bad, Poor, Fair, Good, Excellent). PSNR scores are also visualized. Reports are shareable without requiring AVQT installation.

### Time Window Feature
Four new command-line arguments let users specify start and end points (by frame index or timecode) in both reference and test videos. This enables focused evaluation of specific scenes and comparison of temporally unaligned videos, running faster than full-video evaluation.

### Extended Raw YUV Format Support
AVQT now supports a total of 20 raw YUV formats covering a wide range of chroma sub-samplings and bit depths. The deprecated `reference-fourcc` and `test-fourcc` flags are replaced with separate flags for chroma-subsampling and bit-depth.

### AVQT for Linux (Beta)
A new Linux beta release supports a wide range of Linux distributions, requires no external dependencies, and is essentially plug-and-play. It mirrors the macOS command-line interface and supports all 20 raw YUV formats. Viewing conditions parameters will be added in a future release.

## APIs & Frameworks

- **AVQT (Advanced Video Quality Tool)** — command-line executable for perceptual video quality assessment **[NEW features]**
- **AVFoundation** — used internally for video decoding
- **Metal** — used internally for video processing acceleration
- `--visualize` flag **[NEW]** — generates interactive HTML report
- `--start-frame` / `--end-frame` (reference and test) **[NEW]** — time window arguments for scene-level evaluation
- Raw YUV format flags **[NEW]** — chroma-subsampling and bit-depth flags replacing deprecated `reference-fourcc`/`test-fourcc`
- AVQT for Linux (beta) **[NEW]** — Linux distribution of AVQT tool

## Code Highlights

```bash
# Generate an AVQT report with interactive visualization
avqt --reference ref.mov --test compressed.mov --visualize

# Evaluate a specific scene by frame range
avqt --reference ref.mov --test compressed.mov \
  --reference-start-frame 270 --reference-end-frame 486 \
  --test-start-frame 270 --test-end-frame 486
```

## Takeaways

- Use `--visualize` to produce shareable HTML reports with interactive AVQT and PSNR quality plots.
- The new time window feature enables fast, targeted analysis of specific scenes or temporally misaligned videos.
- Raw YUV support expanded to 20 formats; deprecated `fourcc` flags replaced with separate chroma and bit-depth flags.
- AVQT is now available as a Linux beta, enabling cloud and server-based video quality analysis without moving files.

---
_Source: WWDC22 Session 10149 page (abstract, chapter summaries, code samples, and resource links)._
