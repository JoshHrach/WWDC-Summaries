# Export HDR Media in Your App with AVFoundation
**WWDC20 · Session 10010** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10010/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11

## Overview
This session introduces HDR video export capabilities in AVFoundation, available to third-party developers for the first time in 2020 (previously only accessible to Final Cut Pro X and Compressor). It covers the fundamentals of High Dynamic Range video — transfer functions, color gamuts, metadata formats — and explains how to preserve HDR content through export using both `AVAssetExportSession` and `AVAssetWriter`.

`AVAssetExportSession` with HEVC or Apple ProRes presets now automatically preserves the HDR format of source assets with no code changes required. For more control, `AVAssetWriter` with `AVOutputSettingsAssistant` provides a structured path to configure HDR encoding parameters including transfer function, color primaries, YCbCr matrix, and HEVC profile level.

## Key Topics

**HDR Fundamentals**
- SDR: up to 100 nits; HDR: up to 10,000 nits
- Transfer functions: OETF (encode) and EOTF (decode)
- HLG (Hybrid Log-Gamma): scene-referred, backwards-compatible with SDR displays
- PQ (Perceptual Quantizer): display-referred; basis for Dolby Vision and HDR10
- Color gamuts: BT.709/sRGB < P3 < BT.2020; HDR typically paired with BT.2020 wide gamut
- HDR metadata types: none (HLG), static (HDR10 — MDCV + CLLI SEI messages), dynamic (Dolby Vision)

**AVAssetExportSession HDR Export**
- HEVC presets (`AVAssetExportPresetHEVCHighestQuality`, etc.) now preserve source HDR format (HDR10→HDR10, HLG→HLG, SDR→SDR) with zero code changes **[NEW]**
- Apple ProRes presets also preserve HDR format **[NEW]**
- H.264 presets convert HDR→SDR (recommended for backwards compatibility)
- Mixed HDR/SDR compositions: export auto-converts SDR to HDR
- Mixed HDR formats: HLG preferred over PQ; formats with metadata preferred over those without

**AVAssetWriter HDR Export**
- Full control over codec, bitrate, frame rate, color space, and dynamic range
- `AVVideoCodecKey`: must be HEVC or Apple ProRes for HDR
- `AVVideoColorPropertiesKey`: dictionary with transfer function, color primaries, YCbCr matrix
- `AVVideoCompressionPropertiesKey`: includes `AVVideoProfileLevelKey` = `HEVC_Main10_AutoLevel` (required for HEVC HDR; 8-bit HEVC HDR not supported)
- HLG settings: transfer function = `.itur_2100_HLG`, primaries/matrix = `.itur_2020`
- HDR10 settings: transfer function = `.itur_2100_PQ`, plus optional MDCV and CLLI metadata

**Platform Hardware Support**
- iOS/iPadOS: HEVC hardware encoding on A10 Fusion or newer (iPhone 7+, iPad 2018+, iPod touch 2019+)
- macOS: HEVC and ProRes software encoding on all Macs; HEVC hardware encoding on 2017+ Macs
- Hardware encoding delivers significantly faster export

## APIs & Frameworks

### AVFoundation — AVAssetExportSession
- `AVAssetExportSession(asset:presetName:)` — creates export session
- `AVAssetExportPresetHEVCHighestQuality` — HEVC preset; now preserves HDR **[UPDATED]**
- `AVAssetExportPresetHEVCHighestQualityWithAlpha` — HEVC with alpha **[UPDATED]**
- `AVAssetExportPresetAppleProRes4444` / `AVAssetExportPresetAppleProRes422LPCM` — ProRes presets; preserve HDR **[UPDATED]**
- `AVAssetExportPresetHighestQuality` (H.264) — converts HDR to SDR
- `exportSession.outputURL` — destination file URL
- `exportSession.outputFileType` — e.g., `.mov`, `.mp4`
- `exportSession.exportAsynchronously(completionHandler:)` — begins export

### AVFoundation — AVAssetWriter
- `AVAssetWriter(url:fileType:)` — creates writer
- `AVAssetWriterInput(mediaType:outputSettings:sourceFormatHint:)` — input with format hint for auto-configuration
- `AVOutputSettingsAssistant(preset:)` — preset-based settings builder
- `AVOutputSettingsAssistant.sourceVideoFormat` — set before accessing `videoSettings` for HDR-aware defaults
- `AVOutputSettingsAssistant.videoSettings` — returns configured settings dictionary

### AVFoundation — Output Settings Keys
- `AVVideoCodecKey` — `AVVideoCodecType.hevc` or `AVVideoCodecType.proRes4444` / `proRes422`
- `AVVideoColorPropertiesKey` — dictionary containing:
  - `AVVideoTransferFunctionKey`:
    - `AVVideoTransferFunction_ITU_R_2100_HLG` — HLG transfer function **[NEW]**
    - `AVVideoTransferFunction_SMPTE_ST_2084_PQ` — PQ transfer function (HDR10/Dolby Vision)
  - `AVVideoColorPrimariesKey`:
    - `AVVideoColorPrimaries_ITU_R_2020` — BT.2020 wide color gamut
  - `AVVideoYCbCrMatrixKey`:
    - `AVVideoYCbCrMatrix_ITU_R_2020` — BT.2020 non-constant luminance
- `AVVideoCompressionPropertiesKey` — dictionary containing:
  - `AVVideoProfileLevelKey`: `kVTProfileLevel_HEVC_Main10_AutoLevel` — required for HEVC HDR
- `AVVideoMasteringDisplayColorVolumeKey` — MDCV SEI data (HDR10 optional but recommended)
- `AVVideoContentLightLevelInfoKey` — CLLI SEI data (HDR10 optional but recommended)

### AVFoundation — Supporting Types
- `AVComposition` — temporal combination of track segments for export
- `AVVideoComposition` — spatial/rendering composition
- `AVAssetReader` — read samples from asset for AVAssetWriter pipeline

## Code Highlights

Minimal HEVC HDR export (no changes needed vs SDR):
```swift
guard let exportSession = AVAssetExportSession(
    asset: sourceAsset,
    presetName: AVAssetExportPresetHEVCHighestQuality) else { return }
exportSession.outputURL = outputURL
exportSession.outputFileType = .mov
exportSession.exportAsynchronously { /* handle completion */ }
```

AVAssetWriter with source format hint (auto-configures HDR):
```swift
let assetWriter = try AVAssetWriter(url: outputURL, fileType: .mov)
let outputSettings: [String: Any] = [AVVideoCodecKey: AVVideoCodecType.hevc]
let input = AVAssetWriterInput(
    mediaType: .video,
    outputSettings: outputSettings,
    sourceFormatHint: videoFormatDescription)
assetWriter.add(input)
```

AVAssetWriter with AVOutputSettingsAssistant for HDR10:
```swift
let assistant = AVOutputSettingsAssistant(preset: .hevc1920x1080)!
assistant.sourceVideoFormat = videoFormatDescription
var settings = assistant.videoSettings!
// Add HDR10 color properties
settings[AVVideoColorPropertiesKey] = [
    AVVideoTransferFunctionKey: AVVideoTransferFunction_SMPTE_ST_2084_PQ,
    AVVideoColorPrimariesKey: AVVideoColorPrimaries_ITU_R_2020,
    AVVideoYCbCrMatrixKey: AVVideoYCbCrMatrix_ITU_R_2020
]
settings[AVVideoCompressionPropertiesKey] = [
    AVVideoProfileLevelKey: kVTProfileLevel_HEVC_Main10_AutoLevel
]
```

## Takeaways
- For most apps using HEVC or ProRes export presets: HDR is preserved automatically with no code changes — just ensure you are using a HEVC or ProRes preset.
- When using `AVAssetWriter`, always set `sourceVideoFormat` on `AVOutputSettingsAssistant` before fetching `videoSettings` to get HDR-aware defaults.
- HDR10 requires PQ transfer function, BT.2020 primaries/matrix, HEVC Main10 profile, and optionally MDCV + CLLI SEI metadata for accurate display rendering.
- H.264 presets intentionally convert HDR to SDR — use them deliberately for maximum backwards compatibility, not as the default.

---
_Source: WWDC20 Session 10010 page (abstract, chapter summaries, code samples, and resource links)._
