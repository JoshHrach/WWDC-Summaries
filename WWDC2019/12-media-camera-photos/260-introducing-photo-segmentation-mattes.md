# Introducing Photo Segmentation Mattes
**WWDC19 · Session 260** · [Watch](https://developer.apple.com/videos/play/wwdc2019/260/)

_Platforms:_ iOS 13, iPadOS 13

## Overview
iOS 13 expands the Portrait Mode capture pipeline with three new Semantic Segmentation Mattes — hair, skin, and teeth — on top of the existing Portrait Effects Matte introduced in iOS 12. These per-pixel alpha mattes are generated on-device by Core ML models running on the Apple Neural Engine, and are embedded as auxiliary images into HEIF or JPEG files. Unlike the full-person Portrait Effects Matte, the semantic mattes isolate individual body regions, enabling creative effects such as hair colorization, skin retouching, and teeth whitening entirely within Core Image.

The session is split between two presenters: Jacob covers how the mattes are captured and represented via AVFoundation, and David covers how to use Core Image to read mattes from files, apply filters masked to matte regions, and write results back to HEIF/JPEG. The new `CIFilterBuiltins` header (200+ typed filter accessors) is introduced as a companion improvement that dramatically simplifies filter discovery and usage.

The AVCam sample app has been updated for iOS 13 to demonstrate the full capture lifecycle with Semantic Segmentation Mattes.

## Key Topics

**What Are Semantic Segmentation Mattes?**
Each matte is a grayscale (alpha-channel) image where pixel values indicate how much of a pixel belongs to the named semantic class: 1.0 = fully in the class, 0.0 = not in the class. Fractional values handle partial pixels and boundaries. Hair and skin mattes can overlap; skin and the full Portrait Effects Matte also overlap by design. The mattes are generated at half the size of the original image (quarter resolution) and must be scaled up before compositing with the main image.

**Capture Lifecycle via AVFoundation**
Opt into mattes at session configuration time by specifying a superset of the matte types you may ever request on `AVCapturePhotoOutput.enabledSemanticSegmentationMatteTypes`. Then, per-capture, specify the desired subset on `AVCapturePhotoSettings.enabledSemanticSegmentationMatteTypes`. The system checks available scene content; if no person is detected, matte dimensions will be zero (check this in `willBeginCaptureFor` callback). When photo processing finishes, retrieve each matte via `AVCapturePhoto.semanticSegmentationMatte(for:)`.

**AVSemanticSegmentationMatte**
A new class (similar structure to `AVPortraitEffectsMatte`) with:
- `matteType`: one of `.hair`, `.skin`, `.teeth` (or `.portraitEffects` from iOS 12).
- `pixelBuffer: CVPixelBuffer` — the raw alpha data.
- `dictionaryRepresentation(forAuxiliaryDataType:)` — for embedding in file I/O.
- `applyingExifOrientation(_:)` — rotate to match EXIF orientation.

**Reading from Files via Core Image**
Load the main RGB image with `CIImage(contentsOf:)`. Load each matte as an auxiliary image by passing an options dictionary with the appropriate key: `.auxiliarySegmentationHairMatte`, `.auxiliarySegmentationSkinMatte`, `.auxiliarySegmentationTeethMatte` (in addition to the existing `.auxiliaryPortraitEffectsMatte`). The resulting `CIImage` is at the matte's native (half-size) resolution and must be scaled before compositing.

**Applying Effects with Core Image**
Scale the matte to match the main image using `CIAffineTransform`. Use `CIBlendWithMask` to limit any effect to the matte region: background = original image, foreground = image with effect applied, mask = scaled matte. Examples from the demo:
- Hair colorization: hue-rotate the hair region only.
- Skin smoothing/makeup: desaturate/brighten skin region (clown white effect used for illustration).
- Teeth whitening: increase brightness in the teeth region.
- Combined mattes: logical operations in Core Image to create synthetic mattes (e.g., eyes + mouth from skin + teeth with region exclusion).

**CIFilterBuiltins (New in iOS 13)**
Import `CoreImage.CIFilterBuiltins` to access typed Swift-friendly factory methods for all 200+ built-in Core Image filters without remembering string names or `NSNumber`-wrapped parameters. Properties are strongly typed (e.g., `filter.inputPower: Float` instead of `setValue(NSNumber(...), forKey: "inputPower")`).

**Saving with Auxiliary Mattes**
Save to HEIF using `CIContext.writeHEIFRepresentation(of:to:format:colorSpace:options:)` with an options dictionary containing keys for each matte (supply as `CIImage` values), or supply an array of `AVSemanticSegmentationMatte` objects via the `AVSemanticSegmentationMattes` option key. Saved files can be used as input by Photos extensions or other apps for further processing.

## APIs & Frameworks

**AVFoundation** (iOS 13) **[NEW]**

Configuration:
- `AVCapturePhotoOutput.enabledSemanticSegmentationMatteTypes: [AVSemanticSegmentationMatte.MatteType]` **[NEW]**
- `AVCapturePhotoSettings.enabledSemanticSegmentationMatteTypes: [AVSemanticSegmentationMatte.MatteType]` **[NEW]**

Result retrieval:
- `AVCapturePhoto.semanticSegmentationMatte(for: AVSemanticSegmentationMatte.MatteType) -> AVSemanticSegmentationMatte?` **[NEW]**

Matte type:
- `AVSemanticSegmentationMatte` **[NEW]**
  - `AVSemanticSegmentationMatte.MatteType.hair` **[NEW]**
  - `AVSemanticSegmentationMatte.MatteType.skin` **[NEW]**
  - `AVSemanticSegmentationMatte.MatteType.teeth` **[NEW]**
  - (`.portraitEffects` — existing from iOS 12)
  - `AVSemanticSegmentationMatte.pixelBuffer: CVPixelBuffer` **[NEW]**
  - `AVSemanticSegmentationMatte.matteType: MatteType` **[NEW]**
  - `AVSemanticSegmentationMatte.applyingExifOrientation(_:)` **[NEW]**
  - `AVSemanticSegmentationMatte.dictionaryRepresentation(forAuxiliaryDataType:)` **[NEW]**

Delegate callbacks (existing lifecycle, now includes segmentation matte data):
- `AVCapturePhotoCaptureDelegate.photoOutput(_:willBeginCaptureFor:)` — check matte dimensions
- `AVCapturePhotoCaptureDelegate.photoOutput(_:didFinishProcessingPhoto:error:)` — retrieve mattes

**Core Image** (iOS 13) **[NEW or enhanced]**

Loading mattes from files:
- `CIImage(contentsOf:options:)` with options keys: **[NEW keys]**
  - `.auxiliarySegmentationHairMatte` **[NEW]**
  - `.auxiliarySegmentationSkinMatte` **[NEW]**
  - `.auxiliarySegmentationTeethMatte` **[NEW]**
  - `.auxiliaryPortraitEffectsMatte` (existing from iOS 12)

Creating from AVSemanticSegmentationMatte:
- `CIImage(avSemanticSegmentationMatte:)` **[NEW]** (or use `pixelBuffer`)

Scaling:
- `CIAffineTransform` / `CIImage.transformed(by:)` (existing, needed to scale half-size matte to full size)

Compositing:
- `CIBlendWithMask` (existing) — combine original, effect, matte

Writing with mattes:
- `CIContext.writeHEIFRepresentation(of:to:format:colorSpace:options:)` with keys: **[NEW options]**
  - `.semanticSegmentationSkinMatte: CIImage` **[NEW]**
  - `.semanticSegmentationHairMatte: CIImage` **[NEW]**
  - `.semanticSegmentationTeethMatte: CIImage` **[NEW]**
  - or `.avSemanticSegmentationMattes: [AVSemanticSegmentationMatte]` **[NEW]**

Filter discovery:
- `CoreImage.CIFilterBuiltins` — typed Swift accessors for all built-in filters **[NEW]**
  - Example: `CIFilter.maximumComponent()`, `CIFilter.gammaAdjust()`, `CIFilter.blendWithMask()`

## Code Highlights

Capturing semantic segmentation mattes:
```swift
// At session configuration
photoOutput.enabledSemanticSegmentationMatteTypes = [.hair, .skin, .teeth]

// At capture time
let settings = AVCapturePhotoSettings()
settings.enabledSemanticSegmentationMatteTypes = [.hair, .skin]
photoOutput.capturePhoto(with: settings, delegate: self)

// In delegate
func photoOutput(_ output: AVCapturePhotoOutput,
                 didFinishProcessingPhoto photo: AVCapturePhoto, error: Error?) {
    let skinMatte = photo.semanticSegmentationMatte(for: .skin)
    let hairMatte = photo.semanticSegmentationMatte(for: .hair)
}
```

Loading matte from HEIF file with Core Image:
```swift
let baseImage = CIImage(contentsOf: heifURL)!
let hairMatte = CIImage(contentsOf: heifURL, options: [.auxiliarySegmentationHairMatte: true])!
```

Applying an effect masked to the skin matte:
```swift
import CoreImage.CIFilterBuiltins

let baseImage = CIImage(contentsOf: url)!
let skinMatte = CIImage(contentsOf: url, options: [.auxiliarySegmentationSkinMatte: true])!

// Scale matte from half-size to full image size
let scaleX = baseImage.extent.width / skinMatte.extent.width
let scaleY = baseImage.extent.height / skinMatte.extent.height
let scaledMatte = skinMatte.transformed(by: CGAffineTransform(scaleX: scaleX, y: scaleY))

// Apply grayscale (clown makeup) effect to full image
let gray = CIFilter.maximumComponent()
gray.inputImage = baseImage
let gamma = CIFilter.gammaAdjust()
gamma.inputImage = gray.outputImage!
gamma.power = 0.5
let effectImage = gamma.outputImage!

// Blend: show effect only over skin region
let blend = CIFilter.blendWithMask()
blend.backgroundImage = baseImage
blend.inputImage = effectImage
blend.maskImage = scaledMatte
let result = blend.outputImage!
```

Saving with embedded mattes:
```swift
try context.writeHEIFRepresentation(of: result,
                                    to: outputURL,
                                    format: .RGBA8,
                                    colorSpace: CGColorSpaceCreateDeviceRGB(),
                                    options: [
                                        .semanticSegmentationSkinMatte: scaledSkinImage,
                                        .semanticSegmentationHairMatte: scaledHairImage
                                    ])
```

## Takeaways
- The three new Semantic Segmentation Mattes (hair, skin, teeth) unlock per-region photo editing that previously required custom ML pipelines — Apple's Neural Engine generates them automatically on any A12+ device capturing in Portrait Mode.
- Mattes are half-size (quarter resolution) relative to the main image — always scale them before compositing with `CIAffineTransform`.
- The new `CIFilterBuiltins` import makes Core Image filter composition strongly typed and discoverable without memorizing string-based filter names or `NSNumber` parameters.
- Mattes round-trip through HEIF/JPEG files via both Core Image options and `AVSemanticSegmentationMatte` array parameters, enabling Photos extension workflows that preserve all matte data for subsequent editing.

---
_Source: WWDC19 Session 260 page (transcript, chapter summaries, and resource links)._
