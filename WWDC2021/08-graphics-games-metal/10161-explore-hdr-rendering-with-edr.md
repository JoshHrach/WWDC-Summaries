# Explore HDR Rendering with EDR
**WWDC21 · Session 10161** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10161/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, tvOS 15

## Overview
EDR (Extended Dynamic Range) is Apple's HDR representation and rendering pipeline, analogous to color management. It is an extension of SDR color management—EDR 0.0 = black, EDR 1.0 = SDR reference white, and EDR values above 1.0 represent specular highlights and emissive surfaces. This session explains the EDR model conceptually, describes the four-step process for enabling EDR in any app, covers the native EDR API on macOS via `CAMetalLayer` and `NSOpenGLView`, and details best practices including adaptive tone-mapping via `CAEDRMetadata` and monitoring dynamic EDR headroom via `NSScreen`.

EDR is adaptive: it adjusts to display capabilities and ambient conditions so that HDR content looks optimal across everything from a conventional backlit MacBook display (up to 2× SDR headroom) to Pro Display XDR (up to 400× SDR at minimum brightness), and even renders true HDR on SDR displays in dim environments.

## Key Topics
- **EDR Model:** Floating point representation. 0.0 = black; 1.0 = SDR reference white; values above 1.0 = specular highlights/emissives; EDRmax = highest renderable value on current display and brightness. Values above EDRmax clip to peak white.
- **Four Steps to Enable EDR:** (1) Request EDR by setting `wantsExtendedDynamicRangeContent = YES` on the layer; (2) Assign an extended-range colorspace (e.g., `kCGColorSpaceExtendedLinearDisplayP3`) to the layer/buffer; (3) Select a floating-point pixel buffer format (e.g., `MTLPixelFormatRGBA16Float`); (4) Generate pixel values that actually exceed 1.0 (highlights/emissives).
- **AVPlayer (Auto EDR):** `AVPlayer` automatically renders supported HDR video formats (Dolby Vision, HDR10, HLG) as EDR on all platforms except watchOS. No code changes required for existing `AVPlayer`-based apps.
- **CAMetalLayer (Native EDR):** Set `wantsExtendedDynamicRangeContent`, `colorspace`, and `pixelFormat`. Load HDR still images via `ImageIO` (which decodes to EDR floating-point buffers), create `MTLTexture` of type `RGBA16Float`, and render through the EDR-enabled Metal pipeline.
- **NSOpenGLView (EDR):** Set `wantsExtendedDynamicRangeOpenGLSurface = YES` and use `NSOpenGLPFAColorFloat` with `NSOpenGLPFAColorSize = 64`. No explicit colorspace required (NSOpenGLView is not auto color-managed).
- **Promoting Existing Colorspace to Extended Range:** Use `CGColorSpaceCreateExtended(existingColorspace)` to extend a window's current colorspace to its extended-range variant, enabling apps that do not set colorspace explicitly.
- **CAEDRMetadata Tone Mapper:** Available on macOS via `CAMetalLayer.EDRMetadata`. Preset constructors for HLG and HDR10; performs optical-to-optical tone mapping and soft-clipping for values above current EDRmax.
- **Dynamic EDR Headroom via NSScreen:** `NSScreen.maximumExtendedDynamicRangeColorComponentValue` (dynamic, changes with brightness/True Tone), `maximumPotentialExtendedDynamicRangeColorComponentValue` (static max potential), `maximumReferenceExtendedDynamicRangeColorComponentValue` (reference rendering guarantee). Subscribe to `NSApplicationDidChangeScreenParametersNotification` to track changes.
- **Best Practices:** Only use EDR for specular highlights and emissive surfaces; SDR content should remain in 0.0–1.0. Do not stretch SDR to HDR. Enable EDR only when content has bright highlights AND the display has headroom (EDRmax > 1.0). FP16 buffers consume more bandwidth; use EDR judiciously.

## APIs & Frameworks

**AVFoundation**
- `AVPlayer` – Auto EDR rendering for HDR video (Dolby Vision, HDR10, HLG); no API changes needed

**Core Animation / Metal**
- `CAMetalLayer.wantsExtendedDynamicRangeContent: Bool` – Opt-in to EDR
- `CAMetalLayer.colorspace: CGColorSpace` – Set to extended-range colorspace
- `CAMetalLayer.pixelFormat` – Set to `MTLPixelFormatRGBA16Float` or similar FP format
- `CAEDRMetadata` **[NEW]** – System tone mapper for HDR video metadata
  - `CAEDRMetadata.HLGMetadata()` **[NEW]** – HLG tone mapper (no parameters)
  - `CAEDRMetadata.HDR10Metadata(minLuminance:maxLuminance:opticalOutputScale:)` **[NEW]** – HDR10 tone mapper with explicit mastering display params
- `CAMetalLayer.EDRMetadata: CAEDRMetadata?` **[NEW]** – Assign to enable system tone mapping on the layer

**Core Graphics / ImageIO**
- `CGColorSpaceCreateWithName(kCGColorSpaceExtendedLinearDisplayP3)` – Extended linear wide-gamut colorspace for EDR
- `CGColorSpaceCreateExtended(_:)` – Promote any colorspace to its extended-range variant
- `CGColorSpaceCreateCopyByMatchingToColorSpace(_:_:_:_:)` – Convert EDR linear color to app's nonlinear colorspace
- `CGImageSourceCreateWithURL(_:_:)` / `CGImageSourceCreateImageAtIndex(_:_:_:)` – Load HDR still images (ImageIO decodes to EDR floating-point)
- `CGBitmapContextCreate` with `kCGBitmapFloatComponents | kCGBitmapByteOrder16Host` – FP16 bitmap for EDR pixel data
- `CGContextDrawImage(_:_:_:)` – Draw decoded EDR image into FP16 context

**Metal**
- `MTLPixelFormatRGBA16Float` – FP16 pixel format required for EDR rendering
- `MTLDevice.newTexture(descriptor:)` – Create FP16 texture from descriptor
- `MTLTexture.replace(region:mipmapLevel:withBytes:bytesPerRow:)` – Load EDR bitmap data into texture

**AppKit**
- `NSScreen.maximumExtendedDynamicRangeColorComponentValue: CGFloat` – Current EDRmax (dynamic)
- `NSScreen.maximumPotentialExtendedDynamicRangeColorComponentValue: CGFloat` – Max potential EDR (static)
- `NSScreen.maximumReferenceExtendedDynamicRangeColorComponentValue: CGFloat` – Reference rendering max (static, 0.0 if unsupported)
- `NSApplicationDidChangeScreenParametersNotification` – Posted when display params (brightness, True Tone) change; re-query EDRmax
- `NSWindow.colorSpace` – Used to get and set the extended-range colorspace
- `NSOpenGLView.wantsExtendedDynamicRangeOpenGLSurface: Bool` – Opt-in to EDR for OpenGL
- `NSOpenGLPFAColorFloat`, `NSOpenGLPFAColorSize` – Required for EDR in OpenGL pixel format

## Code Highlights
Four-step EDR opt-in for CAMetalLayer (Objective-C):
```objc
// Step 1: Opt-in to EDR
metalLayer.wantsExtendedDynamicRangeContent = YES;
// Step 2: Extended-range colorspace
metalLayer.colorspace = CGColorSpaceCreateWithName(kCGColorSpaceExtendedLinearDisplayP3);
// Step 3: FP16 pixel format
metalLayer.pixelFormat = MTLPixelFormatRGBA16Float;
// Step 4: Generate pixels > 1.0 in your Metal render pipeline
```

Enabling HLG and HDR10 tone-mapping via CAEDRMetadata:
```objc
// HLG
CAEDRMetadata *hlgMetadata = [CAEDRMetadata HLGMetadata];
metalLayer.EDRMetadata = hlgMetadata;

// HDR10
CAEDRMetadata *hdr10Metadata = [CAEDRMetadata HDR10MetadataWithMinLuminance:0.01
                                                                maxLuminance:1000.0
                                                         opticalOutputScale:100.0];
metalLayer.EDRMetadata = hdr10Metadata;
```

Monitoring dynamic EDR headroom:
```objc
NSScreen *screen = window.screen;
double maxPotential = screen.maximumPotentialExtendedDynamicRangeColorComponentValue;
double maxReference = screen.maximumReferenceExtendedDynamicRangeColorComponentValue;

[[NSNotificationCenter defaultCenter]
    addObserver:self
       selector:@selector(screenChanged:)
           name:NSApplicationDidChangeScreenParametersNotification
         object:nil];

- (void)screenChanged:(NSNotification *)notification {
    double maxEDR = window.screen.maximumExtendedDynamicRangeColorComponentValue;
    // Adjust scene exposure or apply tone-mapping based on maxEDR
}
```

## Takeaways
- EDR 1.0 is always SDR reference white; only emissive surfaces and specular highlights should exceed 1.0—never stretch SDR content into the EDR range.
- `AVPlayer` handles EDR automatically for HDR video; the native `CAMetalLayer` API is needed only for games, custom renderers, and pro apps requiring direct control of HDR pixel values.
- Subscribe to `NSApplicationDidChangeScreenParametersNotification` and re-query `maximumExtendedDynamicRangeColorComponentValue` to adapt tone mapping and bloom effects dynamically as display brightness changes.
- Use `CAEDRMetadata` to enable Apple's system tone mapper rather than writing a custom one; it handles soft-clipping and format-specific optical transfer functions for HLG and HDR10 content.

---
_Source: WWDC21 Session 10161 page (abstract, transcript, and code samples)._
