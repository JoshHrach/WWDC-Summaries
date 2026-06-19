# Boost Performance with MetalFX Upscaling
**WWDC22 · Session 10103** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10103/)

_Platforms:_ macOS Ventura 13, iOS 16, iPadOS 16

## Overview
MetalFX is a new API introduced in 2022 that provides platform-optimized graphics effects for Metal applications, with an initial focus on upscaling. It allows applications to render at a lower resolution and then upscale to the target display resolution using GPU-optimized algorithms — achieving significant performance gains while maintaining high visual quality.

The framework provides two distinct upscaling effects: Spatial Upscaling (simple, high performance, single-frame) and Temporal Antialiasing and Upscaling (multi-frame history, highest quality output with full AA). The session covers API setup for both effects, implementation best practices for texture formats, mip bias, and jitter sequences, and a critical performance consideration around avoiding false dependencies between frames.

Games shipping on Mac with MetalFX support include Grid: Legends, Resident Evil: Village, and No Man's Sky.

## Key Topics

### Spatial Upscaling (`MTLFXSpatialScaler`)
- Single-frame analysis: upscales an anti-aliased, tone-mapped input texture
- Insert after tone-mapping, before any post-processing
- Best input: anti-aliased, noise-free, tone-mapped, values 0–1 in sRGB
- Color processing modes: `.perceptual` (best performance, sRGB 0–1), `.linear`, `.hdr`
- Mip bias recommendation: `log2(renderWidth / outputWidth)` — e.g., 2x upscale → −1 mip bias
- Create `MTLFXSpatialScaler` once at startup or on resolution change (expensive to create)

### Temporal AA and Upscaling (`MTLFXTemporalScaler`)
- Uses data from previous frames to achieve supersampling-quality output
- Required inputs per frame: jittered color, depth, motion vectors, and previous frame output (managed internally)
- Insert before any post-processing (post-FX will pollute temporal history)
- `resetHistory = true` on first frame and scene cuts
- Motion vector convention: render-resolution pixels, direction from current to previous frame position
- `motionVectorScale`: converts app's motion space to MetalFX's expected space
- Jitter sequence: Halton (2,3) with 32 samples for 2x upscaling (~8 samples per output pixel)
- Mip bias recommendation: `log2(renderWidth / outputWidth) - 1` — e.g., 2x upscale → −2 mip bias
- `reversedDepth`: set to match app's depth convention
- `jitterOffset`: must be correct; static scenes with wrong jitter cause object shifting and fuzzy lines

### Best Practices: Performance
- Avoid false dependencies between frames: do not bind the same resource for read and write in independent passes
- False dependencies prevent GPU overlap of successive frames' passes, amplifying the MetalFX encode cost
- Use separate Metal buffers for independent passes to eliminate hazards

### Choosing Between Effects
- Use Temporal AA & Upscaling if: you can supply jittered color, depth, and motion; or you lack a well-tuned AA solution
- Use Spatial Upscaling if: you already have a good AA solution, or cannot supply the temporal inputs

## APIs & Frameworks

**MetalFX** (new framework) **[NEW]**
- `MTLFXSpatialScalerDescriptor` **[NEW]**
  - `.inputWidth`, `.inputHeight` — render resolution
  - `.outputWidth`, `.outputHeight` — target resolution
  - `.colorTextureFormat: MTLPixelFormat`
  - `.outputTextureFormat: MTLPixelFormat`
  - `.colorProcessingMode: MTLFXSpatialScalerColorProcessingMode` — `.perceptual`, `.linear`, `.hdr`
  - `makeSpatialScaler(device: MTLDevice) -> MTLFXSpatialScaler?` **[NEW]**
- `MTLFXSpatialScaler` **[NEW]**
  - `.colorTexture: MTLTexture?`
  - `.outputTexture: MTLTexture?`
  - `encode(commandBuffer: MTLCommandBuffer)`
- `MTLFXTemporalScalerDescriptor` **[NEW]**
  - `.inputWidth`, `.inputHeight`
  - `.outputWidth`, `.outputHeight`
  - `.colorTextureFormat: MTLPixelFormat`
  - `.depthTextureFormat: MTLPixelFormat`
  - `.motionTextureFormat: MTLPixelFormat`
  - `.outputTextureFormat: MTLPixelFormat`
  - `makeTemporalScaler(device: MTLDevice) -> MTLFXTemporalScaler?` **[NEW]**
- `MTLFXTemporalScaler` **[NEW]**
  - `.colorTexture: MTLTexture?`
  - `.depthTexture: MTLTexture?`
  - `.motionTexture: MTLTexture?`
  - `.outputTexture: MTLTexture?`
  - `.resetHistory: Bool` — set `true` on first frame or scene cut
  - `.reversedDepth: Bool`
  - `.jitterOffset: CGPoint` — in range −0.5 to 0.5
  - `.motionVectorScale: CGPoint`
  - `encode(commandBuffer: MTLCommandBuffer)`

**Metal (supporting types)**
- `MTLDevice` — used to create scaler objects
- `MTLCommandBuffer` — scalers encode into command buffers
- `MTLPixelFormat` — texture formats for color, depth, motion, output

## Code Highlights

Spatial upscaler initialization:
```swift
let desc = MTLFXSpatialScalerDescriptor()
desc.inputWidth = 1280; desc.inputHeight = 720
desc.outputWidth = 2560; desc.outputHeight = 1440
desc.colorTextureFormat = .bgra8Unorm_srgb
desc.outputTextureFormat = .bgra8Unorm_srgb
desc.colorProcessingMode = .perceptual
let spatialScaler = desc.makeSpatialScaler(device: mtlDevice)!
```

Spatial upscaler per-frame encode:
```swift
spatialScaler.colorTexture = currentFrameColor
spatialScaler.outputTexture = currentFrameUpscaledColor
spatialScaler.encode(commandBuffer: cmdBuffer)
```

Temporal scaler initialization:
```swift
let desc = MTLFXTemporalScalerDescriptor()
desc.inputWidth = 1280; desc.inputHeight = 720
desc.outputWidth = 2560; desc.outputHeight = 1440
desc.colorTextureFormat = .rgba16Float
desc.depthTextureFormat = .depth32Float
desc.motionTextureFormat = .rg16Float
desc.outputTextureFormat = .rgba16Float
let temporalScaler = desc.makeTemporalScaler(device: mtlDevice)!
temporalScaler.motionVectorScale = CGPoint(x: 1280, y: 720)
```

Temporal scaler per-frame encode:
```swift
temporalScaler.resetHistory = firstFrameOrSceneCut
temporalScaler.colorTexture = currentFrameColor
temporalScaler.depthTexture = currentFrameDepth
temporalScaler.motionTexture = currentFrameMotion
temporalScaler.outputTexture = currentFrameUpscaledColor
temporalScaler.reversedDepth = reversedDepth
temporalScaler.jitterOffset = currentFrameJitterOffset
temporalScaler.encode(commandBuffer: cmdBuffer)
```

## Takeaways
- MetalFX is an Apple-optimized upscaling framework for Metal — start with Spatial Upscaling for quick adoption, advance to Temporal AA and Upscaling for maximum quality.
- Temporal upscaling requires motion vectors and depth; insert it before post-processing effects to preserve temporal history quality.
- Avoid false resource dependencies between frames to allow GPU overlap of successive frame workloads — particularly important because MetalFX encode time is now part of the critical path.
- Mip bias is critical for detail: use `log2(render/target) - 1` as a starting point for temporal upscaling, but increase (less negative) for high-frequency textures to eliminate flicker.

---
_Source: WWDC22 Session 10103 page (abstract, chapter summaries, code samples, and resource links)._
