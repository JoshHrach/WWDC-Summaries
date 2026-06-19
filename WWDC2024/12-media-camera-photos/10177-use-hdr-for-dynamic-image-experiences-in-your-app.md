# Use HDR for Dynamic Image Experiences in Your App
**WWDC24 · Session 10177** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10177/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia 15, visionOS 2, watchOS 11, tvOS 18

## Overview
This session continues the WWDC23 "Support HDR Images in Your App" session, introducing the new Adaptive HDR image format and its corresponding APIs. Adaptive HDR builds upon ISO HDR (introduced in 2023) by embedding a Gain Map alongside a fully backward-compatible SDR baseline in a single file, enabling seamless tone mapping for any display headroom without requiring changes to SDR-only apps or decoders.

Apple drove the standardization of the Gain Map mathematical formula (encoding the log-ratio between HDR and SDR signals) through ISO, currently at Committee Draft stage. iPhone 15 and 15 Pro running iOS 18 capture photos in the new Adaptive HDR HEIC format. The session covers the conceptual framework (headroom, tone mapping, display adjustment reasons), then practical API guidance for reading, editing, displaying (three strategies), and saving Adaptive HDR images.

System apps updated to use these APIs in iOS 18/macOS Sequoia 15: Messages, Quick Look, Preview, and Photos.

## Key Topics

### HDR Concepts: Headroom and Tone Mapping
"Content Headroom" is the ratio of the image's peak brightness to SDR reference white (a page of paper indoors). "Display Headroom" is the display's current ability to render above reference white. Tone mapping adapts image brightness to fit available display headroom. Display headroom can be reduced by hardware limits, screen brightness settings, low battery mode, or app coexistence (OS promotes foreground image to HDR, tone maps background images to SDR).

### Adaptive HDR
Adaptive HDR files contain: (1) an SDR baseline image (fully decodable by older SDR apps), (2) a Gain Map (encodes log ratio of HDR/SDR per pixel, optionally RGB 3-channel), (3) new metadata. Applying the Gain Map to the SDR baseline produces the HDR output. Any intermediate display headroom is achieved by partially applying the Gain Map (weight < 1). Files have one image per `CGImageSourceGetCount` (returns 1); requesting the alternate (TMAP) representation activates the Gain Map and metadata.

### New Reference White Tone Mapping Operator
For ISO HDR files (no Gain Map), iOS 18/macOS Sequoia 15 introduces a new Apple Reference White Tone Mapping Operator, replacing ITU-standard global tone mappers. It better preserves highlight detail and color accuracy. Adaptive HDR files use a custom per-file tone map curve derived from the Gain Map instead of the global operator.

### Reading HDR Images
Load as SDR by default (no options). Load as HDR with `CIImageOption.expandToHDR` or `kCGImageSourceDecodeToHDR`. Load the Gain Map separately with `CIImageOption.auxiliaryHDRGainMap`. Query content headroom with `CIImage.contentHeadroom` (new property), `CGImageGetContentHeadroom()`, or `IOSurface` equivalent. Value 1 = SDR; > 1 = HDR (up to 8); 0 = unknown.

### Edit Strategies
Three approaches: (1) **HDR strategy** — load with `expandToHDR`, edit with HDR-compatible CIFilters, use `toneMapHeadroomFilter` for display; simpler but Gain Map lost; (2) **SDR + Gain strategy** — load SDR and Gain Map separately, apply paired edits (accounting for Gain Map's typically half resolution), combine with `imageByApplyingGainMap:headroom:` for display; preserves Gain Map for best backward-compatible saving; (3) **SDR + HDR strategy** — load both, edit both in parallel, recalculate Gain Map before saving; most flexible but highest complexity.

### Display Tone Mapping
`UIImageView.preferredImageDynamicRange` and SwiftUI's `.allowedDynamicRange` modifier handle tone mapping automatically for `UIImage`/`Image` views. For Core Image / Metal: use new `CIToneMapHeadroom` filter (applies Reference White operator for ISO HDR, per-file curve for Adaptive HDR) passing current display headroom queried from the `MTKView`; or use `CIImage.imageByApplyingGainMap(_:headroom:)` for SDR+Gain strategy.

## APIs & Frameworks

- `CIImage` — Core Image image type
- `CIImage.contentHeadroom` **[NEW]** — Float property; content headroom (1 = SDR, >1 = HDR, 0 = unknown)
- `CIImageOption.expandToHDR` **[NEW in iOS 18]** — loads Adaptive HDR files in full HDR representation
- `CIImageOption.auxiliaryHDRGainMap` **[NEW]** — loads the Gain Map as a separate `CIImage`
- `CIImage.imageByApplyingGainMap(_:headroom:)` **[NEW]** — combines SDR image and Gain Map for a target display headroom
- `CIToneMapHeadroom` filter **[NEW]** — tone maps an HDR CIImage to a target display headroom; uses Reference White operator for ISO HDR, per-file curve for Adaptive HDR
- `CIRenderDestination` — existing API for rendering CIImages to Metal textures
- `CGImageGetContentHeadroom(_:)` **[NEW]** — returns content headroom for a `CGImageRef`
- `CGContextSetEDRTargetHeadroom(_:_:)` **[NEW]** — sets target headroom for a `CGBitmapContext` in extended range mode
- `kCGImageSourceDecodeToHDR` **[NEW]** — ImageIO option to decode image as HDR
- `kCGImageAuxiliaryDataTypeISOGainMap` **[NEW]** — ImageIO auxiliary data type key for Gain Map data
- `CGImageDestinationAddAuxiliaryDataInfo(_:_:_:)` — saves auxiliary Gain Map data alongside an SDR CGImage
- `CGImageDestinationAddImage(_:_:_:)` — saves CGImage (used with SDR colorspace for Adaptive HDR SDR baseline)
- `CIContext.writeHEIFRepresentation(of:to:format:colorSpace:options:)` — extended with:
  - `CIImageRepresentationOption.hdrImage` **[NEW]** — HDR image for SDR+HDR save strategy
  - `CIImageRepresentationOption.hdrGainMapImage` **[NEW]** — Gain Map for SDR+Gain save strategy
- `UIImage`, `UIImageReader` — read HDR file formats
- `UIImageView.preferredImageDynamicRange` **[NEW]** — controls how much dynamic range to display
- `.allowedDynamicRange(_:)` **[NEW]** — SwiftUI modifier on `Image` for dynamic range control
- `MTKView` — Metal view for EDR rendering
- `IOSurface` — content headroom queryable via new property
- `CVPixelBuffer` — headroom queryable by creating CIImage and reading `contentHeadroom`
- ISO HDR — existing standard (2023); HEIF, AVIF, PNG, JPEG XL, etc.
- Adaptive HDR — **[NEW]** ISO Committee Draft; Gain Map embedded in HEIF or JPEG
- Apple Gain Map — pre-existing format (since 2020 iPhone cameras); now upgraded to Adaptive HDR spec
- `CGImageSource` — ImageIO source; `CGImageSourceGetCount` returns 1 for Adaptive HDR files (TMAP alternate is not a separate top-level image)

## Code Highlights

Load Adaptive HDR image as HDR with CIImage:
```swift
let hdrImage = CIImage(contentsOf: url, options: [.expandToHDR: true])
let headroom = hdrImage?.contentHeadroom ?? 1.0
```

Load Gain Map separately:
```swift
let sdrImage = CIImage(contentsOf: url)
let gainMap = CIImage(contentsOf: url, options: [.auxiliaryHDRGainMap: true])
```

Display with automatic tone mapping in SwiftUI:
```swift
let uiImage = UIImageReader.default.image(contentsOf: url)!
Image(uiImage: uiImage)
    .allowedDynamicRange(.high)
```

Apply tone mapping in Core Image + Metal pipeline:
```swift
let displayHeadroom = view.edrMetadata?.maximumEDR ?? 1.0
let toneMapped = hdrImage.applyingFilter("CIToneMapHeadroom",
    parameters: ["inputTargetHeadroom": displayHeadroom])
```

Combine SDR + Gain Map for display:
```swift
let combined = sdrImage.imageByApplyingGainMap(gainMap, headroom: displayHeadroom)
```

Save as Adaptive HDR from SDR + HDR:
```swift
try context.writeHEIFRepresentation(of: sdrImage, to: url,
    format: .RGBA8, colorSpace: sdrColorSpace,
    options: [.hdrImage: hdrImage])
```

## Takeaways

- Adaptive HDR embeds both an SDR baseline and a Gain Map in a single file, making it fully backward compatible with SDR apps while enabling apps that opt in to display full HDR with optimal tone mapping for any display headroom.
- Use `CIImageOption.expandToHDR` or `UIImageReader` + `.allowedDynamicRange(.high)` to get HDR with minimal code; `UIImageView.preferredImageDynamicRange` and SwiftUI's `.allowedDynamicRange` handle tone mapping automatically.
- Three editing strategies balance simplicity (HDR-only), backward compatibility (SDR+Gain), and flexibility (SDR+HDR); choose based on your app's filter requirements and saving needs.
- The new `CIImage.contentHeadroom` property and `CGImageGetContentHeadroom()` API enable correct tone mapping decisions throughout the image pipeline.

---
_Source: WWDC24 Session 10177 page (abstract, chapter summaries, transcript, and resource links)._
