# Explore HLS Variants in AVFoundation
**WWDC21 · Session 10143** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10143/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, tvOS 15

## Overview
This session introduces two new AVFoundation capabilities for working with HLS variant streams in iOS 15: variant inspection (reading video and audio attributes directly from the HLS master playlist) and an expanded, predicate-based download variant selection API that replaces the older download task options with a flexible, composable configuration model.

Variant inspection via `AVAssetVariant` lets apps display accurate content badges (4K, Dolby Vision, Atmos) by reading bitrate, video range, resolution, frame rate, and audio format directly from the master playlist. The download API redesign introduces `AVAssetVariantQualifier`, `AVAssetDownloadContentConfiguration`, and `AVAssetDownloadConfiguration` to give fine-grained control over which HLS variants and media renditions are downloaded for offline playback, including support for primary + auxiliary content configurations and observable download progress via `NSProgress`.

## Key Topics
- **Variant Inspection (NEW in iOS 15):** Access `AVURLAsset.variants` to obtain an array of `AVAssetVariant` objects representing each variant stream in the master playlist. Each variant exposes `peakBitRate`, `averageBitRate`, and nested `videoAttributes` / `audioAttributes` for format-specific properties.
- **AVAssetVariant.VideoAttributes:** `videoRange` (`.sdr`, `.pq`, `.hlg`), `codecTypes`, `presentationSize`, `nominalFrameRate`.
- **AVAssetVariant.AudioAttributes:** `formatIDs` (e.g., `kAudioFormatAppleLossless`), `channelCount`.
- **Variant Qualifier (NEW):** `AVAssetVariantQualifier` wraps an `NSPredicate` expressing variant preferences. Any property of `AVAssetVariant` can be used in the predicate. Custom constructors exist for audio channel count and other non-string properties.
- **Content Configuration (NEW):** `AVAssetDownloadContentConfiguration` pairs a list of `AVAssetVariantQualifier` objects with a list of `AVMediaSelection` objects (audio/subtitle renditions).
- **Download Configuration (NEW):** `AVAssetDownloadConfiguration` is the root object; requires `AVURLAsset` and a display title. It has a `primaryContentConfiguration` and an `auxiliaryContentConfigurations` array. Set `optimizesAuxiliaryContentConfigurations = true` (the default) to prevent duplicate video rendition downloads when auxiliary configs overlap with the primary.
- **Download Task and NSProgress:** Create the download task via `AVAssetDownloadURLSession.makeAssetDownloadTask(downloadConfiguration:)` **[NEW]**. The `AVAssetDownloadTask.progress` property provides an `NSProgress` object for KVO-observable download progress updates.
- **Direct Variant Selection:** Pass a specific `AVAssetVariant` directly to `AVAssetVariantQualifier(variant:)` for cases where business logic makes predicate-based selection impractical.

## APIs & Frameworks

**AVFoundation**

_Variant Inspection_
- `AVURLAsset.variants: [AVAssetVariant]` **[NEW]** – All variant streams from the HLS master playlist
- `AVAssetVariant` **[NEW]** – Represents one HLS EXT-X-STREAM-INF entry
  - `peakBitRate: Double` – Peak bandwidth
  - `averageBitRate: Double?` – Average bandwidth
  - `videoAttributes: AVAssetVariant.VideoAttributes?` – Video-specific properties
  - `audioAttributes: AVAssetVariant.AudioAttributes?` – Audio-specific properties
- `AVAssetVariant.VideoAttributes` **[NEW]**
  - `videoRange: AVVideoRange` – `.sdr`, `.pq` (HDR10/Dolby Vision), `.hlg`
  - `codecTypes: [CMVideoCodecType]` – e.g., `avc1`, `hvc1`, `dvh1`
  - `presentationSize: CGSize` – Resolution
  - `nominalFrameRate: Float` – Frame rate
- `AVAssetVariant.AudioAttributes` **[NEW]**
  - `formatIDs: [AudioFormatID]` – e.g., `kAudioFormatMPEGLayerIII`, `kAudioFormatAppleLossless`
  - `channelCount: Int` – Channel count
- `AVVideoRange` **[NEW]** – `.sdr`, `.pq`, `.hlg`

_Download API_
- `AVAssetVariantQualifier` **[NEW]** – Expresses variant preferences
  - `AVAssetVariantQualifier(predicate: NSPredicate)` **[NEW]** – NSPredicate-based constructor
  - `AVAssetVariantQualifier(variant: AVAssetVariant)` **[NEW]** – Direct variant selection
- `AVAssetDownloadContentConfiguration` **[NEW]** – Pairs variant qualifiers with media selections
  - `variantQualifiers: [AVAssetVariantQualifier]` – Ordered preference list
  - `mediaSelections: [AVMediaSelection]` – Audio/subtitle renditions to download
- `AVAssetDownloadConfiguration` **[NEW]** – Root download configuration
  - `AVAssetDownloadConfiguration(asset:title:)` **[NEW]** – Designated initializer
  - `primaryContentConfiguration: AVAssetDownloadContentConfiguration` – Primary video/audio/subtitle set
  - `auxiliaryContentConfigurations: [AVAssetDownloadContentConfiguration]` – Additional renditions (e.g., lossless audio)
  - `optimizesAuxiliaryContentConfigurations: Bool` – Avoids duplicate video downloads (default `true`)
- `AVAssetDownloadURLSession.makeAssetDownloadTask(downloadConfiguration:)` **[NEW]** – Creates download task from new config model
- `AVAssetDownloadTask.progress: NSProgress` **[NEW]** – KVO-observable download progress

## Code Highlights
Inspecting variant attributes to surface content badges:
```swift
let asset = AVURLAsset(url: masterPlaylistURL)
for variant in asset.variants {
    let isHDR  = variant.videoAttributes?.videoRange == .pq
    let isAtmos = variant.audioAttributes?.channelCount ?? 0 >= 16
    let is4K   = variant.videoAttributes?.presentationSize.width ?? 0 >= 3840
    print("HDR: \(isHDR), Atmos: \(isAtmos), 4K: \(is4K)")
}
```

Predicate-based download configuration (primary HDR <5 Mbps + auxiliary lossless audio):
```swift
let asset = AVURLAsset(url: URL(string: "https://example.com/master.m3u8")!)
let dwConfig = AVAssetDownloadConfiguration(asset: asset, title: "My Movie")

// Primary: HDR under 5 Mbps, English + French audio + English subtitles
let varPred = NSPredicate(format: "videoAttributes.videoRange == %@ && peakBitRate < 5000000",
                          argumentArray: [AVVideoRange.pq])
dwConfig.primaryContentConfiguration.variantQualifiers = [AVAssetVariantQualifier(predicate: varPred)]
dwConfig.primaryContentConfiguration.mediaSelections = [enAudioMS, frAudioMS, enSubtitleMS]

// Auxiliary: lossless audio (English only)
let auxPred = NSPredicate(format: "%d IN audioAttributes.formatIDs",
                          argumentArray: [kAudioFormatAppleLossless])
let auxConfig = AVAssetDownloadContentConfiguration()
auxConfig.variantQualifiers = [AVAssetVariantQualifier(predicate: auxPred)]
auxConfig.mediaSelections = [enAudioMS]
dwConfig.auxiliaryContentConfigurations = [auxConfig]
dwConfig.optimizesAuxiliaryContentConfigurations = true

// Create and start the download task
let downloadTask = avurlSession.makeAssetDownloadTask(downloadConfiguration: dwConfig)
downloadTask.resume()

// Observe progress
let progress = downloadTask.progress
// progress is KVO-observable via NSProgress
```

## Takeaways
- `AVURLAsset.variants` makes it possible to display accurate HDR/Atmos/4K badges without out-of-band metadata—inspect `videoAttributes.videoRange` and `audioAttributes.channelCount` directly.
- The new predicate-based variant qualifier API replaces per-option flags with composable `NSPredicate` expressions, enabling arbitrarily complex variant selection logic from a single configuration object.
- Always set `optimizesAuxiliaryContentConfigurations = true` (already the default) when providing auxiliary configs; skipping this can cause the same video stream to be downloaded twice.
- `AVAssetDownloadTask.progress` is an `NSProgress` object—subscribe with KVO or bind to a `UIProgressView`/`NSProgressIndicator` for standard, system-consistent progress reporting.

---
_Source: WWDC21 Session 10143 page (abstract, transcript, and code samples)._
