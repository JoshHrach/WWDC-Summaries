# Author Fragmented MPEG-4 Content with AVAssetWriter
**WWDC20 · Session 10011** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10011/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, tvOS 14

## Overview
AVAssetWriter gains new capabilities in 2020 to produce fragmented MPEG-4 (fMP4) files suitable for HLS streaming. Previously, AVAssetWriter only wrote complete movie files; it now supports outputting media data as a series of fMP4 segments — the preferred container format for both Apple HLS and CMAF — via a delegate-based streaming API.

The session covers the fMP4 box structure (file type box, movie box, interleaved movie fragment + media data boxes), how the delegate protocol delivers initialization segments and separable segments, passthrough vs. encoding modes, custom segmentation, audio priming compensation for exact A/V sync, and how to build a playlist alongside the segment data. A companion sample app (`fmp4Writer`) demonstrates the full workflow.

## Key Topics
- **fMP4 vs. regular MP4** — Regular MP4 has a movie box containing all sample references followed by one media data box; fMP4 interleaves movie fragment boxes (`moof`) with media data boxes (`mdat`), making it suitable for streaming without seeking. The movie box in fMP4 contains no sample references.
- **`AVAssetWriter(contentType:)`** **[NEW]** — New initializer taking a `UTType` instead of a file URL; writer outputs data via delegate rather than writing to a file.
- **`AVAssetWriter.outputFileTypeProfile`** **[NEW]** — `.mpeg4AppleHLS` or `.mpeg4CMAF`; selecting either triggers fMP4 output mode.
- **`AVAssetWriter.preferredOutputSegmentInterval`** **[NEW]** — `CMTime` specifying target segment duration; in encoding mode, the encoder forces a sync sample at or near this boundary. Set to `.indefinite` for custom segmentation.
- **`AVAssetWriter.initialSegmentStartTime`** **[NEW]** — `CMTime` specifying the logical start time of the first segment; must be shifted by audio priming offset when using Apple HLS profile.
- **`AVAssetWriter.delegate`** **[NEW]** — Object conforming to `AVAssetWriterDelegate`; receives segment data as it is produced.
- **`AVAssetWriterDelegate`** **[NEW]** — Protocol with two optional methods delivering `Data` + `AVAssetSegmentType` (and optionally `AVAssetSegmentReport?`).
- **`AVAssetSegmentType`** **[NEW]** — `.initialization` (file type box + movie box) or `.separable` (one `moof` + one `mdat`); initialization segment is delivered first, then separable segments at each interval.
- **`AVAssetSegmentReport`** **[NEW]** — Metadata about a separable segment; used to construct HLS playlist `#EXTINF` durations and I-frame playlist entries.
- **`AVAssetWriter.flushSegment()`** **[NEW]** — Manually triggers segment output in custom segmentation mode (passthrough only); must be called before a sync sample.
- **`AVAssetTrack.hasAudioSampleDependencies`** **[NEW]** — `Bool`; indicates whether an audio track (e.g., USAC) has inter-sample dependencies; determines whether passthrough segmentation is safe.
- **Audio priming and A/V sync** — AAC encoding adds ~2,112 samples (~48 ms at 44,100 Hz) of silence (priming) before the first true audio sample. Apple HLS profile shifts `baseMediaDecodeTime` of audio backward by the priming amount; to keep it non-negative, shift both audio and video forward by a fixed time offset (e.g., 10 seconds, matching Apple's Media File Segmenter).
- **CMAF vs. Apple HLS profile** — CMAF uses an edit list box to compensate for priming; Apple HLS profile shifts `baseMediaDecodeTime` instead for backward compatibility with older players.
- **Passthrough vs. encoding mode** — Passthrough (`outputSettings: nil`): only one `AVAssetWriterInput` allowed; segments are cut at next sync sample after interval. Encoding mode: encoder generates forced sync samples at interval boundaries; multiple inputs (video + audio) allowed.

## APIs & Frameworks

### AVFoundation
- **`AVAssetWriter(contentType: UTType)`** **[NEW]** — `init(contentType:)` — no output URL; data delivered via delegate
- **`AVAssetWriter.outputFileTypeProfile`** **[NEW]** — `AVFileTypeProfile`; `.mpeg4AppleHLS`, `.mpeg4CMAF`
- **`AVAssetWriter.preferredOutputSegmentInterval`** **[NEW]** — `CMTime`; use `.indefinite` for custom segmentation
- **`AVAssetWriter.initialSegmentStartTime`** **[NEW]** — `CMTime`
- **`AVAssetWriter.delegate`** **[NEW]** — `(any AVAssetWriterDelegate)?`
- **`AVAssetWriter.flushSegment()`** **[NEW]** — Triggers custom segment output in passthrough mode
- **`AVAssetWriterDelegate`** **[NEW]** — Protocol:
  - `assetWriter(_:didOutputSegmentData:segmentType:)` — delivers `Data` + `AVAssetSegmentType`
  - `assetWriter(_:didOutputSegmentData:segmentType:segmentReport:)` — also delivers `AVAssetSegmentReport?`
- **`AVAssetSegmentType`** **[NEW]** — `enum`: `.initialization`, `.separable`
- **`AVAssetSegmentReport`** **[NEW]** — Info for playlist generation: track segment reports, duration, etc.
- **`AVAssetTrack.hasAudioSampleDependencies`** **[NEW]** — `var hasAudioSampleDependencies: Bool { get }`
- **`AVAssetWriterInput(mediaType:outputSettings:)`** — Existing; pass `nil` outputSettings for passthrough mode

## Code Highlights

Instantiate AVAssetWriter for fMP4 output (no file URL):
```swift
let assetWriter = AVAssetWriter(contentType: UTType(AVFileType.mp4.rawValue)!)
let videoInput = AVAssetWriterInput(mediaType: .video, outputSettings: compressionSettings)
assetWriter.add(videoInput)
```

Configure for Apple HLS fMP4 output:
```swift
assetWriter.outputFileTypeProfile = .mpeg4AppleHLS
assetWriter.preferredOutputSegmentInterval = CMTime(seconds: 6.0, preferredTimescale: 1)
assetWriter.initialSegmentStartTime = myInitialSegmentStartTime  // shifted for audio priming
assetWriter.delegate = myDelegateObject
```

Delegate: route initialization vs. separable segment data:
```swift
func assetWriter(_ writer: AVAssetWriter,
                 didOutputSegmentData segmentData: Data,
                 segmentType: AVAssetSegmentType,
                 segmentReport: AVAssetSegmentReport?) {
    switch segmentType {
    case .initialization:
        writeInitializationSegment(segmentData)
    case .separable:
        writeMediaSegment(segmentData)
        if let report = segmentReport { updatePlaylist(with: report) }
    }
}
```

Custom segmentation (passthrough, manual flush before sync sample):
```swift
assetWriter.outputFileTypeProfile = .mpeg4AppleHLS
assetWriter.preferredOutputSegmentInterval = .indefinite
// ... append samples ...
assetWriter.flushSegment()  // call before next sync sample
```

## Takeaways
- The new `AVAssetWriter(contentType:)` initializer + delegate API replaces file-based output with a streaming push model; no temporary file needed to produce fMP4 segments.
- Set `outputFileTypeProfile` to `.mpeg4AppleHLS` or `.mpeg4CMAF` to enable fMP4 output; in encoding mode, the encoder automatically inserts sync samples at `preferredOutputSegmentInterval` boundaries.
- The delegate receives two segment types: one initialization segment (sent first) and then separable segments at each interval — package these into HLS `.m3u8` playlist entries manually using `AVAssetSegmentReport`.
- Shift `initialSegmentStartTime` and all sample timestamps forward (e.g., by 10 s) when using Apple HLS profile to ensure `baseMediaDecodeTime` of audio remains non-negative after priming compensation, preserving exact A/V sync.
- Use `AVAssetTrack.hasAudioSampleDependencies` to determine whether passthrough mode is viable for an audio track before attempting to create a single `AVAssetWriterInput`.

---
_Source: WWDC20 Session 10011 page (abstract, transcript, and code samples)._
