# Display EDR Content with Core Image, Metal, and SwiftUI
**WWDC22 · Session 10114** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10114/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13

## Overview
This session explains how to add Extended Dynamic Range (EDR) rendering support to a Core Image-based, multiplatform SwiftUI application using Metal and MTKView. EDR allows pixels to go beyond the standard 0–1 brightness range, reaching brightness values above SDR white (represented as 1.0) up to the display's current headroom. The session introduces a new sample code project demonstrating best practices for wiring together SwiftUI, ViewRepresentable, MTKView, Core Image, and a Renderer delegate, then shows the three-step process to enable EDR, and finally covers how to use Core Image's 150+ EDR-capable filters.

## Key Topics

### EDR Terminology
- **SDR** (Standard Dynamic Range): RGB 0.0 (black) to 1.0 (white).
- **EDR** (Extended Dynamic Range): 0.0 is still black, 1.0 is SDR white, values >1.0 are brighter than SDR white — up to the display headroom.
- **Headroom**: Display's current maximum nits / SDR white nits. Dynamic — changes with ambient conditions and display brightness. Must be read every frame.
- Values above headroom are clipped.

### SwiftUI + Metal + Core Image Architecture
Three-layer architecture demonstrated in the new sample project:
1. **`MetalView`** (`ViewRepresentable`): Wraps `MTKView` for SwiftUI. Sets `preferredFramesPerSecond` (for animation) or `enableSetNeedsDisplay` (for interactive/editing). Sets `framebufferOnly = false` to allow Core Image compute pipeline.
2. **`Renderer`** (MTKView delegate): Owns `MTLCommandQueue` and `CIContext`. Implements `draw(in:)` — calculates content scale factor, creates `CIRenderDestination`, requests a `CIImage` from the content provider, composites, and renders.
3. **`ContentView`**: Provides the `imageProvider` closure that returns a `CIImage` for the requested time/scale/headroom.

### Three Steps to Enable EDR
1. **Initialize view for EDR**: Set `CAMetalLayer.wantsExtendedDynamicRangeContent = true`, `MTKView.colorPixelFormat = .rgba16Float`, and `MTKView.colorspace = CGColorSpace.extendedLinearDisplayP3`.
2. **Calculate headroom every frame in `draw(in:)`**:
   - macOS: `screen?.maximumExtendedDynamicRangeColorComponentValue`
   - iOS: `screen?.currentEDRHeadroom`
   - Pass headroom to the image provider on every draw call.
3. **Build EDR CIImages using the headroom value**: Scale bright colors relative to headroom; use EDR-aware filters.

### Core Image EDR Filters
- Over 150 built-in `CIFilter` classes support EDR — they handle values outside 0–1 natively because Core Image's working color space is unclamped and linear.
- Examples: `CIColorControls`, `CIExposureAdjust`, `CILinearGradient`, `CIRippleTransition`, `CIAreaLogarithmicHistogram` **[NEW]**, gradient generators.
- Check EDR support: look for `kCICategoryHighDynamicRange` in `filter.attributes[kCIAttributeFilterCategories]`.
- Xcode QuickLook for `CIFilter` variables shows documentation including categories **[NEW]**.
- `CIColorCubeWithColorSpace.extrapolate = true` — new property **[NEW]** allows applying SDR cube data to EDR images with extrapolation.
- Best practice: use EDR brightness (values >1.0) in moderation — less is more impactful.

### Custom Kernel Best Practices
- Remove `clamp`, `min`, or `max` calls that artificially limit RGB values to 0–1 if they're not needed.
- Alpha must remain in 0–1 range — never multiply alpha by the EDR scale factor (only scale RGB channels).
- Custom kernels in unclamped linear color space will correctly pass through EDR values.

## APIs & Frameworks

### Core Image
- `CIRenderDestination(width:height:pixelFormat:commandBuffer:mtlTextureProvider:)` — render to MTLTexture
- `CIContext.startTask(toRender:from:to:at:)` — asynchronous render task
- `CIFilter.rippleTransition()` — supports EDR
- `CIFilter.linearGradient()` — generate EDR gradient by setting color0 to values >1.0
- `CIFilter.checkerboardGenerator()` — procedural content generation
- `CIFilter.colorCubeWithColorSpace()` — color lookup table
- `CIColorCubeWithColorSpace.extrapolate: Bool` — **[NEW]** extrapolate SDR cube data for EDR input
- `CIFilter.areaLogarithmicHistogram()` — **[NEW]** histogram for arbitrary brightness range
- `kCICategoryHighDynamicRange` — category constant to check EDR support
- `CIColor(red:green:blue:colorSpace:)` — create EDR color using extended linear color space

### Metal / MetalKit
- `MTKView` — metal rendering view
- `MTKViewDelegate` — renderer delegate protocol
- `MTKView.framebufferOnly = false` — required for Core Image compute pipeline
- `MTKView.colorPixelFormat = .rgba16Float` — required for EDR
- `MTKView.colorspace` — set to `CGColorSpace.extendedLinearDisplayP3`
- `CAMetalLayer.wantsExtendedDynamicRangeContent = true` — opt in to EDR **[NEW behavior]**

### SwiftUI
- `ViewRepresentable` / `NSViewRepresentable` / `UIViewRepresentable` — bridge MTKView to SwiftUI
- `MTKView.preferredFramesPerSecond` — animation-driven rendering
- `MTKView.enableSetNeedsDisplay` — control-driven rendering (image editing apps)

### Screen / Display
- macOS: `NSScreen.maximumExtendedDynamicRangeColorComponentValue` — current EDR headroom
- iOS: `UIScreen.currentEDRHeadroom` — current EDR headroom **[NEW]**
- `CGColorSpace.extendedLinearDisplayP3` — wide gamut, unclamped color space for EDR rendering
- `CGColorSpace.extendedLinearSRGB` — for creating EDR `CIColor` values

## Code Highlights

```swift
// Step 1: Initialize MTKView for EDR
if let caMtlLayer = view.layer as? CAMetalLayer {
    caMtlLayer.wantsExtendedDynamicRangeContent = true
    view.colorPixelFormat = .rgba16Float
    view.colorspace = CGColorSpace(name: CGColorSpace.extendedLinearDisplayP3)
}

// Step 2: Get headroom every frame
let screen = view.window?.screen
#if os(macOS)
let headroom = screen?.maximumExtendedDynamicRangeColorComponentValue ?? 1.0
#else
let headroom = screen?.currentEDRHeadroom ?? 1.0
#endif

// Step 3: Use headroom in CIFilter (linear gradient for specular highlight)
let gradient = CIFilter.linearGradient()
let w = min(headroom, 8.0)  // Cap at reasonable max
gradient.color0 = CIColor(red: w, green: w, blue: w,
    colorSpace: CGColorSpace(name: CGColorSpace.extendedLinearSRGB)!)!
gradient.color1 = .clear

// Apply SDR color cube data to EDR image using extrapolation
let cube = CIFilter.colorCubeWithColorSpace()
cube.cubeData = sdrCubeData
cube.extrapolate = true  // NEW: extrapolate for EDR input
cube.inputImage = edrImage
```

## Takeaways
- EDR headroom is dynamic — always read it fresh every frame before building the image; never cache it across frames.
- Three steps: set `rgba16Float` + `wantsExtendedDynamicRangeContent` on the Metal layer, read headroom per frame, use EDR-aware CIFilters that generate values >1.0.
- Use EDR brightness sparingly — a single specular highlight at `headroom` brightness is far more impactful than an entirely bright image.
- The new `CIColorCubeWithColorSpace.extrapolate` property is a low-effort path to make existing SDR LUT-based filters work with EDR content.

---
_Source: WWDC22 Session 10114 page (abstract, chapter summaries, code samples, and resource links)._
