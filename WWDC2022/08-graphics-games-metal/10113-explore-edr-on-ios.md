# Explore EDR on iOS
**WWDC22 · Session 10113** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10113/)

_Platforms:_ iOS 16, iPadOS 16

## Overview
EDR (Extended Dynamic Range) is Apple's HDR representation and rendering pipeline. This session announces that the EDR APIs — previously macOS-only — are now available on iOS and iPadOS 16, enabling apps to render HDR content on iPhone and iPad without clipping to SDR. The session covers reading HDR still images with Image I/O, converting them to Metal textures, opting a `CAMetalLayer` into EDR, querying headroom values for informed rendering decisions, and delegating tone mapping to the system using `CAEDRMetadata`.

Two new pro-color features are also introduced for the 12.9-inch iPad Pro with Liquid Retina XDR display: Reference Mode (a fixed 100-nit SDR / 1000-nit HDR reference response for color-critical workflows) and EDR rendering over Sidecar (extending iPad Pro as a reference-grade secondary display for Apple silicon Macs).

## Key Topics

**EDR pixel representation** — SDR content maps to the 0–1 float range; values above 1.0 represent brightness beyond SDR white. EDR is linear (2.0 EDR is not perceptibly twice as bright as 1.0). Values above the current headroom are clipped; no automatic tone mapping is applied unless explicitly requested via `CAEDRMetadata`.

**EDR headroom** — Instantaneous headroom = display max brightness / current SDR brightness. A 1000-nit display at 100-nit SDR brightness has a 10× headroom. Queried via `UIScreen.potentialEDRHeadroom` (max possible) and `UIScreen.currentEDRHeadroom` (current). Unlike macOS (`NSScreen`), iOS does not broadcast a headroom-change notification; it does broadcast `UIScreen.referenceDisplayModeStatusDidChangeNotification`.

**Reference Mode** — New on 12.9-inch iPad Pro; fixes SDR brightness at 100 nits and HDR peak at 1000 nits, disables True Tone / Auto-Brightness / Night Shift, and provides a one-to-one media-to-display mapping for HLG, HDR10, SDR BT.709, SDR P3, and Dolby Vision formats.

**EDR over Sidecar** — When Reference Mode is enabled on iPad Pro, Sidecar enables reference-grade EDR rendering from an Apple silicon Mac, matching the Mac's HDR video preset with P3 color and D65 white point.

**Rendering workflow** — Load HDR image via `CGImageSourceCreateWithURL` → draw into a 16-bit float `CGBitmapContext` → copy into `MTLTexture (rgba16Float)` → render to a `CAMetalLayer` with `wantsExtendedDynamicRangeContent = true` and an extended linear color space.

**Supported formats** — iOS supports `MTLPixelFormatRGBA16Float` with extended linear color spaces (e.g., `kCGColorSpaceExtendedLinearDisplayP3`), and `MTLPixelFormatBGR10A2Unorm` with PQ or HLG color spaces.

**CAEDRMetadata tone mapping** — Opt into system tone mapping by assigning a `CAEDRMetadata` instance to `CAMetalLayer.edrMetadata`. Constructors for HLG (no parameters), HDR10 with mastering display luminance, and HDR10 with SEI messages (`MasterDisplayColourVolume`, `ContentLightLevelInformation`).

## APIs & Frameworks

### QuartzCore / CAMetalLayer
- `CAMetalLayer.wantsExtendedDynamicRangeContent` — enables EDR rendering on the layer **[NEW on iOS]**
- `CAMetalLayer.pixelFormat` — set to `MTLPixelFormatRGBA16Float` or `MTLPixelFormatBGR10A2Unorm`
- `CAMetalLayer.colorspace` — set to an extended or HDR color space (e.g., `kCGColorSpaceExtendedLinearDisplayP3`)
- `CAMetalLayer.edrMetadata` — assign `CAEDRMetadata` to opt into system tone mapping

### CAEDRMetadata
- `CAEDRMetadata.isAvailable` — checks whether tone mapping is supported on the current platform **[NEW on iOS]**
- `CAEDRMetadata.hlg` — HLG tone mapping metadata constructor
- `CAEDRMetadata.hdr10(minLuminance:maxLuminance:opticalOutputScale:)` — HDR10 with mastering display luminance values
- `CAEDRMetadata.hdr10(displayInfo:contentInfo:opticalOutputScale:)` — HDR10 with `MasterDisplayColourVolume` and `ContentLightLevelInformation` SEI messages

### UIKit / UIScreen
- `UIScreen.potentialEDRHeadroom` — maximum possible EDR headroom for the display **[NEW]**
- `UIScreen.currentEDRHeadroom` — current instantaneous EDR headroom **[NEW]**
- `UIScreen.referenceDisplayModeStatus` — `.enabled`, `.limited`, `.notEnabled`, `.notSupported` **[NEW]**
- `UIScreen.referenceDisplayModeStatusDidChangeNotification` — notification sent when Reference Mode status changes **[NEW]**

### Image I/O
- `CGImageSourceCreateWithURL(_:_:)` — load image from URL
- `CGImageSourceCreateImageAtIndex(_:_:_:)` — create `CGImage` from source
- `kCGImageSourceShouldAllowFloat` — new option to request floating-point buffers for HDR formats **[NEW]**
- `CGBitmapContext` — decode HDR image into linear float buffer

### Metal
- `MTLPixelFormatRGBA16Float` — 16-bit float pixel format for EDR
- `MTLPixelFormatBGR10A2Unorm` — 10-bit packed pixel format for PQ/HLG EDR
- `MTLTextureDescriptor` — configure texture for EDR rendering
- `MTLTexture.replace(region:mipmapLevel:withBytes:bytesPerRow:)` — upload bitmap data to GPU

### Color Spaces
- `kCGColorSpaceExtendedLinearDisplayP3` — extended linear Display P3 (for float16 EDR)
- `kCGColorSpaceExtendedLinearSRGB` — alternative linear extended color space
- PQ (`kCGColorSpaceITUR_2100_PQ`) and HLG (`kCGColorSpaceITUR_2100_HLG`) — for 10-bit packed formats

## Code Highlights

Opting a CAMetalLayer into EDR:
```swift
var layer = CAMetalLayer()
layer.wantsExtendedDynamicRangeContent = true
layer.pixelFormat = .rgba16Float
layer.colorspace = CGColorSpace(name: kCGColorSpaceExtendedLinearDisplayP3)
```

Querying headroom and observing Reference Mode:
```swift
let screen = windowScene.screen
let maxPotentialEDR = screen.potentialEDRHeadroom
if maxPotentialEDR < 1.5 { /* SDR path */ }

// In draw callback:
let maxEDR = screen.currentEDRHeadroom  // use for tone mapping

// Observe Reference Mode changes:
NotificationCenter.default.addObserver(self,
    selector: #selector(screenChangedEvent(_:)),
    name: UIScreen.referenceDisplayModeStatusDidChangeNotification,
    object: nil)
```

Applying system tone mapping:
```swift
if CAEDRMetadata.isAvailable {
    layer.edrMetadata = CAEDRMetadata.hdr10(minLuminance: 0.005,
                                             maxLuminance: 1000,
                                             opticalOutputScale: 100)
}
```

## Takeaways
- EDR APIs (`wantsExtendedDynamicRangeContent`, `CAEDRMetadata`, headroom queries) are now available on iOS 16 with the same interface as macOS — existing macOS EDR code requires no changes to run on iOS.
- Always use extended color space variants (e.g., `kCGColorSpaceExtendedLinearDisplayP3`) with float16 EDR; standard (non-extended) color spaces clip to SDR.
- Query `UIScreen.currentEDRHeadroom` each frame to perform manual tone mapping, or delegate tone mapping entirely to the system via `CAEDRMetadata`.
- Reference Mode on 12.9-inch iPad Pro provides a constant 10× EDR headroom and disables all adaptive display features, enabling professional color-grading workflows on iPad.

---
_Source: WWDC22 Session 10113 page (abstract, chapter summaries, code samples, and resource links)._
