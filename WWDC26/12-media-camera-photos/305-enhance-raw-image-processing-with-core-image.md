# Enhance RAW image processing with Core Image
**WWDC26 · Session 305** · [Watch](https://developer.apple.com/videos/play/wwdc2026/305/)

_Platforms:_ iOS, iPadOS, macOS

## Overview
Core Image's RAW processing pipeline has evolved through nine versions and now supports 784 camera models. RAW 9 is the headline update for WWDC26, introducing a Core ML-powered demosaic and denoise stage that runs on the Apple Neural Engine to deliver significantly sharper images with more accurate colors and better noise reduction compared to RAW 8.

The session covers how to opt in to RAW 9 via the `CIRAWFilter` API, the best practices for interactive editing (repeated renders at screen resolution) versus batch export (multiple files at full resolution), and two new `CIImageProcessor` capabilities that give developers precise control over tile sizing and temporary buffer allocation.

The performance story is deliberately split: interactive editing should enable `cacheIntermediates` and use Metal-backed `MTKView`s with the `scaleFactor` property to avoid rendering at full sensor resolution on every frame; export workflows should disable caching, tune `memoryLimit`, and call `heifRepresentation` / `jpegRepresentation` in sequence.

## Key Topics

### How Core Image Supports RAW
RAW processing proceeds through demosaic, denoising, convolution, and color adjustment stages. All stages are surfaced through the `CIRAWFilter` API and are supported across macOS and iOS/iPadOS.

### The Evolution of RAW Support
Nine versions of the pipeline from version 1 through RAW 9. Each version has expanded camera model support; RAW 9 now covers 784 models.

### RAW 9 Overview and Quality Improvements
RAW 9 uses a Core ML model for both demosaic and denoise, replacing the previous algorithmic stages. Side-by-side comparisons demonstrate clear improvements in edge sharpness, color fidelity, and noise reduction at high ISOs.

### Enable and Edit RAW 9 with CIRAWFilter API
Opt in by setting `decoderVersion` on a `CIRAWFilter` instance. Check `supportedDecoderVersions` to confirm whether a given camera model supports version 9. The full editing parameter set (exposure, noise reduction, sharpness, contrast, and more) remains available via `CIRAWFilter` properties.

### RAW 9 Performance
RAW 9 runs on the Apple Neural Engine, so it is substantially more efficient than a pure GPU path for the demosaic/denoise stages. The session provides concrete profiling guidance and separates the interactive vs. export performance strategies.

### Interactive Editing
Use `scaleFactor` on the filter to render a downscaled output matching the display, enable `cacheIntermediates` so intermediate results are reused across repeated parameter changes, use the Extended Virtual Addressing entitlement to handle large memory footprints, and render into Metal-backed `MTKView`s.

### Exporting to Other Formats
Create a dedicated `CIContext` with `cacheIntermediate: false` and an appropriate `memoryLimit`. Call `heifRepresentation` or `jpegRepresentation` serially for batch jobs.

### New CIImageProcessor Features
Two new capabilities: (1) explicit output tile sizes via `apply(withTiledExtent:inputs:arguments:)`, and (2) temporary pixel buffer allocation inside the processor via `output.temporaryPixelBuffer(identifier:format:width:height:pixelBufferAttributes:)` — Core Image caches and reuses the scratch buffers across calls.

## APIs & Frameworks

### Core Image
- `CIRAWFilter` — primary class for RAW decoding and editing
  - `decoderVersion` property — **[NEW]** opt in to RAW 9
  - `supportedDecoderVersions` — query versions available for a given camera model
  - Editing properties: `exposure`, `noiseReduction`, `sharpness`, `contrast`, `localToneMapAmount`, etc.
  - `scaleFactor` property — render at reduced resolution for interactive preview
  - `cacheIntermediates` — enable for interactive editing, disable for export
- `CIContext`
  - `.cacheIntermediate` option key — set to `false` for export contexts
  - `.memoryLimit` option key — tune RAM budget for export (e.g., `512` MB)
  - `heifRepresentation(of:format:colorSpace:options:)` — export to HEIF
  - `jpegRepresentation(of:colorSpace:options:)` — export to JPEG
- `CIImageProcessorKernel` — base class for custom processors
  - `apply(withTiledExtent:inputs:arguments:)` — **[NEW]** explicit tile-based rendering
  - `output.temporaryPixelBuffer(identifier:format:width:height:pixelBufferAttributes:)` — **[NEW]** cached scratch buffer allocation
  - `roi(forInput:arguments:outputRect:)` — region-of-interest callback
  - `process(with:arguments:output:)` — processing callback
- `CIImageProcessorInput` — `.pixelBuffer`, `.region`
- `CIImageProcessorOutput` — `.pixelBuffer`, `.region`

### Metal / MetalKit
- `MTKView` — recommended render target for interactive RAW editing

### Entitlements
- `com.apple.developer.kernel.extended-virtual-addressing` — required to handle the large intermediate buffers produced by RAW 9 during interactive editing

## Code Highlights

Export context with memory tuning:
```swift
let exportCtx = CIContext(options: [
    .cacheIntermediate: false,
    .memoryLimit: 512
])
```

Explicit tile-size processor invocation:
```swift
let tiles: [CIVector] = /* build CGRect tiles */
let result = try MyProcessor.apply(withTiledExtent: tiles, inputs: [inImg], arguments: [:])
```

Temporary scratch buffer inside a processor:
```swift
guard let scratch = output.temporaryPixelBuffer(
    identifier: "myScratch",
    format: kCVPixelFormatType_64RGBAHalf,
    width: Int(output.region.width),
    height: Int(output.region.height),
    pixelBufferAttributes: nil) else { return }
```

## Takeaways
- RAW 9 uses a Core ML model on the Apple Neural Engine for demosaic and denoise — opt in via `CIRAWFilter.decoderVersion` and check `supportedDecoderVersions` first.
- Interactive editing and batch export require distinct `CIContext` configurations; mixing strategies will hurt either responsiveness or memory usage.
- The two new `CIImageProcessor` APIs (explicit tile sizes and temporary buffers) enable fine-grained performance tuning for custom pipeline stages that process large images.
- The Extended Virtual Addressing entitlement is necessary to sustain RAW 9's intermediate memory footprint during interactive editing sessions.

---
_Source: WWDC26 Session 305 page (abstract, chapter summaries, code samples, and resource links)._
