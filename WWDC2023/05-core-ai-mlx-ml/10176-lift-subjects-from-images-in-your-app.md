# Lift Subjects from Images in Your App
**WWDC23 · Session 10176** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10176/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14

## Overview
This session introduces developer APIs for subject lifting — the ability to detect and extract foreground subjects from images. The feature that debuted as a system-level user interaction in iOS 16 (long-press to lift a subject) is now available to third-party developers via two frameworks: VisionKit for easy, out-of-process UI integration, and Vision for a more flexible, lower-level, in-process approach that integrates with compositing pipelines like Core Image.

VisionKit's `ImageAnalysisInteraction` adds system-like subject lifting with a few lines of code. The new Vision `VNForegroundInstanceMaskRequest` provides class-agnostic foreground segmentation: it segments any foreground object regardless of semantic class (people, animals, objects, food, etc.), returning per-instance soft masks at full image resolution. These masks are directly usable with Core Image filters for HDR-preserving compositing.

## Key Topics

### What Is a Subject?
- Any foreground object(s) in a photo — not limited to people or animals; can be food, buildings, shoes, etc.
- Images can have multiple subjects; subjects can be combined (e.g., a person and their dog as one subject).
- Instance: a single distinct segmented object.
- Background instance always has index 0; foreground instances labeled 1, 2, 3, … (ordering not guaranteed).

### VisionKit Approach (Recommended for UI)
- `ImageAnalysisInteraction` (iOS/iPadOS) or `ImageAnalysisOverlayView` (macOS) — add to a view to enable system subject lifting interactions.
- `preferredInteractionTypes` — set to `.imageSubject` for lifting only, or `.automatic` for subject lifting + Live Text + data detectors.
- `ImageAnalyzer` + `analyze(_:configuration:)` — generates an `ImageAnalysis` asynchronously.
- `ImageAnalysis.subjects` — async property returning all detected subjects as an array of `Subject` objects.
- `Subject.image` — the cropped image of the subject.
- `Subject.bounds` — bounding rect of the subject.
- `ImageAnalysis.highlightedSubjects` — the currently highlighted (selected) subjects; writable to change selection programmatically.
- `ImageAnalysis.subject(at:)` — async method returning the subject at a given point (useful for hit testing).
- `ImageAnalysis.image(for:)` — async method compositing multiple subjects into one image.
- Runs out-of-process; image size is limited; no direct Core Image integration.

### Vision Approach (Advanced / Compositing)
- `VNForegroundInstanceMaskRequest` **[NEW]** — class-agnostic foreground segmentation request.
- `VNImageRequestHandler` — wraps the input image for processing.
- Results: array with a single `VNInstanceMaskObservation` if subjects are detected.
- `VNInstanceMaskObservation.allInstances` — `IndexSet` of all foreground instance indices.
- `VNInstanceMaskObservation.generateMaskedImage(ofInstances:from:croppedToInstancesExtent:)` — generates a masked `CVPixelBuffer` for selected instances.
- `VNInstanceMaskObservation.generateScaledMaskForImage(forInstances:from:)` (or `createScaledMask`) — generates a single-channel float pixel buffer (soft mask) for use with Core Image.
- `croppedToInstancesExtent` parameter — if `true`, tight-crops the output; if `false`, preserves input image dimensions.
- Runs in-process; no image resolution limit; suitable for high-resolution and HDR workflows.

### Compositing with Core Image
- `CIFilter.blendWithMask()` (`CIBlendWithMask`) — composites subject over a new background using the Vision-generated mask.
- Inputs: source image, mask from Vision, background image (empty = transparent).
- Preserves HDR information of source image.
- Can be combined with other CIFilters: `CIExposureAdjust` (dim background), bokeh blur, etc.

### Instance Label Lookup for Hit Testing
- `VNInstanceMaskObservation` provides a pixel buffer mapping pixels to instance indices (UInt8 values).
- Instance 0 = background.
- Tap position from UIKit (normalized 0–1, origin top-left) maps to pixel buffer coordinates.
- Use Vision's coordinate transform helper to convert between UIKit and image space.

## APIs & Frameworks
- `VisionKit` framework — high-level subject lifting with system UI
- `ImageAnalysisInteraction` **[NEW]** (iOS/iPadOS) — adds subject lifting interactions to a UIView
- `ImageAnalysisOverlayView` **[NEW]** (macOS) — adds subject lifting interactions to an NSView
- `ImageAnalysisInteraction.preferredInteractionTypes` — `.imageSubject`, `.automatic`
- `VisualLookUpType.imageSubject` **[NEW]** — interaction type for subject lifting only
- `ImageAnalyzer` — performs image analysis asynchronously
- `ImageAnalyzer.analyze(_:configuration:)` — generates `ImageAnalysis`
- `ImageAnalyzerConfiguration` — specifies analysis types
- `ImageAnalysis.subjects` **[NEW]** — async array of detected subjects
- `ImageAnalysis.Subject` **[NEW]** — struct with `image` and `bounds` properties
- `ImageAnalysis.highlightedSubjects` **[NEW]** — writable set of highlighted subjects
- `ImageAnalysis.subject(at:)` **[NEW]** — async hit test returning subject at a point
- `ImageAnalysis.image(for:)` **[NEW]** — async composite of multiple subjects
- `Vision` framework — low-level ML vision analysis
- `VNForegroundInstanceMaskRequest` **[NEW]** — class-agnostic instance segmentation request
- `VNInstanceMaskObservation` **[NEW]** — observation containing segmentation results
- `VNInstanceMaskObservation.allInstances` **[NEW]** — `IndexSet` of all foreground instance indices
- `VNInstanceMaskObservation.generateMaskedImage(ofInstances:from:croppedToInstancesExtent:)` **[NEW]** — generates masked pixel buffer
- `VNInstanceMaskObservation.generateScaledMaskForImage(forInstances:from:)` **[NEW]** — generates soft segmentation mask pixel buffer
- `VNImageRequestHandler` — wraps image for Vision request processing
- `Core Image` / `CIFilter` — image compositing and effects
- `CIBlendWithMask` (CIFilter) — composites subject over background using soft mask
- `CIExposureAdjust` — exposure adjustment for background dimming effect
- `CVPixelBuffer` — raw pixel data representation for masks

## Code Highlights

VisionKit subject lifting in a UIImageView:
```swift
let interaction = ImageAnalysisInteraction()
interaction.preferredInteractionTypes = .imageSubject
imageView.addInteraction(interaction)
```

Getting all subjects from an analysis:
```swift
let analyzer = ImageAnalyzer()
let config = ImageAnalyzer.Configuration([.visualLookUp])
let analysis = try await analyzer.analyze(image, configuration: config)
let subjects = await analysis.subjects
```

Vision foreground instance mask request:
```swift
let request = VNForegroundInstanceMaskRequest()
let handler = VNImageRequestHandler(cgImage: inputImage)
try handler.perform([request])

if let observation = request.results?.first {
    let maskedImage = try observation.generateMaskedImage(
        ofInstances: observation.allInstances,
        from: handler,
        croppedToInstancesExtent: true
    )
}
```

Core Image compositing with Vision mask:
```swift
let mask = try observation.generateScaledMaskForImage(
    forInstances: selectedInstances, from: handler)
let maskImage = CIImage(cvPixelBuffer: mask)

let blendFilter = CIFilter.blendWithMask()
blendFilter.inputImage = sourceImage
blendFilter.maskImage = maskImage
blendFilter.backgroundImage = newBackgroundImage // or empty CIImage() for transparent
let result = blendFilter.outputImage
```

## Takeaways
- VisionKit's `ImageAnalysisInteraction` is the fastest path to system-standard subject lifting UI; it requires only a few lines of code and handles all interaction automatically.
- Vision's `VNForegroundInstanceMaskRequest` is class-agnostic — it segments any foreground object, not just people — making it broadly applicable for creative apps.
- Use the `generateScaledMaskForImage` soft mask with Core Image's `CIBlendWithMask` filter to preserve HDR data while compositing lifted subjects over new backgrounds.
- Instance label lookup (from the pixel buffer) enables interactive single-subject selection by mapping tap positions to instance indices without re-running the Vision request.

---
_Source: WWDC23 Session 10176 page (abstract, chapter summaries, code samples, and resource links)._
