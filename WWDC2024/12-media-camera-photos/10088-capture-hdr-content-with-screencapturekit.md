# Capture HDR Content with ScreenCaptureKit
**WWDC24 · Session 10088** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10088/)

_Platforms:_ macOS 15

## Overview
This session dives into the new HDR capture capabilities added to ScreenCaptureKit in macOS 15. Apple's display pipeline now supports HDR content end to end, and ScreenCaptureKit exposes that pipeline to developers so they can capture high-dynamic-range content from displays, windows, and applications — including content rendered at up to 16 bits per component.

The talk walks through what HDR means for screen capture, which pixel formats carry HDR data, and how to configure a stream to opt into HDR output. It also covers the interplay with tone mapping so that apps can control whether captured content is delivered in a tonemapped form suitable for SDR display or in extended-range form for downstream HDR workflows.

## Key Topics
- **What is HDR in the context of screen capture?** — macOS renders EDR (Extended Dynamic Range) content on compatible displays; ScreenCaptureKit can now expose that raw luminance data to capture clients.
- **Pixel format selection** — choosing between `SCStreamConfiguration`'s SDR and HDR-capable formats; the HDR path requires a 10-bit or floating-point pixel format.
- **Tone mapping control** — `SCStreamConfiguration.captureDynamicRange` lets you choose between `.standard` (SDR tone-mapped output), `.HDRLocalDisplay`, and `.HDRCanonicalDisplay` to match the intended consumption.
- **Color space and metadata** — captured `CMSampleBuffer` frames carry the correct `CVImageBuffer` color attachments and HDR metadata so downstream consumers can interpret luminance correctly.
- **Practical integration** — updating an existing `SCStream`-based capture pipeline to support HDR with minimal code changes.

## APIs & Frameworks

**ScreenCaptureKit**
- `SCStream` — core streaming object; unchanged in surface API but now routes HDR pixel data
- `SCStreamConfiguration` — main configuration type
  - **[NEW]** `captureDynamicRange: SCCaptureDynamicRange` — controls whether the stream delivers SDR or HDR frames
- **[NEW]** `SCCaptureDynamicRange` enum — `.standard`, `.HDRLocalDisplay`, `.HDRCanonicalDisplay`
- `SCStreamOutput` — delegate protocol; `stream(_:didOutputSampleBuffer:of:)` now may deliver HDR `CMSampleBuffer`
- `SCContentFilter` — unchanged; used to specify capture targets
- `SCShareableContent` — unchanged; enumerates capturable content

**Core Media / Core Video**
- `CMSampleBuffer` — frame delivery type; HDR frames carry extended color attachments
- `CVImageBuffer` / `CVPixelBuffer` — underlying pixel storage; HDR output uses `kCVPixelFormatType_420YpCbCr10BiPlanarVideoRange` or float formats
- `CVColorPrimaries`, `CVTransferFunction`, `CVYCbCrMatrix` — color space attachment keys present on HDR sample buffers

## Code Highlights
Configure HDR capture by setting `captureDynamicRange` before starting the stream:

```swift
let config = SCStreamConfiguration()
config.width = 3456
config.height = 2234
config.pixelFormat = kCVPixelFormatType_420YpCbCr10BiPlanarVideoRange
config.captureDynamicRange = .HDRLocalDisplay

let stream = SCStream(filter: filter, configuration: config, delegate: nil)
try stream.addStreamOutput(self, type: .screen, sampleHandlerQueue: .main)
try await stream.startCapture()
```

Downstream, inspect the HDR color attachments on the delivered `CMSampleBuffer` to confirm extended range metadata is present before passing frames to a video encoder or compositor.

## Takeaways
- Set `SCStreamConfiguration.captureDynamicRange` to `.HDRLocalDisplay` or `.HDRCanonicalDisplay` to receive HDR pixel data; the default `.standard` continues to deliver SDR-tone-mapped frames.
- Pair HDR capture with a 10-bit or floating-point `pixelFormat` — an SDR pixel format ignores the HDR dynamic range setting.
- Inspect `CVImageBuffer` color attachments on delivered sample buffers to confirm HDR metadata before feeding frames into a video pipeline.
- Test on a Pro Display XDR or similar EDR-capable display; on SDR-only hardware the HDR modes fall back gracefully.

---
_Source: WWDC24 Session 10088 page (abstract, chapter summaries, code samples, and resource links)._
