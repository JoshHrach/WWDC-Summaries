# Capture and Process ProRAW Images
**WWDC21 · Session 10160** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10160/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12

## Overview
Apple ProRAW is a hybrid image format introduced on iPhone 12 Pro that combines the computational photography pipeline (Smart HDR, Deep Fusion, Night mode) with the editability of RAW. ProRAW files are standard Adobe DNG files that store linearized, demosaiced, losslessly compressed 12-bit RGB data with up to 14 stops of dynamic range, along with DNG tags that encode Apple's rendering recipe and optional semantic segmentation masks (person, skin, sky).

This session covers the full ProRAW production pipeline: capturing with AVFoundation, storage and retrieval via PhotoKit, and editing and display using Core Image. A new Swift-friendly `CIRAWFilter` API (iOS 15 / macOS 12) simplifies RAW image processing and exposes named properties for exposure, white balance, sharpness, local tone mapping, and Extended Dynamic Range output.

## Key Topics

### ProRAW Format
- Stored in standard Adobe DNG (linearized DNG variant)
- Losslessly compressed 12-bit RGB with companding curve; effective 14 stops of dynamic range
- DNG tags encode rendering recipe: `LinearizationTable`, `BaselineExposure`, `BaselineSharpness`, `ProfileGainTableMap` (DNG 1.6), `ProfileToneCurve`
- Contains full-resolution JPEG-quality prerendered preview
- Optional semantic segmentation masks (person, skin, sky) embedded when scene applicable
- File sizes: 10–40 MB, scene-dependent
- Supported devices: iPhone 12 Pro and later (at time of session)

### Capture (AVFoundation)
- Requires `.photo` session preset or format with `isHighestPhotoQualitySupported`
- Must enable `isAppleProRAWEnabled` on `AVCapturePhotoOutput` before session start
- Select ProRAW pixel format using `isAppleProRAWPixelFormat(_:)` class method
- Supports `.balanced` and `.quality` photo quality prioritization (enabling image fusion)
- Supports all device types including multi-camera virtual devices
- Can capture ProRAW-only, ProRAW + processed pair, or ProRAW with embedded JPEG thumbnail
- Custom compression via `AVCapturePhotoFileDataRepresentationCustomizer` — adjust bit depth (8–12 bit lossless, or lossy <1.0 quality)

### Storage and Retrieval (PhotoKit)
- Save with `PHAssetCreationRequest` using `.photo` resource type
- New `PHAssetCollectionSubtype.smartAlbumRAW` (iOS 15) fetches all RAW assets
- Retrieve RAW data via `PHAssetResource` checking `.photo` or `.alternatePhoto` type conforming to `UTType.rawImage`

### Editing and Display (Core Image)
- New `CIRAWFilter` class (iOS 15 / macOS 12) with named Swift properties replaces `CIFilter(imageURL:options:)` pattern
- Key adjustable properties: `exposure`, `neutralTemperature`, `neutralTint`, `sharpnessAmount`, `localToneMapAmount`
- Linear scene-referred output: set `baselineExposure`, `shadowBias`, `boostAmount`, `localToneMapAmount` to 0, disable `isGamutMappingEnabled`
- Save to 8-bit HEIC (`writeHEIFRepresentation`) or 10-bit HEIC (`writeHEIF10Representation`) (new in iOS 15)
- EDR display on Mac: set `extendedDynamicRangeAmount = 1.0`, render to `MTKView` with `rgba16Float` pixel format and `wantsExtendedDynamicRangeContent = true`

## APIs & Frameworks

**AVFoundation**
- `AVCapturePhotoOutput.isAppleProRAWSupported` — checks device/format support
- `AVCapturePhotoOutput.isAppleProRAWEnabled` **[NEW]** — enable ProRAW capture pipeline
- `AVCapturePhotoOutput.isAppleProRAWPixelFormat(_:)` **[NEW]** — identifies ProRAW pixel format
- `AVCapturePhotoOutput.isBayerRAWPixelFormat(_:)` — identifies Bayer RAW pixel format
- `AVCapturePhotoOutput.maxPhotoQualityPrioritization` — `.speed`, `.balanced`, `.quality`
- `AVCapturePhotoSettings(rawPixelFormatType:)` — ProRAW-only capture
- `AVCapturePhotoSettings(rawPixelFormatType:processedFormat:)` — ProRAW + processed pair
- `AVCapturePhotoSettings.rawEmbeddedThumbnailPhotoFormat` — specify embedded JPEG thumbnail
- `AVCapturePhotoSettings.availableRawEmbeddedThumbnailPhotoCodecTypes` **[NEW]**
- `AVCapturePhoto.isRawPhoto` — distinguishes ProRAW from processed in delegate
- `AVCapturePhoto.fileDataRepresentation()` — DNG file data
- `AVCapturePhoto.fileDataRepresentation(with:)` — customized DNG via `AVCapturePhotoFileDataRepresentationCustomizer`
- `AVCapturePhoto.pixelBuffer` — raw pixel buffer
- `AVCapturePhotoFileDataRepresentationCustomizer` protocol
  - `replacementAppleProRAWCompressionSettings(for:defaultSettings:maximumBitDepth:)` **[NEW]**
- `AVVideoAppleProRAWBitDepthKey` **[NEW]** — bit depth key for compression settings
- `AVCaptureDevice.Format.isHighestPhotoQualitySupported` — formats supporting ProRAW

**PhotoKit**
- `PHAssetCollectionSubtype.smartAlbumRAW` **[NEW iOS 15]** — RAW smart album
- `PHAssetCreationRequest.addResource(with:fileURL:options:)` — save ProRAW DNG
- `PHAssetResource` — `.photo`, `.alternatePhoto` resource types
- `PHAssetResource.uniformTypeIdentifier` — check conformance to `UTType.rawImage`
- `PHAssetResourceManager.default().requestData(for:options:dataReceivedHandler:completionHandler:)`

**Core Image** (`import CoreImage`)
- `CIRAWFilter` **[NEW iOS 15 / macOS 12]** — Swift-friendly RAW filter class
  - `CIRAWFilter(imageURL:)` — create from file URL
  - `CIRAWFilter(imageData:identifierHint:)` — create from data
  - `.previewImage` — access embedded preview **[NEW]**
  - `.semanticSegmentationSkinMatte` — skin mask image **[NEW]**
  - `.semanticSegmentationSkyMatte` — sky mask image **[NEW]**
  - `.semanticSegmentationHairMatte` — hair mask image **[NEW]**
  - `.outputImage` — render with current settings
  - `.exposure` — EV adjustment (float)
  - `.neutralTemperature` — white balance in Kelvin
  - `.neutralTint` — white balance tint
  - `.sharpnessAmount` — 0.0–1.0
  - `.localToneMapAmount` — 0.0–1.0
  - `.baselineExposure` — baseline exposure offset
  - `.shadowBias` — shadow lift
  - `.boostAmount` — overall boost
  - `.isGamutMappingEnabled` — toggle gamut mapping
  - `.extendedDynamicRangeAmount` — 0.0–1.0, for EDR display
- `CIContext.writeHEIFRepresentation(of:to:format:colorSpace:options:)` — save 8-bit HEIC
- `CIContext.writeHEIF10Representation(of:to:format:colorSpace:options:)` **[NEW]** — save 10-bit HEIC
- `CIRenderDestination(bitmapData:width:height:bytesPerRow:format:)` — render to buffer
- `CIImage(contentsOf:options:)` with `.auxiliarySemanticSegmentationSkinMatte` option

**ImageIO / CGImageSource**
- `CGImageSourceCreateWithURL(_:_:)` — open DNG
- `CGImageSourceCreateThumbnailAtIndex(_:_:_:)` — get preview thumbnail

**MetalKit / Metal**
- `MTKView` subclass for EDR rendering
- `MTLPixelFormat.rgba16Float` — pixel format for EDR
- `CAMetalLayer.wantsExtendedDynamicRangeContent` — enable EDR on Mac

## Code Highlights

Enable ProRAW and capture:
```swift
photoOutput.isAppleProRAWEnabled = photoOutput.isAppleProRAWSupported
photoOutput.maxPhotoQualityPrioritization = .quality

guard let proRawPixelFormat = photoOutput.availableRawPhotoPixelFormatTypes.first(
    where: { AVCapturePhotoOutput.isAppleProRAWPixelFormat($0) }) else { return }

let photoSettings = AVCapturePhotoSettings(rawPixelFormatType: proRawPixelFormat)
photoSettings.photoQualityPrioritization = .quality
photoOutput.capturePhoto(with: photoSettings, delegate: delegate)
```

Edit with CIRAWFilter (new API):
```swift
let rawFilter = CIRAWFilter(imageURL: url)
rawFilter.exposure = -1.0
rawFilter.neutralTemperature = 6500
rawFilter.localToneMapAmount = 0.5
let outputImage = rawFilter.outputImage
```

Save to 10-bit HEIC:
```swift
try ciContext.writeHEIF10Representation(of: rawFilter.outputImage!,
    to: theURL, format: .RGBA8,
    colorSpace: CGColorSpace(name: CGColorSpace.displayP3)!, options: [:])
```

## Takeaways
- ProRAW combines Apple's computational photography pipeline with RAW editability, stored as a standard DNG with up to 14 stops of dynamic range.
- The new `CIRAWFilter` class provides a Swift-friendly API replacing the old key-value `CIFilter` pattern for all RAW editing operations.
- `PHAssetCollectionSubtype.smartAlbumRAW` (new iOS 15) makes it easy to surface all RAW images from the user's photo library.
- ProRAW supports EDR display on compatible Mac displays, with `extendedDynamicRangeAmount = 1.0` enabling specular highlights beyond SDR tone mapping.

---
_Source: WWDC21 Session 10160 page (abstract, chapter summaries, code samples, and resource links)._
