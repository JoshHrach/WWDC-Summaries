# Decode ProRes with AVFoundation and VideoToolbox
**WWDC20 · Session 10090** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10090/)

_Platforms:_ macOS Big Sur 11

## Overview
This session teaches developers how to build an efficient ProRes decoding pipeline for Mac apps with custom Metal rendering engines. It covers the full stack from compressed file data to Metal textures: using `AVAssetReader` for the highest-level approach, `AVSampleBufferGenerator` or manual `CMSampleBuffer` construction for more control, and `VTDecompressionSession` for direct Video Toolbox access. The session also covers optimal pixel format selection to avoid unnecessary buffer copies, hardware decoder control (including Afterburner), and two approaches for bridging decoded `CVPixelBuffer` frames into Metal textures safely.

Video decoders run in a sandboxed out-of-process server — providing both security isolation and application stability (a decoder crash yields a decode error rather than an app crash). Hardware decoding is enabled by default on current OS versions and requires no opt-in.

## Key Topics

**Media Stack Layers**
From high to low level: AVKit (too high-level for custom pipelines) → AVFoundation (`AVAssetReader`, `AVSampleBufferGenerator`) → VideoToolbox (`VTDecompressionSession`) → CoreMedia / CoreVideo primitives. This session focuses on the AVFoundation, VideoToolbox, and CoreVideo layers.

**Key Media Types**
- `CVPixelBuffer` — uncompressed raster image data; backed by `IOSurface`.
- `CMBlockBuffer` — wraps arbitrary compressed sample data.
- `CMSampleBuffer` — wraps either a `CMBlockBuffer` (compressed) or `CVPixelBuffer` (uncompressed); carries presentation/decode timestamps and a `CMFormatDescription`.
- `IOSurface` — memory abstraction enabling efficient transfer between frameworks, processes, and GPUs.
- `CVPixelBufferPool` — recycles `IOSurface`-backed buffers; allocated buffers return to the pool on release.

**AVAssetReader — Simplest Full Pipeline**
`AVAssetReader` with an `AVAssetReaderTrackOutput` reads from file, handles edits and frame dependencies, optimizes samples for RPC transfer to the sandboxed decoder, and returns decoded `CVPixelBuffer` frames in the requested pixel format. Set `alwaysCopiesSampleData = NO` for maximum efficiency. Requesting a native decoder output format avoids format conversion copies.

**Optimal Pixel Formats (avoid conversion copies)**
- ProRes 4444: `kCVPixelFormatType_4444AYpCbCr16` (Y416)
- ProRes 422: `kCVPixelFormatType_422YpCbCr16` (v216)
- ProRes RAW: `kCVPixelFormatType_64RGBAHalf` (RGhA)

**Generating CMSampleBuffers for VTDecompressionSession**
Three options: `AVAssetReader` with `outputSettings = nil` (compressed output, RPC-optimized); `AVSampleBufferGenerator` with `AVSampleCursor` (no edit/dependency awareness); manual construction from `CMBlockBuffer` + `CMVideoFormatDescription` + `CMSampleTimingInfo` (not RPC-optimized).

**VTDecompressionSession**
Direct control over the hardware decoder. Key creation parameters: `videoFormatDescription` (must match incoming samples), `destinationImageBufferAttributes` (output pixel format, optional scaling), and `videoDecoderSpecification` (hardware/software preference). Use `kVTDecodeFrame_EnableAsynchronousDecompression` for best performance. Do not block inside the output handler/callback — it serializes frame delivery.

**Hardware Decoder Control**
- Default: hardware decode automatically used when available (including Afterburner); no opt-in needed in current OS versions.
- Force hardware: `kVTVideoDecoderSpecification_RequireHardwareAcceleratedVideoDecoder = true` (fails session creation if unavailable).
- Force software: `kVTVideoDecoderSpecification_EnableHardwareAcceleratedVideoDecoder = false`.
- Pro video codecs: call `VTRegisterProfessionalVideoWorkflowVideoDecoders()` once at app launch.

**CVPixelBuffer → Metal Texture (Two Approaches)**
1. **IOSurface direct**: get `IOSurface` from `CVPixelBuffer`, create `MTLTexture` from it, call `IOSurfaceIncrementUseCount` to prevent pool recycling, decrement in `MTLCommandBuffer` completion handler.
2. **CVMetalTextureCache** (preferred): `CVMetalTextureCacheCreate`, then `CVMetalTextureCacheCreateTextureFromImage` → `CVMetalTextureGetTexture`. Automatically handles IOSurface reuse detection. Release `CVMetalTexture` in `MTLCommandBuffer` completion handler.

## APIs & Frameworks

### AVFoundation
- `AVAsset` — represents a media asset
- `AVAssetReader` — reads and decodes samples from an `AVAsset`
- `AVAssetReaderTrackOutput` — configures decoded or compressed output from a track
- `AVAssetReaderTrackOutput.alwaysCopiesSampleData` — set to `NO`/`false` for zero-copy efficiency
- `AVAssetReaderTrackOutput.outputSettings` — pixel format dict or `nil` for compressed output
- `AVAssetReader.startReading()` — begins reading
- `AVAssetReaderOutput.copyNextSampleBuffer()` — returns next `CMSampleBuffer`
- `AVSampleBufferGenerator` — generates `CMSampleBuffer` from an `AVAssetTrack`
- `AVSampleCursor` — cursor for traversing samples in an `AVAssetTrack`
- `AVSampleBufferRequest` — describes a sample buffer request

### VideoToolbox
- `VTRegisterProfessionalVideoWorkflowVideoDecoders()` — registers pro video decoders (call once at launch)
- `VTDecompressionSessionCreate` — creates a hardware/software decode session
- `VTDecompressionSessionDecodeFrameWithOutputHandler` — block-based async frame decode
- `VTDecompressionSessionDecodeFrame` — callback-based frame decode
- `kVTDecodeFrame_EnableAsynchronousDecompression` — flag for async decode (recommended)
- `kVTVideoDecoderSpecification_RequireHardwareAcceleratedVideoDecoder` — require HW decoder
- `kVTVideoDecoderSpecification_EnableHardwareAcceleratedVideoDecoder` — enable/disable HW decoder
- `VTDecompressionOutputHandler` — block type receiving decoded `CVImageBufferRef`

### CoreMedia
- `CMSampleBuffer` / `CMSampleBufferRef` — media sample container
- `CMBlockBuffer` / `CMBlockBufferRef` — compressed data container
- `CMVideoFormatDescription` / `CMVideoFormatDescriptionRef` — describes video format
- `CMSampleTimingInfo` — presentation and decode timestamps
- `CMSampleBufferGetFormatDescription` — retrieves format description from sample
- `CMSampleBufferGetImageBuffer` — retrieves `CVImageBuffer` from decoded sample
- `CMBlockBufferCreateWithMemoryBlock` — creates block buffer from raw memory
- `CMVideoFormatDescriptionCreate` — creates video format description
- `CMSampleBufferCreateReady` — creates a complete sample buffer

### CoreVideo
- `CVPixelBuffer` / `CVPixelBufferRef` — uncompressed frame buffer
- `CVPixelBufferPool` / `CVPixelBufferPoolRef` — recyclable buffer pool
- `CVPixelBufferGetIOSurface` — retrieves IOSurface backing
- `CVMetalTextureCache` / `CVMetalTextureCacheRef` — Metal texture cache for CVPixelBuffers
- `CVMetalTextureCacheCreate` — creates a Metal texture cache
- `CVMetalTextureCacheCreateTextureFromImage` — wraps a CVPixelBuffer as a Metal texture
- `CVMetalTextureGetTexture` — retrieves `MTLTexture` from `CVMetalTexture`
- `kCVPixelFormatType_4444AYpCbCr16` — Y416 (ProRes 4444 native)
- `kCVPixelFormatType_422YpCbCr16` — v216 (ProRes 422 native)
- `kCVPixelFormatType_64RGBAHalf` — RGhA (ProRes RAW native)

### IOSurface / Metal
- `IOSurfaceIncrementUseCount` — prevents pool recycling while Metal uses the surface
- `IOSurfaceDecrementUseCount` — releases the surface back to the pool
- `MTLDevice.newTexture(descriptor:iosurface:plane:)` — creates Metal texture from IOSurface
- `MTLCommandBuffer.addCompletedHandler` — used to release CVMetalTexture or decrement IOSurface use count after GPU completion

## Code Highlights

AVAssetReader with native ProRes 4444 output (no conversion):
```objc
NSDictionary *outputSettings = @{ (id)kCVPixelBufferPixelFormatTypeKey :
                                  @(kCVPixelFormatType_4444AYpCbCr16) };
AVAssetReaderTrackOutput *output = [AVAssetReaderTrackOutput
    assetReaderTrackOutputWithTrack:track outputSettings:outputSettings];
output.alwaysCopiesSampleData = NO;
```

VTDecompressionSession creation with hardware decode:
```objc
CMFormatDescriptionRef formatDesc = CMSampleBufferGetFormatDescription(sampleBuffer);
CFDictionaryRef pixelBufferAttributes = (__bridge CFDictionaryRef)@{
    (id)kCVPixelBufferPixelFormatTypeKey : @(kCVPixelFormatType_4444AYpCbCr16) };
VTDecompressionSessionRef session;
VTDecompressionSessionCreate(kCFAllocatorDefault, formatDesc, NULL,
                              pixelBufferAttributes, NULL, &session);
```

CVMetalTextureCache approach for Metal integration:
```objc
CVMetalTextureCacheRef cache;
CVMetalTextureCacheCreate(kCFAllocatorDefault, NULL, metalDevice, NULL, &cache);
CVMetalTextureCacheCreateTextureFromImage(kCFAllocatorDefault, cache, pixelBuffer,
    NULL, pixelFormat, width, height, 0, &cvTexture);
id<MTLTexture> texture = CVMetalTextureGetTexture(cvTexture);
// Release cvTexture in MTLCommandBuffer completion handler
```

## Takeaways
- Use `AVAssetReader` with native pixel formats (Y416 for ProRes 4444, v216 for ProRes 422) and `alwaysCopiesSampleData = NO` for the most efficient ProRes decoding pipeline — it handles everything including Afterburner automatically.
- Hardware decoding (including Afterburner) is enabled by default in current OS versions; no opt-in is required, but `VTRegisterProfessionalVideoWorkflowVideoDecoders()` must be called once for pro codec access.
- Use `CVMetalTextureCache` rather than direct `IOSurface` bridging for simpler and safer Metal texture integration — it handles IOSurface reuse detection automatically.
- Never block inside a `VTDecompressionSession` output handler; doing so serializes all subsequent frame delivery and creates back pressure through the decoder.

---
_Source: WWDC20 Session 10090 page (abstract, chapter summaries, code samples, and resource links)._
