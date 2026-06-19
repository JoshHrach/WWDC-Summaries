# Metal for Pro Apps
**WWDC19 · Session 608** · [Watch](https://developer.apple.com/videos/play/wwdc2019/608/)

_Platforms:_ macOS Catalina 10.15, iPadOS 13

## Overview
This session targets creative professional application developers building video editors, 3D animation tools, photo compositors, and audio production software. It covers four interconnected topics: building an efficient 8K video editing pipeline using Video Toolbox, IOSurface, and Metal Performance Shaders; enabling HDR rendering with EDR (Extended Dynamic Range) via `CAMetalLayer`; scaling CPU and GPU parallelism using multi-threaded command buffer encoding and parallel render encoders; and leveraging multi-GPU configurations — including the Mac Pro's new Infinity Fabric Link — with the `MTLDevice` peer group API.

Real-world benchmarks from DaVinci Resolve (real-time 8K raw playback), Affinity Photo (4K compositing on four external GPUs at 60+ fps), and Final Cut Pro (multiple 8K ProRes streams using Infinity Fabric Link) anchor each concept in production-scale results.

A key memory management insight: 8K uncompressed frames reach 270 MB at 9 GB/s for 30 fps, approaching PCIe bandwidth limits. Pre-warming CPU buffers, using `CVPixelBufferPool`, `CVMetalTextureCache`, and `MTLHeap` aliasable allocations dramatically reduces page-fault overhead and enables sustained real-time throughput.

## Key Topics
- **8K video editing pipeline** — `VTDecompressionSession` (async, hardware decode) → `CVMetalTextureCache` (zero-copy IOSurface-backed textures) → MPS pixel processing → `VTCompressionSession` (hardware encode on same GPU device); `CVDisplayLink` for predictable frame rate
- **Memory management for large assets** — pre-warm CPU buffers; `CVPixelBufferPool` and `CVMetalTextureCache` for buffer reuse; `MTLHeap` with aliasable resources for transient allocations; avoid mid-workflow allocations
- **EDR / HDR rendering** — Extended Dynamic Range model scales HDR pixel values relative to SDR brightness headroom; `CAMetalLayer` with `wantsExtendedDynamicRangeContent`, wide-gamut color space (BT.2020 or P3), float16 pixel format, PQ/HLG/Gamma transfer functions; `NSScreen.maximumExtendedDynamicRangeColorComponentValue` for dynamic headroom
- **CPU parallelism** — multiple `MTLCommandBuffer` objects encoded on separate threads; `enqueue()` to set execution order without blocking; `MTLParallelRenderCommandEncoder` for parallel encoding within a single render pass
- **GPU channel parallelism** — blit, compute, and render encoders run on separate asynchronous GPU channels; replace `waitUntilCompleted` with completion handlers; decode 10 frames ahead to remove data dependencies; preload bitmaps to eliminate blit-to-compute stalls
- **Multi-GPU** — `MTLDevice.peerGroupID`, `peerIndex`, `peerCount`; `MTLDevice.location`, `locationNumber`, `maxTransferRate`; load balancing via alternating frames, tile queues, or random tile assignment; `MTLSharedEvent` for cross-GPU synchronization
- **Infinity Fabric Link / peer group transfer API** — `MTLTexture` remote views; up to 5x PCIe bandwidth; operates on its own parallel GPU channel **[NEW macOS Catalina]**

## APIs & Frameworks
- **Metal**
  - `MTLDevice` — `peerGroupID`, `peerIndex`, `peerCount`, `location`, `locationNumber`, `maxTransferRate` **[NEW for multi-GPU]**
  - `MTLCommandBuffer.enqueue()` — pre-register execution order **[NEW usage pattern]**
  - `MTLParallelRenderCommandEncoder` — parallel render encoding on multiple CPU threads
  - `MTLHeap` — batch allocate GPU memory; `MTLResource.makeAliasable()` for transient reuse
  - `MTLSharedEvent` — cross-GPU, cross-process, CPU-GPU synchronization; `encodeSignalEvent(_:value:)` / `encodeWaitForEvent(_:value:)`
  - `MTLTexture` remote views — cross-GPU texture access via Infinity Fabric Link peer group API **[NEW]**
  - `CAMetalLayer.wantsExtendedDynamicRangeContent: Bool` — opt in to EDR rendering
  - `CAMetalLayer.colorspace` — wide-gamut (BT.2020, P3)
  - `CAMetalLayer.pixelFormat` — `.rgba16Float` recommended for HDR
  - `CAEDRMetadata` — attach mastering display / tone mapping metadata to `CAMetalLayer` **[NEW macOS Catalina]**
- **Video Toolbox**
  - `VTDecompressionSession` — hardware video decode; `kVTDecodeFrameFlags_EnableAsynchronousDecompression`; device-specific (`kVTVideoDecoderSpecification_RequireHardwareAcceleratedVideoDecoder`)
  - `VTCompressionSession` — hardware video encode; specify `MTLDevice` to minimize copies; `CVPixelBufferPool` for encoder-native buffer format
  - Supported codecs: H.264, HEVC, ProRes, ProRes RAW, and more
- **Core Video**
  - `CVMetalTextureCache` — zero-copy Metal texture from `CVPixelBuffer` backed by IOSurface
  - `CVPixelBufferPool` — reuse pixel buffers for encoder compatibility
  - `CVDisplayLink` — high-precision VBLANK timer for frame pacing; `CVTimeStamp` for current time and next-VBLANK scheduling
  - `IOSurface` — shared, GPU-resident, interprocess image buffer
- **Metal Performance Shaders**
  - `MPSImageGaussianBlur` and other MPS kernels for in-pipeline pixel processing
  - In-place encoding with fallback allocator
- **AppKit / Core Animation**
  - `NSScreen.maximumExtendedDynamicRangeColorComponentValue` — dynamic EDR headroom (changes with ambient conditions/brightness)
  - `NSScreen.NSScreenColorSpaceDidChange` notification — redraw when EDR headroom changes

## Code Highlights

```swift
// Zero-copy video decode → Metal texture
let textureCache = CVMetalTextureCache.create(with: metalDevice)

// On decode completion:
var cvTexture: CVMetalTexture?
CVMetalTextureCacheCreateTextureFromImage(
    kCFAllocatorDefault, textureCache, pixelBuffer,
    nil, .bgra8Unorm, width, height, 0, &cvTexture)
let mtlTexture = CVMetalTextureGetTexture(cvTexture!)
```

```swift
// Multi-threaded command buffer encoding
let cb1 = commandQueue.makeCommandBuffer()!
let cb2 = commandQueue.makeCommandBuffer()!
cb1.enqueue()  // reserve execution slot before encoding
cb2.enqueue()

DispatchQueue.global().async { encodePass1(into: cb1); cb1.commit() }
DispatchQueue.global().async { encodePass2(into: cb2); cb2.commit() }
```

```swift
// MTLHeap with aliasable transient resources
let heap = device.makeHeap(descriptor: heapDesc)!
let blurUniforms = heap.makeBuffer(length: 256, options: .storageModePrivate)!
let gradeUniforms = heap.makeBuffer(length: 256, options: .storageModePrivate)!
// After use:
gradeUniforms.makeAliasable()  // heap can reuse this memory
let intermediateBuffer = heap.makeBuffer(length: frameSize, options: .storageModePrivate)!
```

```swift
// Cross-GPU sync with MTLSharedEvent + remote texture view (Infinity Fabric Link)
let event = auxDevice.makeSharedEvent()!
// On auxiliary GPU:
encodeRender(on: auxCommandBuffer, texture: auxTexture)
auxCommandBuffer.encodeSignalEvent(event, value: 1)
// On display GPU:
displayCommandBuffer.encodeWaitForEvent(event, value: 1)
let remoteView = auxTexture.makeRemoteTextureView(for: displayDevice)!
blitEncoder.copy(from: remoteView, ..., to: displayTexture, ...)
```

## Takeaways
- Build the video pipeline end-to-end on the same `MTLDevice`: use `CVMetalTextureCache` for zero-copy decode, MPS for processing, and specify the same device to `VTCompressionSession` for encoding — eliminating every unnecessary GPU-CPU copy.
- `CVDisplayLink` paired with `presentDrawable(afterMinimumDuration:)` is the only reliable way to achieve jitter-free video at mismatched frame/display rates.
- Use float16 pixel formats and `CAEDRMetadata` for HDR; query `NSScreen.maximumExtendedDynamicRangeColorComponentValue` dynamically and redraw on change notifications.
- The new peer group API with Infinity Fabric Link achieves up to 5x PCIe bandwidth for GPU-to-GPU copies on Mac Pro, enabling Final Cut Pro's real-time multi-stream 8K workflows.

---
_Source: WWDC19 Session 608 page (abstract, chapter summaries, code samples, and resource links)._
