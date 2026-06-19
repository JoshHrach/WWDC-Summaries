# Edit and Play Back HDR Video with AVFoundation
**WWDC20 · Session 10009** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10009/)

_Platforms:_ macOS Big Sur 11 (HDR editing and playback; HDR playback was previously iOS-only)

## Overview
macOS Big Sur adds HDR video editing support to AVFoundation, building on the HDR playback capability Apple introduced on iOS two years earlier and macOS the year prior. HDR (High Dynamic Range) video greatly extends luminance range beyond SDR's 100 nit ceiling, up to 10,000 nits, and is commonly paired with Wide Color (ITU-R 2020 primaries) and 10-bit pixel formats. Apple supports both HLG and PQ HDR transfer functions.

The session walks through three `AVVideoComposition` construction patterns and explains what each requires to correctly handle HDR sources. It also covers color property control for mixed-format input scenarios and the `AVPlayer.eligibleForHDRPlayback` API for conditional HDR preview.

## Key Topics

### Three Ways to Build AVVideoComposition

**1. Built-in compositor with AVMutableVideoComposition**
- Used for blending multiple video layers and frame-level geometry transformations (crop, scale, rotate, transition effects).
- HDR support is automatic — no code changes required. The built-in compositor detects HDR input frames and handles them natively.
- Caveat: if the app explicitly sets color properties to SDR values, those are respected and the compositor converts HDR input to SDR before compositing.

**2. `applyingCIFiltersWithHandler` constructor**
- Used for filtering effects (blur, color tint, artistic effects) on a single video track.
- All built-in `CIFilter` types handle HDR sources without modification.
- Custom Metal CoreImage kernels must be updated to be HDR-aware: intensities can exceed 1.0, so any assumption of [0, 1] clamped range may produce incorrect results (e.g., a color inverter returning `1.0 - s.r` will produce negative values for HDR pixels).

**3. Custom compositor class**
- Most flexible option; supports multi-layer blending, per-layer geometry transformations, and filter effects.
- Requires explicit opt-in for HDR: must declare support for 10-bit pixel formats and set `supportsHDRSourceFrames = true`. Without this flag, the framework converts HDR frames to SDR before delivering them to the compositor.
- Must also set `supportsWideColorSourceFrames = true` because HDR sources carry Wide Color.

### Color Property Control
When the app does not specify composition color properties, the framework follows a preference order: HLG > PQ > SDR. Setting `colorPrimaries`, `colorTransferFunction`, and `colorYCbCrMatrix` overrides this and forces all input — regardless of source format — to be converted to the specified color space before reaching the compositor.

Standard value combinations:
- HLG: `AVVideoColorPrimaries_ITU_R_2020`, `AVVideoTransferFunction_ITU_R_2100_HLG`, `AVVideoYCbCrMatrix_ITU_R_2020`
- PQ: `AVVideoColorPrimaries_ITU_R_2020`, `AVVideoTransferFunction_SMPTE_ST_2084_PQ`, `AVVideoYCbCrMatrix_ITU_R_2020`
- SDR: `AVVideoColorPrimaries_ITU_R_709_2`, `AVVideoTransferFunction_ITU_R_709_2`, `AVVideoYCbCrMatrix_ITU_R_709_2`

### HDR Playback Eligibility
`AVPlayer.eligibleForHDRPlayback` (class var, available macOS 10.15+) returns `true` when the system hardware and at least one connected display both support HDR playback. HDR playback is not available on watchOS or macOS Catalyst. This property should gate HDR composition during preview — apps that are not eligible can still export HDR using `AVAssetExportSession` or `AVAssetWriter`.

## APIs & Frameworks

### AVFoundation
- `AVMutableVideoComposition` — `.colorPrimaries`, `.colorTransferFunction`, `.colorYCbCrMatrix` (existing properties; set to control HDR behavior)
- `AVMutableVideoComposition(asset:applyingCIFiltersWithHandler:)` — CI filter–based composition constructor
- `AVAsynchronousCIImageFilteringRequest` — passed to the handler; `.sourceImage: CIImage`, `.finish(with:context:)`
- `AVVideoCompositing` protocol (custom compositor) — two new properties:
  - `supportsHDRSourceFrames: Bool` **[NEW]** — must be `true` to receive HDR frames
  - `supportsWideColorSourceFrames: Bool` **[NEW]** — must be `true` to receive Wide Color frames
- `sourcePixelBufferAttributes` / `requiredPixelBufferAttributesForRenderContext` — include 10-bit format `kCVPixelFormatType_420YpCbCr10BiPlanarVideoRange` to support HDR
- `AVPlayer.eligibleForHDRPlayback: Bool` **[NEW class var]** — checks system + display HDR capability
- `AVAssetExportSession`, `AVAssetWriter` — used for export; HDR export settings covered in companion session "Export HDR Media in Your App with AVFoundation"

### Core Image / Metal
- All built-in `CIFilter` types — HDR-aware, no changes needed
- Custom Metal CoreImage kernels — must handle values > 1.0; `#include <CoreImage/CoreImage.h>` required

## Code Highlights

Custom compositor configured for HDR:
```swift
class SampleCustomCompositor: NSObject, AVVideoCompositing {
    var sourcePixelBufferAttributes: [String: Any]? = [
        kCVPixelBufferPixelFormatTypeKey as String:
            [kCVPixelFormatType_420YpCbCr10BiPlanarVideoRange]
    ]
    var requiredPixelBufferAttributesForRenderContext: [String: Any] = [
        kCVPixelBufferPixelFormatTypeKey as String:
            [kCVPixelFormatType_420YpCbCr10BiPlanarVideoRange]
    ]
    var supportsHDRSourceFrames = true
    var supportsWideColorSourceFrames = true

    func startRequest(_ request: AVAsynchronousVideoCompositionRequest) { ... }
    func renderContextChanged(_ newRenderContext: AVVideoCompositionRenderContext) {}
}
```

Conditional HDR color properties based on playback eligibility:
```swift
if AVPlayer.eligibleForHDRPlayback {
    videoComposition.colorPrimaries = AVVideoColorPrimaries_ITU_R_2020
    videoComposition.colorTransferFunction = AVVideoTransferFunction_ITU_R_2100_HLG
    videoComposition.colorYCbCrMatrix = AVVideoYCbCrMatrix_ITU_R_2020
} else {
    videoComposition.colorPrimaries = AVVideoColorPrimaries_ITU_R_709_2
    videoComposition.colorTransferFunction = AVVideoTransferFunction_ITU_R_709_2
    videoComposition.colorYCbCrMatrix = AVVideoYCbCrMatrix_ITU_R_709_2
}
```

HDR-aware Metal CoreImage kernel (highlights pixels above SDR range):
```metal
#include <metal_stdlib>
#include <CoreImage/CoreImage.h>
using namespace metal;

extern "C" float4 HDRHighlight(coreimage::sample_t s, coreimage::destination dest) {
    if (s.r > 1.0 || s.g > 1.0 || s.b > 1.0)
        return float4(2.0, 0.0, 0.0, 1.0); // ultra-bright red in HDR
    else
        return s;
}
```

## Takeaways
- Built-in compositor and `applyingCIFiltersWithHandler` with stock `CIFilter`s are HDR-capable without any code changes — apps using these patterns get HDR editing for free.
- Custom compositors must declare `supportsHDRSourceFrames = true` and include 10-bit pixel format types; omitting this flag causes the framework to silently downconvert HDR input to SDR.
- Use `AVPlayer.eligibleForHDRPlayback` to gate the preview composition color space; HDR export eligibility is separate from playback eligibility.
- Custom Metal CoreImage kernels must handle luminance values above 1.0 — SDR-era code that clamps or inverts at 1.0 will produce incorrect results on HDR sources.

---
_Source: WWDC20 Session 10009 page (abstract, transcript, code samples, and resource links)._
