# Delivering Optimized Metal Apps and Games
**WWDC19 · Session 606** · [Watch](https://developer.apple.com/videos/play/wwdc2019/606/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15, tvOS 13

## Overview
This session presents 18 concrete Metal optimization best practices organized into three areas: general GPU performance, memory bandwidth reduction, and memory footprint reduction. The session uses Afterpulse (by Digital Legends) as a real-world case study, demonstrating how each practice applies in a production deferred-rendering game and quantifying the savings achieved.

Two new tools debut: the Metal Memory Viewer (integrated into the Metal Frame Debugger) shows all live Metal resources grouped by type, storage mode, and usage with actionable issue annotations; and the Metal Resource Allocations instrument track shows per-allocation events over time. A new C-based API allows querying available device memory at runtime, enabling adaptive streaming and memory spike avoidance. On-device GPU capture via a new programmatic API makes QA testing without Xcode feasible.

## Key Topics

### General Performance (Best Practices 1–5)
- **Choose resolution per effect** — Shadow maps and SSAO can use lower resolution; UI should composite at native. Use the Dependency Viewer to audit each pass's resolution.
- **Minimize non-opaque overdraw** — Render opaque geometry first; cull fully transparent meshes. GPU Counters in the Frame Debugger show fragment invocations ÷ pixels written.
- **Submit GPU work early** — Schedule all off-screen (shadow maps, deferred, post-process) in a separate command buffer committed before calling `nextDrawable()`; the drawable command buffer is committed last. Prevents CPU/GPU idle gaps visible in Metal System Trace.
- **Stream resources from a dedicated thread** — Allocating Metal resources on the render thread causes stalls. New Allocations track in Metal System Trace shows allocation events on the same timeline as GPU work.
- **Design for sustained thermal performance** — Test at Serious thermal state using new Xcode Device Conditions; use Xcode Energy Gauge thermal track to verify. **[NEW Device Conditions]**

### Memory Bandwidth (Best Practices 6–11)
- **Compress texture assets offline** — Use ASTC for A8+ devices; PVRTC for A7. Generate full mipmap chains. Savings example: 16 MB uncompressed → <3 MB compressed with mipmaps.
- **Enable GPU lossless texture compression** — Use `.private` storage mode and conservative usage flags (no `.shaderWrite` or `.pixelView` unless needed). For `.shared` textures, call `optimizeContentsForGPUAccess` after CPU writes.
- **Choose minimal pixel format** — Avoid unnecessary channels; prefer 16-bit over 32-bit for intermediate buffers. 128-bit formats (RGBA32Float) sample at quarter rate on Apple GPUs.
- **Set correct load/store actions** — Transient render targets must use `.clear` load action and `.dontCare` store action. Dependency Viewer flags incorrect store actions and quantifies the bandwidth cost. **[NEW issue annotations]**
- **Use MSAA with memoryless storage** — iOS tile-based GPUs resolve MSAA on-chip with no extra bandwidth; multisample textures should use `.memoryless` storage mode and `.dontCare` store action. Example: saves 85 MB bandwidth.
- **Leverage Tile Memory explicitly** — Use programmable blending for single-pass deferred rendering (G-buffer stays on-chip, never stored to system memory). iOS tile shaders and image blocks give similar benefits.

### Memory Footprint (Best Practices 12–18)
- **Use memoryless render targets** — Transient G-buffer attachments don't need system memory allocation; `.memoryless` storage mode eliminates backing store. Afterpulse saved ~60 MB by switching.
- **Stream only needed resources** — Load at launch what will always be needed; free splash/tutorial assets immediately; stream from a dedicated thread; use the Memory Viewer "unused" filter to find candidates.
- **Use smaller assets** — Compress meshes and textures; load lower mip levels under memory pressure; reduce off-screen buffer resolutions (shadow maps, SSAO).
- **Metal resource heaps + aliasing** — Allocate one large heap; alias intermediate render targets that have no dependencies (SSAO buffers, depth-of-field intermediates) to dramatically reduce peak memory.
- **Purgeable memory for caches** — Mark idle cache resources as `.volatile`; they don't count toward the app limit. Mark as `.nonVolatile` before use; check for `.empty` state (data may need regeneration). Reset to `.volatile` in the command buffer completion handler.
- **Release Metal function and PSO references** — Free `MTLFunction` references after PSO creation; they are not needed for rendering. Release PSOs no longer in use when memory-limited.

## APIs & Frameworks

### Metal **[NEW APIs]**
- `MTLDevice.currentAllocatedSize` — current total Metal resource memory in bytes **[NEW]**
- `MTLDevice.recommendedMaxWorkingSetSize` — recommended limit before performance degrades **[NEW]**
- `MTLStorageMode.memoryless` — render target exists only in tile memory, no system memory backing
- `MTLTextureUsage` — avoid `.shaderWrite` and `.pixelView` on render targets to enable lossless compression
- `MTLTexture.optimizeContentsForGPUAccess(region:mipLevel:slice:)` — explicit GPU compression for shared textures
- `MTLRenderPassAttachmentDescriptor.loadAction` — `.clear`, `.load`, `.dontCare`
- `MTLRenderPassAttachmentDescriptor.storeAction` — `.store`, `.dontCare`, `.multisampleResolve`, `.storeAndMultisampleResolve`
- `MTLHeap` — single large allocation; `MTLHeap.makeTexture(descriptor:offset:)` for aliasing
- `MTLTexture.setPurgeableState(_:)` — `.volatile`, `.nonVolatile`, `.empty`, `.keepCurrent`
- `MTLCommandBuffer.addCompletedHandler(_:)` — reset purgeable state after GPU completion
- `MTLCaptureManager.shared()` — programmatic GPU capture **[NEW]**
- `MTLCaptureManager.startCapture(with:)` / `stopCapture()` — trigger capture from app code **[NEW]**
- `MTLCaptureDescriptor` — configure capture scope **[NEW]**
- `MetalCaptureEnabled` Info.plist key — enable on-device programmatic capture **[NEW]**
- New `MTLPixelFormat.depth16Unorm` — half the memory of depth32 for shadow maps **[NEW]**

### Xcode Tools **[NEW]**
- Metal Memory Viewer — bar chart + table of all live Metal resources; filter by type/storage/usage/name/pixel format; issue annotations; "unused" and "used" filters **[NEW]**
- Metal Resource Allocations instrument — allocation/deallocation events on the Metal System Trace timeline **[NEW]**
- Dependency Viewer — render pass graph with issue icons for store action problems and unused attachments; new group compaction; new Issues button **[NEW improvements]**
- GPU Counters gauge — fragment invocations ÷ pixels written for overdraw analysis
- Thermal State track in Energy Gauge — shows device conditions and thermal state **[NEW]**

## Code Highlights

Multiple command buffers for early GPU submission:

```swift
// Early submission — all off-screen work
let offscreenCB = commandQueue.makeCommandBuffer()!
encodeOffscreenPasses(into: offscreenCB)
offscreenCB.commit()

// Wait for drawable as late as possible
let drawable = view.currentDrawable!

// Late submission — on-screen work
let onscreenCB = commandQueue.makeCommandBuffer()!
encodeOnscreenPasses(into: onscreenCB, drawable: drawable)
onscreenCB.present(drawable)
onscreenCB.commit()
```

Optimal private texture with conservative usage flags:

```swift
let descriptor = MTLTextureDescriptor.texture2DDescriptor(
    pixelFormat: .bgra8Unorm,
    width: width, height: height, mipmapped: false)
descriptor.storageMode = .private
descriptor.usage = [.renderTarget]        // not .shaderWrite, not .pixelView
let texture = device.makeTexture(descriptor: descriptor)!
```

Memoryless MSAA texture:

```swift
let msaaDesc = MTLTextureDescriptor.texture2DDescriptor(
    pixelFormat: .bgra8Unorm, width: w, height: h, mipmapped: false)
msaaDesc.storageMode = .memoryless
msaaDesc.usage = .renderTarget
msaaDesc.sampleCount = 4
msaaDesc.textureType = .type2DMultisample
let msaaTex = device.makeTexture(descriptor: msaaDesc)!
// Render pass: loadAction = .clear, storeAction = .multisampleResolve
```

Adaptive quality using available memory + programmatic capture:

```swift
let available = device.currentAllocatedSize
let limit = device.recommendedMaxWorkingSetSize
if available > limit * 9 / 10 {
    let captureManager = MTLCaptureManager.shared()
    let descriptor = MTLCaptureDescriptor()
    descriptor.captureObject = device
    try? captureManager.startCapture(with: descriptor)
    // render one frame normally
    commandBuffer.addCompletedHandler { _ in
        captureManager.stopCapture()
    }
}
```

## Takeaways

- Submitting all off-screen GPU work in a separate command buffer before requesting the drawable is the single highest-impact general performance fix: it eliminates CPU/GPU idle gaps that cause frame drops.
- `.memoryless` storage mode for transient G-buffer attachments costs nothing (no image quality trade-off) and can save tens of megabytes of footprint and bandwidth simultaneously.
- The Metal Memory Viewer's "unused" filter plus issue annotations is the fastest way to find resources that should be streamed out or marked volatile; start there before doing any manual memory analysis.
- Purgeable memory for resource caches and Metal heap aliasing for intermediate render targets are the two most powerful advanced techniques for managing peak memory without sacrificing visual quality.

---
_Source: WWDC19 Session 606 page (abstract, full transcript, and resource links)._
