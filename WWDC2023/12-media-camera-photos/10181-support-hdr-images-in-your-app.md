# Support HDR images in your app
**WWDC23 · Session 10181** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10181/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, tvOS 17

## Overview
This session covers the complete pipeline for High Dynamic Range (HDR) still images in Apple frameworks. It introduces the new ISO technical specification TS22028-5 for encoding HDR content into existing still image formats (HEIF, PNG, TIFF) and explains how to use new and existing APIs across SwiftUI, UIKit, AppKit, Core Image, CoreGraphics, and CALayer to identify, load, display, create, and write ISO HDR images. A second HDR type — Gain Map HDR — has been captured by iPhone since 2020 and is now also accessible via new API that reconstructs the HDR representation from the embedded gain map.

The key design principle is that HDR content flows through an unclamped, color-space-aware pipeline. Deprecated APIs (e.g., `UIGraphicsBeginImageContextWithOptions`) silently clamp or discard HDR data; the session highlights safe replacement APIs and pixel formats for each stage of the pipeline.

## Key Topics

### HDR Concepts: EDR and Headroom
Extended Dynamic Range (EDR) is Apple's framework-level representation of HDR content. Reference white = 1.0; display peak = maximum the hardware supports. "Headroom" is the ratio of peak to reference white. Current iPhone 14 supports up to 8× headroom; 12.9" iPad Pro and MacBook Pro support up to 16×; Pro Display XDR up to 400×. Most other Apple displays support up to 2×.

### ISO HDR (TS22028-5) Image Format
Requirements:
- **Transfer function:** HLG (Hybrid Log-Gamma) or PQ (Perceptual Quantizer)
- **Color primaries:** BT.2020 (wide gamut)
- **Bit depth:** 10 bits per component minimum (HEIF capable; traditional JPEG is not)
- **Metadata:** ICC profile or CICP tags; optional diffuse white luminance (default 203 cd/m²), reference environment, scene-referred flag, mastering/content color volume, content light level

### Gain Map HDR
iPhone images since 2020 contain an embedded gain map that reconstructs HDR from the SDR image. New in iOS 17: `UIImageReader` and `CIImage`/`CGImageSource` options can expand Gain Map HDR to its full HDR representation.

### Display Dynamic Range in SwiftUI and UIKit
Three dynamic range modes, applicable to SwiftUI `Image`, `UIImageView`, and `NSImageView`:
- `.high` — full HDR, system tone-maps to current display capabilities
- `.standard` — tone-map everything to SDR
- `.constrainedHigh` — limited headroom; useful for thumbnail grids where SDR images shouldn't look dim next to HDR

### HDR-Safe Image Resizing
- **Deprecated (not HDR-safe):** `UIGraphicsBeginImageContextWithOptions`
- **Safe alternatives:**
  - `UIImage.prepareThumbnail(of:completionHandler:)` (iOS 15+) — preferred for thumbnails
  - `UIGraphicsImageRenderer` with `imageRendererFormat` from the source image

### Reading and Identifying HDR Images
- `UIImage` and `NSImage` automatically load ISO HDR; `UIImage.isHighDynamicRange` property detects ISO HDR
- `CGColorSpaceUsesITUR_2100TF(_:)` — returns `true` for ISO HDR color spaces (use with CoreGraphics/Core Image/AppKit)
- `CIImage(contentsOf:)` — automatically reads ISO HDR with correct color space conversion recipe
- `CGImageSourceCreateImageAtIndex(_:_:_:)` with `decodeRequest: .decodeToHDR` — ISO HDR; `.decodeToSDR` — tone-map to SDR

### Gain Map HDR Access via Core Image
- `CIImage(contentsOf:options:[.expandToHDR: true])` — reconstructs HDR from gain map; color space becomes HDR
- `CIImage(contentsOf:options:[.toneMapHDRtoSDR: true])` — forces SDR rendering (useful for feature detection)
- `CIRAWFilter` — `extendedDynamicRangeAmount: Float` (0=SDR, 1=max HDR from RAW file)

### Writing ISO HDR Files
- `UIImage.pngData()` / `UIImage.heifData()` — automatically write 16-bit PNG or 10-bit HEIF when the image contains HDR content **[NEW behavior]**
- `CIContext.writePNGRepresentation(of:to:format:colorSpace:)` with `RGBA16` format + HDR color space
- `CIContext.writeTIFFRepresentation(of:to:format:colorSpace:)` with `RGBA16` — lossless, large files
- **Best practice:** `CIContext.writeHEIF10Representation(of:to:colorSpace:)` — lossy, smallest ISO HDR output

### CALayer HDR Rendering
- `CALayer.wantsExtendedDynamicRangeContent = true` **[NEW]** — enables HDR rendering with system tone-mapping when the display headroom is insufficient
- `CAMetalLayer` — no automatic tone-mapping; values above display peak are clamped (use for custom tone-mapping pipelines)
- Supported `CALayer` content types for HDR: `CGImage`, `CVPixelBuffer`, `IOSurface` tagged with ISO HDR color space

### CVPixelBuffer / IOSurface HDR Pipeline
- Create with 10-bit biplanar full-range pixel format + `IOSurfacePropertiesKey`
- Attach ISO HDR color space metadata (`CVBufferSetAttachment`)
- Then wrap in `CIImage(cvPixelBuffer:)` for Core Image processing

## APIs & Frameworks

- **SwiftUI**
  - `Image.allowedDynamicRange(_:)` modifier **[NEW]** — `.high`, `.constrainedHigh`, `.standard`
- **UIKit**
  - `UIImageView.preferredImageDynamicRange` property **[NEW]** — `.high`, `.constrainedHigh`, `.standard`
  - `UIImage.isHighDynamicRange: Bool` property **[NEW]** — `true` for ISO HDR content
  - `UIImage.pngData()` / `UIImage.heifData()` — automatically produce ISO HDR output when image is HDR **[NEW behavior]**
  - `UIGraphicsImageRenderer` with `imageRendererFormat` — HDR-safe resizing
  - `UIImage.prepareThumbnail(of:completionHandler:)` (iOS 15) — HDR-safe thumbnail
  - `UIImageReader` **[NEW]** — reads Gain Map HDR images; returns HDR representation on HDR displays by default
    - `UIImageReader.Configuration` — set `prefersHighDynamicRange = true`
- **AppKit**
  - `NSImageView` — same `preferredImageDynamicRange` property as UIKit **[NEW]**
- **Core Image**
  - `CIImage(contentsOf:options:)` — automatic ISO HDR support; options:
    - `.expandToHDR` **[NEW]** — reconstruct Gain Map HDR or expand RAW to HDR
    - `.toneMapHDRtoSDR` **[NEW]** — force SDR tone-mapping
  - `CIRAWFilter.extendedDynamicRangeAmount: Float` — 0.0–1.0, controls RAW HDR output **[NEW]**
  - `CIContext.writeHEIF10Representation(of:to:colorSpace:options:)` — 10-bit HEIF ISO HDR output
  - `CIContext.writePNGRepresentation(of:to:format:colorSpace:options:)` — 16-bit PNG ISO HDR
  - `CIContext.writeTIFFRepresentation(of:to:format:colorSpace:options:)` — 16-bit TIFF ISO HDR
  - `CIFormat.RGB10` **[NEW]** — 10-bit RGB format; half the memory of RGBA16 for CGImage conversion
  - 150+ built-in filters support HDR natively; check `kCIAttributeFilterCategories` for `kCICategoryHighDynamicRange`
- **CoreGraphics**
  - `CGColorSpaceUsesITUR_2100TF(_:) -> Bool` — identifies ISO HDR color spaces
  - `CGImageSourceCreateImageAtIndex(_:_:_:)` with `kCGImageSourceDecodeRequest`:
    - `kCGImageSourceDecodeToHDR` **[NEW]** — load as HDR
    - `kCGImageSourceDecodeToSDR` **[NEW]** — load tone-mapped to SDR
  - ISO HDR-safe `CGBitmapInfo` flags: float (32-bit), half float (16-bit), 16-bit integer, 10-bit RGB
- **CALayer / QuartzCore**
  - `CALayer.wantsExtendedDynamicRangeContent: Bool` **[NEW]** — enable HDR with tone-mapping
  - `CAMetalLayer.wantsExtendedDynamicRangeContent` — existing property; no tone-mapping
- **PhotosUI / PhotoKit**
  - `PHPickerViewController` — use `.current` encoding policy + `.images` type filter to prevent SDR transcoding
  - `PHImageManager` — access ISO HDR via `requestAVAsset` or data representation
- **UIKit (display query)**
  - `UIScreen.potentialEDRHeadroom` — current display's HDR headroom capability (iOS/iPadOS)
  - `NSScreen.maximumPotentialExtendedDynamicRangeColorComponentValue` — macOS equivalent

## Code Highlights

Display HDR in SwiftUI and UIKit:
```swift
// SwiftUI
Image(url: hdrImageURL)
    .allowedDynamicRange(.high)

// UIKit
let imageView = UIImageView(image: UIImage(contentsOfFile: path))
imageView.preferredImageDynamicRange = .high
```

Load Gain Map HDR from Photos with UIImageReader:
```swift
var config = UIImageReader.Configuration()
config.prefersHighDynamicRange = true
let reader = UIImageReader(configuration: config)
let hdrImage = reader.image(data: photoData)
```

Expand Gain Map HDR in Core Image:
```swift
let ciImage = CIImage(contentsOf: url, options: [.expandToHDR: true])
// ciImage.colorSpace will be an HDR color space when gain map is available
```

Write ISO HDR HEIF with Core Image:
```swift
let context = CIContext()
try context.writeHEIF10Representation(of: ciImage, to: outputURL, colorSpace: CGColorSpace(name: CGColorSpace.itur_2100_HLG)!)
```

Enable HDR on a CALayer:
```swift
let layer = CALayer()
layer.wantsExtendedDynamicRangeContent = true
layer.contents = hdrCGImage  // must be ISO HDR tagged
```

## Takeaways

- SwiftUI `.allowedDynamicRange(.high)` and `UIImageView.preferredImageDynamicRange = .high` are the simplest entry points — they work transparently with SDR images and upgrade to HDR when the image and display support it.
- Use `.constrainedHigh` for thumbnail grids and contexts where SDR and HDR images appear side by side, preventing the SDR images from looking dim by comparison.
- Access Gain Map HDR (available in trillions of iPhone photos since 2020) with `CIImage(contentsOf:options:[.expandToHDR: true])` or `UIImageReader` with `prefersHighDynamicRange = true` — no special capture required.
- Never pass HDR images through deprecated APIs (`UIGraphicsBeginImageContextWithOptions`); use `UIGraphicsImageRenderer` with `imageRendererFormat` or `UIImage.prepareThumbnail` to preserve HDR data throughout the pipeline.

---
_Source: WWDC23 Session 10181 page (abstract, transcript, and resource links)._
