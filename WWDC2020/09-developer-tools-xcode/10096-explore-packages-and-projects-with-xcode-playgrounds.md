# Explore Packages and Projects with Xcode Playgrounds
**WWDC20 · Session 10096** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10096/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11

## Overview
Xcode 12 delivers a full integration of Playgrounds with the modern build system, unlocking seamless use of Swift packages, framework targets, asset catalogs, and Core ML models directly inside playgrounds. The new "Build Active Scheme" option causes Xcode to automatically build all targets in the selected scheme before running a playground, making any resulting modules importable without manual steps.

The session walks through three scenarios: using a playground embedded inside a Swift package as living, runnable documentation; importing framework and package targets from a parent project into a playground; and using build-system-supported resource types (asset catalogs, `.mlmodel` files) as playground resources. Full build logs for playgrounds are now available in the Report Navigator, making build failures diagnosable.

## Key Topics

**Playgrounds in Swift Packages**
- Playgrounds placed inside a Swift package are recognized by Xcode 12 and can import the package's own modules
- Ideal for package documentation — code samples are runnable and show live results
- When a package is added as a project dependency, its embedded playgrounds remain accessible (read-only)

**Build Active Scheme**
- New per-playground setting: "Build Active Scheme" (enabled by default for new playgrounds)
- When enabled, Xcode builds all targets in the active scheme before executing the playground
- Any module in the scheme or a dependency of a scheme target becomes importable
- Eliminates the previous need to manually build frameworks before opening a playground

**Resources in Playgrounds**
- All resource types the build system supports are now usable in playground Resources folders
- Asset catalogs (`.xcassets`) are compiled by the build system and accessible via `UIImage(named:)`
- Core ML models (`.mlmodel`) are compiled and generate a Swift class automatically
- Drag resources into the playground's `Resources` folder; Xcode handles compilation

**Build Logs**
- Full playground build logs are now visible in Xcode's Report Navigator
- Shows compilation of scheme targets, generated modules, and playground sources/resources

## APIs & Frameworks

### PlaygroundSupport
- `PlaygroundPage.current.setLiveView(_:)` — sets a SwiftUI or UIKit view as the live view in the playground

### Core ML
- `MLModelConfiguration` — configuration object for model instantiation
- `YOLOv3(configuration:)` — auto-generated class from compiled `.mlmodel` (name matches model class in model metadata)
- `.model` property — returns `MLModel` from a typed model wrapper

### Vision
- `VNCoreMLModel(for:)` — wraps an `MLModel` for use with Vision requests
- `VNCoreMLRequest(model:completionHandler:)` — performs Core ML inference via Vision
- `VNImageRequestHandler(cgImage:)` — handler for single-image Vision requests
- `VNImageRequestHandler.perform(_:)` — executes an array of `VNRequest`
- `VNRecognizedObjectObservation` — result of object detection; contains `labels` and `boundingBox`
- `VNRecognizedObjectObservation.labels` — array of `VNClassificationObservation` sorted by confidence
- `VNImageRectForNormalizedRect(_:_:_:)` — converts normalized Vision coordinates to image-space `CGRect`

### UIKit / SwiftUI
- `UIImage(named:)` — loads images from asset catalog resources
- SwiftUI `View`, `List`, `Image`, `ZStack`, `Rectangle`, `GeometryReader` — used in result visualization

## Code Highlights

Loading an asset catalog image:
```swift
import UIKit
let image = UIImage(named: "ingredient/orange")
```

Instantiating a compiled Core ML model:
```swift
import CoreML
let yoloModel = try YOLOv3(configuration: MLModelConfiguration()).model
```

Running Vision object detection and reading results:
```swift
import Vision
let model = try VNCoreMLModel(for: yoloModel)
let request = VNCoreMLRequest(model: model) { _, _ in }
let handler = VNImageRequestHandler(cgImage: image.cgImage!)
try? handler.perform([request])
let observations = request.results as! [VNRecognizedObjectObservation]
let topLabel = observations.first?.labels[0].identifier
```

Setting a SwiftUI live view:
```swift
import PlaygroundSupport
PlaygroundPage.current.setLiveView(
    RecognizedObjectVisualizer(withResults: results)
        .frame(width: 500, height: 800)
)
```

## Takeaways
- Enable "Build Active Scheme" in playground settings to automatically import any framework or package in the active scheme — no manual build steps required.
- Embed a playground in your Swift package's `Playgrounds/` folder to ship runnable documentation alongside the package code.
- Asset catalogs and `.mlmodel` files in a playground's `Resources` folder are compiled by the build system, enabling `UIImage(named:)` and auto-generated model classes.
- Full build logs in the Report Navigator make it straightforward to debug playground compilation failures.

---
_Source: WWDC20 Session 10096 page (abstract, chapter summaries, code samples, and resource links)._
