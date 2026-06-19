# Explore Computer Vision APIs
**WWDC20 · Session 10673** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10673/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, tvOS 14

## Overview
This session demonstrates how to combine Core Image, Vision, and Core ML to build powerful Computer Vision pipelines without relying on third-party libraries. The presenters, Frank Doepke and David Hayward, walk through using Core Image for preprocessing and post-processing while letting Vision perform the heavy analytical work, keeping everything in Apple's optimized image pipeline to avoid costly matrix conversions and memory overhead.

The session introduces two major new Vision APIs in 2020: Contour Detection and Optical Flow. Contour Detection finds edges and shapes in images and returns structured hierarchical results as CGPath objects, while Optical Flow provides per-pixel motion vectors between frames, enabling precise motion analysis that image registration alone cannot achieve.

A practical punch-card reading demo ties the concepts together, showing how a Python/OpenCV Computer Vision blog post can be translated into idiomatic Swift using UIKit, Core Image, and Vision — with no external dependencies and full platform optimization.

## Key Topics

**Core Image for Preprocessing**
- Downscaling with `CILanczosScaleTransform` or `CIAffineTransform` for best performance/quality tradeoff
- Morphology operations (Dilate, Erode, Close) to suppress noise before Vision requests
- Grayscale conversion via `CIColorMatrix` or `CIMaximumComponent`
- Noise reduction using `CIMedianFilter`, `CIGaussianBlur`, `CIBoxBlur`, `CINoiseReduction`
- Edge detection with `CIConvolution3X3` (Sobel) or `CIGaborGradients`
- Contrast enhancement via `CIColorPolynomial` and `CIColorControls`
- New thresholding filters: `CIColorThreshold` and `CIColorThresholdOtsu`
- New inter-frame comparison: `CIColorAbsoluteDifference` and `CILabDeltaE`
- Wide/HDR color space handling via `image.matchedFromWorkingSpace` / `matchedToWorkingSpace`

**Contour Detection**
- `VNDetectContoursRequest` returns a `VNContoursObservation` with hierarchical contours
- Tradeoff between `maximumImageDimension`, performance, and accuracy
- Polygon approximation via epsilon-based simplification to filter noise
- `VNGeometryUtils` for bounding circle, area, and perimeter calculations
- Aspect ratio correction for normalized coordinate spaces

**Optical Flow**
- `VNGenerateOpticalFlowRequest` is new in Vision 2020
- Returns `VNPixelBufferObservation` with floating-point interleaved X/Y motion per pixel
- Surpasses image registration for scenes with independent object motion
- Custom Core Image Metal kernel to visualize flow fields with color-coded direction arrows

**Post-Processing with Core Image**
- Regenerating barcode images from `VNBarcodeObservation.barcodeDescriptor`
- Applying vignette effects based on face observation coordinates
- Visualizing optical flow vector fields with custom CI kernels

## APIs & Frameworks

### Vision (new in 2020 unless noted)
- `VNDetectContoursRequest` **[NEW]** — finds edges/contours in images
- `VNContoursObservation` **[NEW]** — result type; properties: `topLevelContours`, `contourCount`, `normalizedPath`
- `VNContour` **[NEW]** — individual contour; properties: `childContours`, `indexPath`, `normalizedPoints`, `pointCount`, `aspectRatio`, `normalizedPath`
- `VNGeometryUtils` **[NEW]** — `boundingCircle`, area, perimeter calculations for contours
- `VNGenerateOpticalFlowRequest` **[NEW]** — per-pixel motion between two frames
- `VNPixelBufferObservation` — result of optical flow; floating-point pixel buffer with X/Y channels
- `VNSequenceRequestHandler` — performs requests across video frames
- `VNImageRequestHandler` — performs requests on a single image
- `VNDetectFaceRectanglesRequest` — existing face detection
- `VNRecognizeTextRequest` — existing text recognition
- `VNRectangleObservation` — existing rectangle result
- `VNBarcodeObservation` — existing barcode result; `barcodeDescriptor` used with Core Image

### Core Image (new filters in 2020)
- `CIColorThreshold` **[NEW]** — black/white conversion with explicit threshold
- `CIColorThresholdOtsu` **[NEW]** — auto threshold based on image histogram
- `CIColorAbsoluteDifference` **[NEW]** — pixel-level absolute difference between two images
- `CILanczosScaleTransform` — high-quality downscaling
- `CIAffineTransform` — linear interpolation resampling
- `CIMorphologyRectangleMaximum` — dilate (brighten small features)
- `CIMorphologyRectangleMinimum` — erode (shrink bright features)
- `CIColorMatrix` — custom RGB-to-grayscale weighting
- `CIMaximumComponent` — grayscale from dominant channel
- `CIMedianFilter` — noise reduction preserving edges
- `CIGaussianBlur` — fast noise reduction
- `CIBoxBlur` — fast noise reduction
- `CINoiseReduction` — dedicated noise filter
- `CIConvolution3X3` — Sobel edge detection
- `CIGaborGradients` — 2D gradient vector, noise-tolerant edge detection
- `CIColorPolynomial` — arbitrary 3rd-degree contrast function
- `CIColorControls` — linear contrast/brightness/saturation adjustment
- `CILabDeltaE` — perceptual color difference comparison
- `CIMeshGenerator`, `CITextGenerator` — result visualization overlays
- `CIKernel` (Metal Core Image) — custom kernels for optical flow visualization
- `CIImage.matchedFromWorkingSpace(to:)` / `matchedToWorkingSpace(from:)` — color space conversion

### High-Level Frameworks
- `VNDocumentCameraViewController` (VisionKit) — document scanning UI

## Code Highlights

Contour detection pipeline:
```swift
let contourRequest = VNDetectContoursRequest()
contourRequest.maximumImageDimension = 512
let handler = VNImageRequestHandler(ciImage: inputImage, options: [:])
try handler.perform([contourRequest])
let observation = contourRequest.results?.first as! VNContoursObservation
// Render all contours as a single CGPath
context.addPath(observation.normalizedPath)
```

Optical flow with Core Image visualization:
```swift
let visionRequest = VNGenerateOpticalFlowRequest(targetedCIImage: source, options: [:])
try requestHandler.perform([visionRequest], on: previousImage)
if let obs = visionRequest.results?.first as? VNPixelBufferObservation {
    let flowImage = CIImage(cvImageBuffer: obs.pixelBuffer)
    let visualizer = OpticalFlowVisualizerFilter()
    visualizer.inputImage = flowImage
    let output = visualizer.outputImage
}
```

## Takeaways
- Combining Core Image and Vision keeps processing in Apple's optimized pipeline — no matrix conversions, no third-party libraries, minimal memory overhead.
- `VNDetectContoursRequest` enables shape analysis with hierarchical child contours, polygon simplification, and `VNGeometryUtils` for area/perimeter/bounding-circle metrics.
- `VNGenerateOpticalFlowRequest` provides per-pixel motion data that image registration cannot, enabling detailed motion analysis in video.
- New `CIColorThreshold`, `CIColorThresholdOtsu`, and `CIColorAbsoluteDifference` filters in Core Image 2020 make preprocessing for CV tasks simpler and more effective.

---
_Source: WWDC20 Session 10673 page (abstract, chapter summaries, code samples, and resource links)._
