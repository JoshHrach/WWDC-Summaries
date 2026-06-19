# Display HDR Video in EDR with AVFoundation and Metal
**WWDC22 · Session 110565** · [Watch](https://developer.apple.com/videos/play/wwdc2022/110565/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, tvOS 16

## Overview
This session surveys the full stack of Apple video frameworks for HDR/EDR playback and demonstrates four concrete workflows — from simple AVKit playback to Metal-accelerated real-time effects. The session maps each framework to its appropriate use case: AVKit for standard playback UI, AVFoundation's AVPlayerLayer for custom views, Core Video's `CADisplayLink` + `AVPlayerItemVideoOutput` for frame-by-frame processing, and `CVMetalTextureCache` for the most efficient path from decoded frame to Metal shader.

The core message is to use the highest-level framework that meets your needs. When you do need to drop down to Metal for custom shaders or Core Image for filter chains, the session provides the exact property combinations required to keep the full EDR signal intact from decode to display.

## Key Topics

### Framework Selection Guide
- **AVKit (`AVPlayerViewController`)**: Easiest; full transport UI, chapters, PiP, subtitles. Best for standard playback.
- **AVFoundation (`AVPlayer` + `AVPlayerLayer`)**: Play HDR media into a custom view; automatic EDR on supported displays.
- **Core Video (`CADisplayLink` + `AVPlayerItemVideoOutput`)**: Per-frame access as `CVPixelBuffer`; send to Core Image filters or Metal shaders.
- **Video Toolbox (`VTDecompressionSession`)**: Low-level direct hardware decoder access; for advanced use cases only.
- **Core Media**: Lowest level data types and media pipeline primitives.

### CAMetalLayer EDR Setup (required for custom rendering)
Three properties must all be set:
1. `CAMetalLayer.wantsExtendedDynamicRangeContent = true`
2. `MTLPixelFormat.rgba16Float` (or 10-bit PQ/HLG formats)
3. Color space: `kCGColorSpaceExtendedLinearDisplayP3`

### AVPlayerItemVideoOutput for Frame Access
- Create `AVPlayerItemVideoOutput` with output settings specifying:
  - `AVVideoColorPrimariesKey`: `AVVideoColorPrimaries_P3_D65`
  - `AVVideoTransferFunctionKey`: `AVVideoTransferFunction_Linear` (preserves EDR range)
  - `AVVideoYCbCrMatrixKey`: `AVVideoYCbCrMatrix_ITU_R_2020`
  - `AVVideoAllowWideColorKey`: `true`
  - `kCVPixelBufferPixelFormatTypeKey`: `kCVPixelFormatType_64RGBAHalf`
- AVFoundation automatically performs color conversion across clips with different color spaces.
- Use `CADisplayLink` (iOS) or `CVDisplayLink` (macOS) to access frames at display refresh rate.
- `AVPlayerItemVideoOutput.copyPixelBuffer(forItemTime:itemTimeForDisplay:)` — returns nil if no new frame available (skip render, keep previous frame).

### Core Image Integration
- Create `CIImage(cvPixelBuffer: buffer)` from the decoded frame.
- Chain EDR-compatible `CIFilter` instances; check for `kCICategoryHighDynamicRange` to verify compatibility.
- Render result to `CIRenderDestination` backed by the `CAMetalLayer`.
- Caution: non-EDR filters may clamp values to SDR.

### CVMetalTextureCache (recommended for Metal)
- Preferred over manual `IOSurface` → `MTLTexture` conversion.
- `CVMetalTextureCacheCreate` — create a texture cache associated with a `MTLDevice`.
- `CVMetalTextureCacheCreateTextureFromImage` — get a `CVMetalTexture` from a `CVPixelBuffer`.
- `CVMetalTextureGetTexture` — extract the `MTLTexture` for use in Metal shaders.
- Keeps `MTLTexture`-to-`IOSurface` mapping alive for performance; avoids manual format conversion concerns.
- In Objective-C: release `CVMetalTextureRef` only in Metal command buffer completion handlers.

## APIs & Frameworks

### AVKit
- `AVPlayerViewController` — full-featured playback UI with automatic EDR
- `AVPlayerViewController.player` — set to an `AVPlayer` instance

### AVFoundation
- `AVPlayer(url:)` — create player from URL
- `AVPlayerLayer(player:)` — render to a custom layer
- `AVPlayerItem` — media asset item
- `AVPlayerItemVideoOutput` — per-frame pixel buffer output
- `AVPlayerItemVideoOutput(outputSettings:)` — configure output format
- `AVPlayerItemVideoOutput.copyPixelBuffer(forItemTime:itemTimeForDisplay:)` — decode frame
- `AVPlayerItemVideoOutput.hasNewPixelBuffer(forItemTime:)` — check for new frame
- `AVPlayerItemVideoOutput.itemTime(forHostTime:)` — convert wall clock to item time
- `AVVideoColorPrimariesKey`, `AVVideoTransferFunctionKey`, `AVVideoYCbCrMatrixKey` — color space configuration
- `AVVideoColorPrimaries_P3_D65` — P3 D65 primaries
- `AVVideoTransferFunction_Linear` — linear transfer (EDR-preserving)
- `AVVideoYCbCrMatrix_ITU_R_2020` — ITU-R BT.2020 matrix
- `AVVideoAllowWideColorKey` — enable wide color output
- `kCVPixelFormatType_64RGBAHalf` — half-float RGBA pixel format

### Core Video
- `CADisplayLink` (iOS) / `CVDisplayLink` (macOS) — display-synchronized callback
- `CVPixelBuffer` — decoded video frame data
- `CVMetalTextureCache` — efficient CVPixelBuffer-to-MTLTexture bridge
- `CVMetalTextureCacheCreate(allocator:cacheAttributes:metalDevice:textureAttributes:cacheOut:)` **[key API]**
- `CVMetalTextureCacheCreateTextureFromImage(...)` **[key API]**
- `CVMetalTextureGetTexture(_:)` — extract MTLTexture

### Metal / Core Animation
- `CAMetalLayer.wantsExtendedDynamicRangeContent` — opt into EDR
- `CAMetalLayer.pixelFormat = .rgba16Float`
- `CAMetalLayer.colorspace = kCGColorSpaceExtendedLinearDisplayP3`
- `MTLPixelFormat.rgba16Float` — half-float pixel format

## Code Highlights

```swift
// Simple HDR playback in a custom view
let player = AVPlayer(url: videoURL)
let playerLayer = AVPlayerLayer(player: player)
playerLayer.frame = view.bounds
view.layer.addSublayer(playerLayer)
player.play()

// Configure CAMetalLayer for EDR custom rendering
layer.wantsExtendedDynamicRangeContent = true
layer.pixelFormat = .rgba16Float
layer.colorspace = CGColorSpace(name: CGColorSpace.extendedLinearDisplayP3)

// Configure AVPlayerItemVideoOutput for EDR frames
let videoColorProperties = [
    AVVideoColorPrimariesKey: AVVideoColorPrimaries_P3_D65,
    AVVideoTransferFunctionKey: AVVideoTransferFunction_Linear,
    AVVideoYCbCrMatrixKey: AVVideoYCbCrMatrix_ITU_R_2020
]
let outputSettings: [String: Any] = [
    AVVideoAllowWideColorKey: true,
    AVVideoColorPropertiesKey: videoColorProperties,
    kCVPixelBufferPixelFormatTypeKey as String: kCVPixelFormatType_64RGBAHalf
]
let videoOutput = AVPlayerItemVideoOutput(outputSettings: outputSettings)

// CVMetalTextureCache usage
var mtlTextureCache: CVMetalTextureCache?
CVMetalTextureCacheCreate(kCFAllocatorDefault, nil, mtlDevice, nil, &mtlTextureCache)

var cvTexture: CVMetalTexture?
CVMetalTextureCacheCreateTextureFromImage(
    kCFAllocatorDefault, mtlTextureCache!, pixelBuffer, nil,
    .rgba16Float, width, height, 0, &cvTexture)
let texture = CVMetalTextureGetTexture(cvTexture!)
```

## Takeaways
- Choose the highest-level framework that meets your needs: AVKit for standard playback, AVPlayerLayer for custom views, DisplayLink + AVPlayerItemVideoOutput for per-frame processing, CVMetalTextureCache for Metal.
- Three `CAMetalLayer` properties must all be set correctly to preserve the full EDR signal: `wantsExtendedDynamicRangeContent`, `.rgba16Float` pixel format, and extended linear P3 color space.
- `AVVideoTransferFunction_Linear` in the output settings is the key to preserving EDR values above 1.0 in decoded pixel buffers.
- `CVMetalTextureCache` is the recommended way to bridge `CVPixelBuffer` to `MTLTexture` — faster and simpler than manual `IOSurface` conversion.

---
_Source: WWDC22 Session 110565 page (abstract, chapter summaries, code samples, and resource links)._
