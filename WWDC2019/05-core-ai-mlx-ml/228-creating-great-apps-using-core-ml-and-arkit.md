# Creating Great Apps Using Core ML and ARKit
**WWDC19 · Session 228** · [Watch](https://developer.apple.com/videos/play/wwdc2019/228/)

_Platforms:_ iOS 13, iPadOS 13

## Overview
This session walks through the end-to-end construction of an educational math game for children that blends Core ML, ARKit, Vision, PencilKit, and the Speech framework. The app lets children roll physical dice on a table, have the device recognize each die and its face value via object detection, draw or speak their mathematical answers, and then see results reflected in an AR game board.

Two recurring themes guide the session: first, asking whether machine learning can solve a given problem better than a programmatic approach (and identifying the right data for that); second, understanding model behavior by carefully inspecting both inputs and outputs so unexpected results can be diagnosed and corrected.

The journey covers training a custom object detection model with Create ML, integrating it through Vision, handling multi-stroke handwriting recognition with MNIST + PencilKit, using on-device speech recognition, and finally compositing everything into an ARKit experience.

## Key Topics

- **Object detection for dice recognition** — why a programmatic approach fails (color, shape, perspective all vary) and how Create ML was used to train a custom object detector on annotated dice images; iterating from detecting whole dice to detecting only the tops to handle occlusion.
- **Debugging model behavior** — visualizing bounding-box output on a live camera feed; identifying image-orientation bugs by inspecting the raw input image with Xcode Quick Look; understanding why unusual outputs often point to a bad input rather than a bad model.
- **Determining roll completion** — comparing consecutive frame observations: same count, same face values, and bounding boxes overlapping ≥85% before declaring a roll finished.
- **Handwriting recognition with MNIST** — using a pre-trained MNISTClassifier model with Vision and PencilKit; diagnosing stroke-thinning artifacts caused by downscaling and fixing them with a thicker marker tool.
- **Multi-stroke / multi-digit input handling** — a programmatic approach using stroke-overlap detection to decide when to combine strokes into a single prediction vs. treat them as separate digits.
- **On-device speech recognition** — using the Speech framework with `requiresOnDeviceRecognition = true` for offline, private digit input.
- **ARKit integration** — rendering a circular nine-section game board in AR; combining dice detection results with user input to drive game state.

## APIs & Frameworks

### Core ML
- `MLModel` — loading custom trained models **[NEW in context of Create ML integration]**
- `MNISTClassifier` — pre-trained handwritten digit classifier (available on Core ML Models page) **[NEW]**

### Vision
- `VNRecognizedObjectObservation` — result type from object detection requests; carries bounding box in normalized image coordinates **[NEW]**
- `VNCoreMLRequest` — wrapping a Core ML model for Vision inference
- `VNImageRequestHandler` — performing Vision requests on a captured image
- `VNDetectRectanglesRequest` — (referenced implicitly via object detection pipeline)

### Create ML
- Object detection model training with annotated bounding-box data **[NEW]**
- Multi-class object detection (dice faces as separate classes) **[NEW]**

### ARKit
- `ARSCNView` — AR scene rendering host
- World tracking and plane detection for placing the game board

### PencilKit
- `PKCanvasView` — drawing canvas **[NEW]**
- `PKTool` / marker tool with configurable stroke width **[NEW]**
- `allowsFingerDrawing` property **[NEW]**
- Extracting a `UIImage` snapshot from `PKCanvasView` for Vision inference

### Speech
- `SFSpeechRecognizer` — speech-to-text **[NEW offline support]**
- `requiresOnDeviceRecognition` property — forces fully on-device, offline recognition **[NEW]**

## Code Highlights

Handling `VNRecognizedObjectObservation` results:

```swift
func handleObservations(_ observations: [VNRecognizedObjectObservation]) {
    // Count dice
    let diceCount = observations.count

    // Draw bounding boxes (normalized → view coordinates)
    for observation in observations {
        let viewRect = mapToViewCoordinates(observation.boundingBox)
        let layer = makeRoundedRectLayer(for: viewRect)
        overlayLayer.addSublayer(layer)
    }
}
```

Detecting roll completion:

```swift
func rollHasEnded(previous: [VNRecognizedObjectObservation],
                  current:  [VNRecognizedObjectObservation]) -> Bool {
    guard current.count == previous.count else { return false }
    var matches = 0
    for prev in previous {
        for curr in current {
            guard curr.labels.first?.identifier == prev.labels.first?.identifier else { continue }
            let overlap = prev.boundingBox.intersection(curr.boundingBox).area /
                          prev.boundingBox.area
            if overlap > 0.85 { matches += 1; break }
        }
    }
    return matches == previous.count
}
```

Configuring PencilKit for thicker strokes:

```swift
canvasView.allowsFingerDrawing = true
canvasView.tool = PKInkingTool(.marker, color: .black, width: 20)
```

Requiring on-device speech recognition:

```swift
let recognizer = SFSpeechRecognizer()
recognizer?.supportsOnDeviceRecognition  // check capability
let request = SFSpeechAudioBufferRecognitionRequest()
request.requiresOnDeviceRecognition = true
```

## Takeaways

- Always visualize model inputs (use Xcode Quick Look on images) and outputs (draw bounding boxes) to diagnose unexpected behavior — many "model bugs" are actually data-pipeline bugs such as wrong image orientation or unintended downscaling.
- Object detection models can serve dual purposes: detecting presence/count *and* classifying object state (e.g., die face value) by treating each state as a separate class.
- Not every problem needs ML; once a good model is in place, interpreting its output programmatically (e.g., overlap comparison for roll detection, stroke grouping for multi-digit input) can be simpler and more reliable than training additional models.
- The new `requiresOnDeviceRecognition` flag on `SFSpeechRecognizer` enables fully offline, privacy-preserving speech input in iOS 13.

---
_Source: WWDC19 Session 228 page (abstract, full transcript, code samples, and resource links)._
