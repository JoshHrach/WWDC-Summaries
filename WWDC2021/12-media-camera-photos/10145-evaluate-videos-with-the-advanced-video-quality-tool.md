# Evaluate Videos with the Advanced Video Quality Tool
**WWDC21 · Session 10145** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10145/)

_Platforms:_ macOS Monterey 12

## Overview
This session introduces the Advanced Video Quality Tool (AVQT), a new macOS command-line utility that uses perceptual quality modeling to score compressed video files against a reference source. Unlike traditional metrics such as PSNR or SSIM, which can give identical scores to perceptually very different videos, AVQT correlates closely with human subjective quality ratings across diverse content types including animation, natural scenes, and sports.

AVQT is built on AVFoundation (supporting all its video formats including SDR and HDR variants) and uses Metal for GPU-accelerated computation, achieving over 175 frames per second on 1080p content. This means a 10-minute 1080p video can be analyzed in under 1.5 minutes. The tool is viewing-setup-aware, accepting display size, resolution, and viewing distance parameters that influence the predicted perceptual score. A key workflow application is tuning HLS delivery tier bitrates for specific content by using AVQT scores as quality feedback.

## Key Topics
- **What AVQT Measures:** Outputs a floating-point quality score from 1–5 (mimicking subjective MOS-style ratings) at both frame level and segment level (default segment = 6 seconds). Frame scores are aggregated into segment scores via configurable temporal pooling.
- **Why PSNR/SSIM Falls Short:** Both metrics can yield identical scores for videos with vastly different perceptual quality. AVQT is validated against publicly available human-subject datasets (Waterloo IVC 4K, VQEG HD3) with high Pearson Correlation Coefficient (PCC) and low RMSE.
- **Metal-Accelerated Processing:** All preprocessing (decoding, scaling) is handled natively by AVQT; no need to pre-decode to raw pixel formats. Metal offloads pixel-level computation to the GPU.
- **AVFoundation Format Support:** SDR, HDR10, HLG, and Dolby Vision video formats are all supported directly.
- **Viewing Setup Awareness:** `--viewing-distance` and `--display-resolution` flags account for the fact that artifact visibility depends on how the viewer is watching; at greater viewing distances, perceived quality is higher.
- **HLS Bitrate Optimization Workflow:** Start with HLS Authoring Specification target bitrates, encode the content, run AVQT, compare scores against a quality threshold (e.g., 4.5 for a 4K tier), and iteratively adjust bitrates based on feedback.

## APIs & Frameworks
- **AVQT (Advanced Video Quality Tool)** **[NEW]** – macOS command-line executable; available via Developer Downloads Portal
  - Input: `--reference` (source video file), `--test` (compressed video file), `--output` (CSV output file path)
  - `--segment-duration` – Duration of each scored segment (default 6 seconds)
  - `--temporal-pooling` – Controls how frame scores aggregate to segment scores
  - `--viewing-distance` – Distance from viewer to screen (in multiples of screen height H, e.g., `1.5H`, `3H`)
  - `--display-resolution` – Target display resolution for viewing setup awareness
  - Output CSV: frame-level scores + segment-level scores
- **AVFoundation** – Used internally by AVQT for video decoding and format support
  - `AVAssetWriter` – Referenced as one typical upstream compression workflow
  - Supported formats: SDR, HDR10, HLG, Dolby Vision
- **Metal** – Used internally by AVQT for GPU-accelerated pixel-level quality computation
- **Compressor** – Referenced as one typical upstream compression workflow
- **HLS (HTTP Live Streaming)** – Primary target use case; HLS Authoring Specification referenced for initial bitrate targets

## Code Highlights
Basic AVQT invocation from the command line:
```bash
# Check installation
which AVQT

# View all available flags
AVQT --help

# Compute quality scores for a compressed video against its source
AVQT --reference sample_reference.mov \
     --test sample_compressed.mov \
     --output sample_output.csv
```

Output CSV structure:
```
frame_number, score
1, 4.2
2, 4.1
...
[Segment 1 score: 4.15]
```

Viewing-setup-aware evaluation:
```bash
AVQT --reference source.mov \
     --test compressed.mov \
     --output output.csv \
     --viewing-distance 1.5H \
     --display-resolution 3840x2160
```

## Takeaways
- AVQT is the first Apple-provided objective perceptual video quality metric tool, validated against human-subject datasets with high correlation accuracy across content types where PSNR and SSIM fail.
- Its Metal-powered GPU processing and native AVFoundation decoding mean no raw decode pre-processing step is needed, making it practical for large-volume video catalog quality screening.
- Viewing-setup parameters are essential for accurately modeling quality in real delivery contexts—particularly for 4K content where viewing distance dramatically affects perceived artifact severity.
- The recommended workflow is to use AVQT scores as a feedback loop when tuning HLS tier bitrates: set a quality threshold, encode, score, and iterate bitrate until the score meets the threshold for each content type.

---
_Source: WWDC21 Session 10145 page (abstract, chapter summaries, code samples, and resource links)._
