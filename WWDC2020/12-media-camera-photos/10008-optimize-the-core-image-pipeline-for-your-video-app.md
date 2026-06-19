# Optimize the Core Image pipeline for your video app
**WWDC20 · Session 10008** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10008/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, tvOS 14

## Overview
This session explains how to get the best performance when using Core Image to apply real-time effects to video. It covers three key areas: creating `CIContext` efficiently, writing custom CIKernels in Metal Shading Language, and choosing the right view class (`AVPlayerView` vs `MTKView`) for video playback with Core Image filters.

Core Image is built on Metal internally, so understanding how to share Metal command queues between Core Image and other Metal rendering work is essential for avoiding pipeline bubbles and wasted GPU cycles. The session also emphasizes writing custom kernels in `.ci.metal` files to shift compile costs to build time and unlock high-performance Metal features.

## Key Topics

### Creating CIContext Efficiently
- Create only **one** `CIContext` per view — contexts are expensive to initialize
- Set `.cacheIntermediates: false` when processing video, since every frame differs; disabling the cache lowers memory usage significantly
- Give the context a `.name` for easier Core Image debugging
- When combining Core Image with other Metal rendering (e.g., using Metal textures as CI input/output), create the `CIContext` with the **same** `MTLCommandQueue` used by the rest of the app — this eliminates CPU wait commands and allows efficient GPU pipeline overlap

### Writing Custom CIKernels in Metal
- Prefer built-in Core Image filters (`CoreImage.CIFilterBuiltins`) since they are all implemented in Metal and fully optimized
- Write custom kernels in `.ci.metal` source files (include `<CoreImage/CoreImage.h>`) — benefits:
  - Eliminates runtime compilation (shifts to app build time)
  - Access to gather-reads, group-writes, and half-float math
  - Syntax highlighting and build-time error checking in Xcode
- `CIColorKernel` functions must be `extern "C"`, return `float4`, and accept `coreimage::sample_t` and `coreimage::destination` parameters
- `coreimage::sample_t` is a linear premultiplied RGBA `float4` suitable for both SDR and HDR images
- `dest.coord()` provides the pixel coordinate for position-dependent effects

### Rendering to Views
- Avoid `UIImageView` / `NSImageView` for video — designed for static content
- **`AVPlayerView`** (simplest): use `AVMutableVideoComposition(asset:applyingCIFiltersWithHandler:)` — the handler receives `AVAsynchronousCIImageFilteringRequest` per frame; apply filter and call `request.finish(with:context:)`
- **`MTKView`** (most flexible and performant):
  - Set `framebufferOnly = false` to allow Core Image to use Metal Compute
  - For HDR on macOS: set `colorPixelFormat = .rgba16Float` and `wantsExtendedDynamicRangeContent = true` on the `CAMetalLayer`
  - Create `CIRenderDestination` with a **closure** returning the drawable texture (not the texture directly) — allows CI to start enqueueing Metal work before the previous frame completes, improving pipelining
  - Call `context.startTask(toRender:from:to:at:)` then present the drawable in a separate command buffer

### HDR Video
- `AVAsynchronousCIImageFilteringRequest.sourceImage` for a 10-bit HDR source is automatically color-managed from HLG to the Core Image working space
- Xcode debugger: hover over a `CIImage` variable and click the eye icon to see a visual recipe of the filter graph

## APIs & Frameworks

- **Core Image** framework
- `CIContext` — `init(options:)`, `init(MTLCommandQueue:options:)`, `init(mtlDevice:options:)` **[KEY: share queue]**
  - Option `.cacheIntermediates` **[KEY for video]**
  - Option `.name`
  - `startTask(toRender:from:to:at:)` **[NEW]**
- `CIImage` — input/output image type
- `CIFilter` — base class
- `CIColorKernel` — `extern "C"` Metal function returning `float4`
- `CIRenderDestination` — `init(width:height:pixelFormat:commandBuffer:mtlTextureProvider:)` **[NEW closure-based init]**
- `CoreImage.CIFilterBuiltins` module — typed filter accessors (e.g., `CIFilter.motionBlur()`)
- `coreimage::sample_t` — Metal type for CI kernel pixel input
- `coreimage::destination` — Metal type providing `coord()` in CI kernels
- `AVMutableVideoComposition` — `init(asset:applyingCIFiltersWithHandler:)`
- `AVAsynchronousCIImageFilteringRequest` — `sourceImage`, `finish(with:context:)`, `finish(with:)` (error)
- **MetalKit** — `MTKView`, `MTKViewDelegate.draw(in:)`
- `MTLCommandQueue` — shared queue for CI + custom Metal work
- `MTLCommandBuffer` — `present(_:)`, `commit()`
- `CAMetalLayer` — `wantsExtendedDynamicRangeContent` (macOS HDR)
- `MTLPixelFormat.rgba16Float` — HDR pixel format
- Metal Shading Language: `<CoreImage/CoreImage.h>`, `CIKernelMetalLib.h`

## Code Highlights

Create a `CIContext` that shares a Metal command queue:
```swift
let context = CIContext(MTLCommandQueue: queue, options: [.cacheIntermediates: false, .name: "MyAppView"])
```

Metal kernel for HDR zebra-stripe detection (`.ci.metal`):
```metal
#include <CoreImage/CoreImage.h>
using namespace metal;
extern "C" float4 HDRZebra(coreimage::sample_t s, float time, coreimage::destination dest) {
    float diagLine = dest.coord().x + dest.coord().y;
    float zebra = fract(diagLine / 20.0 + time * 2.0);
    if ((zebra > 0.5) && (s.r > 1 || s.g > 1 || s.b > 1))
        return float4(2.0, 0.0, 0.0, 1.0);
    return s;
}
```

`MTKView` draw method with deferred texture closure:
```swift
let rd = CIRenderDestination(width: Int(size.width), height: Int(size.height),
                              pixelFormat: colorPixelFormat, commandBuffer: nil)
          { () -> MTLTexture in return view.currentDrawable!.texture }
context.startTask(toRender: image, from: rect, to: rd, at: point)
```

## Takeaways

- Create a single `CIContext` per view with `.cacheIntermediates: false` and share it with the app's `MTLCommandQueue` to eliminate GPU pipeline bubbles.
- Write custom kernels in `.ci.metal` files — doing so shifts runtime compile costs to build time and unlocks half-float math and gather-reads for better GPU utilization.
- Use `AVPlayerView` for the simplest video+CI integration; use `MTKView` with the closure-based `CIRenderDestination` for maximum control and performance.
- For HDR video on macOS, set `colorPixelFormat = .rgba16Float` and `wantsExtendedDynamicRangeContent = true` on the Metal layer.

---
_Source: WWDC20 Session 10008 page (abstract, chapter summaries, code samples, and resource links)._
