# Detect Animal Poses in Vision
**WWDC23 · Session 10045** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10045/)

_Platforms:_ iOS 17, iPadOS 17, tvOS 17, macOS Sonoma 14

## Overview
Vision gains a major new capability in 2023: Animal Body Pose detection for cats and dogs. This session by Nadia Zouba from the Vision team introduces `VNDetectAnimalBodyPoseRequest`, which identifies 25 skeletal joints across an animal's body in real time, enabling applications from pet activity monitoring to creative photo embellishments. The session also covers several other important Vision framework updates including new stateful tracking requests, `MLComputeDevice` support, and improvements to barcode, text, and face-quality detection.

The animal pose API mirrors the design of the existing `VNDetectHumanBodyPoseRequest` from 2020, making it straightforward for developers already familiar with human pose detection to adopt. It runs on the Neural Engine for real-time performance with live camera streams.

## Key Topics

### Animal Body Pose Detection
`VNDetectAnimalBodyPoseRequest` is the primary new API. It processes images or video frames and returns one `VNAnimalBodyPoseObservation` per detected animal. Each observation contains the joint locations for up to 25 landmarks organized into six named joint groups. The API supports detecting up to two animals per image.

**Joint Groups:**
- **Head** — ears (left, right), eyes (left, right), nose
- **Forelegs** — front left and right paws and joints
- **Hindlegs** — rear left and right paws and joints
- **Trunk** — neck
- **Tail** — three tail joints (root, mid, tip)
- **All** — union of all 25 joints

**Input requirements:** Image must be at least 64 pixels on each side. Runs in real time on Neural Engine.

**Use cases demonstrated:**
- Live skeletal overlay on a walking dog via camera feed
- Pose classification (stretching, standing, begging, running, sleeping)
- Combined with `VNRecognizeAnimalsRequest` for type + location + pose
- Animal activity analysis from video (motion over time)
- Creative embellishments: placing emoji or image overlays on detected joint locations (hats, glasses, skates)
- Camera tracking with DockKit motorized stands

### New Stateful Tracking Requests
`VNTargetedImage`-based requests are now available as Stateful Requests. Three new derived tracking requests are introduced, each named with the `Track` verb to make tracking workflows cleaner and more explicit.

### MLComputeDevice Support **[NEW]**
Vision now supports the `MLComputeDevice` API, allowing developers to:
- Query where a Vision request executes (CPU, GPU, Neural Engine)
- Specify which compute device to use for a given request

### Multilabel Classification Compatibility **[NEW]**
Core ML and Create ML multilabel classifiers (models that can output more than one label per input) are now compatible with Vision. Previously, only single-label classifiers were supported.

### Barcode Detection: Revision 4 **[NEW]**
`VNDetectBarcodesRequest` revision 4 adds:
- **MSIPlessey** symbology support
- Color-inverted QR code detection
- Note: Revision 1 is deprecated.

### Text Recognition Updates
`VNRecognizeTextRequest` adds language support for **Thai** and **Vietnamese**.

### Face Capture Quality: Revision 3 **[NEW]**
`VNDetectFaceCaptureQualityRequest` revision 3 improves quality scoring accuracy.

## APIs & Frameworks

### Vision — Animal Body Pose **[NEW]**
- `VNDetectAnimalBodyPoseRequest` — new request to detect animal skeletal joints **[NEW]**
- `VNAnimalBodyPoseObservation` — observation returned per detected animal **[NEW]**
- `VNAnimalBodyPoseObservation.JointName` — enum of 25 named joints **[NEW]**
- `VNAnimalBodyPoseObservation.JointsGroupName` — enum: `.head`, `.forelegs`, `.hindlegs`, `.trunk`, `.tail`, `.all` **[NEW]**
- `.recognizedPoints(_:)` — method on observation to retrieve a dictionary of recognized joints for a given group **[NEW]**
- `VNRecognizedPoint` — individual joint with normalized image coordinates and confidence
- `VNImageRequestHandler` — used to process the animal pose request on a single image
- `VNSequenceRequestHandler` — for processing on a video stream (live camera frames as `CMSampleBuffer`)

### Vision — Tracking Updates **[NEW]**
- Stateful Request variants for `VNTargetedImage`-based requests **[NEW]**
- New Track-prefixed stateful tracking request classes **[NEW]**

### Vision — Compute Device **[NEW]**
- `MLComputeDevice` — query and select compute device (CPU / GPU / Neural Engine) for Vision requests **[NEW]**

### Vision — Barcode **[UPDATED]**
- `VNDetectBarcodesRequest` — revision 4 **[NEW revision]**
  - MSIPlessey symbology **[NEW]**
  - Color-inverted QR code detection **[NEW]**
  - Revision 1 deprecated

### Vision — Text Recognition **[UPDATED]**
- `VNRecognizeTextRequest` — Thai and Vietnamese language support **[NEW]**

### Vision — Face Quality **[UPDATED]**
- `VNDetectFaceCaptureQualityRequest` — revision 3 with improved quality/accuracy **[NEW revision]**

### Existing Supporting APIs
- `VNDetectHumanBodyPoseRequest` — existing human pose API (design precedent for animal pose)
- `VNRecognizeAnimalsRequest` — detects and labels cats/dogs with bounding box (combine with animal pose for type + location + pose)

## Code Highlights

```swift
// 1. Create the animal body pose request
let request = VNDetectAnimalBodyPoseRequest()

// 2. Create a request handler from a CMSampleBuffer (live camera)
let handler = VNImageRequestHandler(cmSampleBuffer: sampleBuffer, options: [:])

// 3. Perform the request
try handler.perform([request])

// 4. Get observations
guard let observations = request.results as? [VNAnimalBodyPoseObservation] else { return }

for observation in observations {
    // 5. Get all joints
    let allPoints = try observation.recognizedPoints(.all)
    
    // 6. Draw skeleton by connecting joints
    for (_, point) in allPoints where point.confidence > 0.5 {
        // Convert normalized coords and draw
    }
}
```

## Takeaways
- `VNDetectAnimalBodyPoseRequest` follows the same pattern as `VNDetectHumanBodyPoseRequest` — adopt it with minimal learning curve.
- Combine with `VNRecognizeAnimalsRequest` to get species label + bounding box + full skeleton for maximum context.
- Use the Neural Engine path (default) for real-time performance with camera streams; check `MLComputeDevice` for fine-grained compute control.
- Animal pose unlocks creative, health, and behavior-monitoring use cases for pet-focused apps; update barcode detection to revision 4 to pick up MSIPlessey and inverted QR support.

---
_Source: WWDC23 Session 10045 page (abstract, chapter summaries, code samples, and resource links)._
