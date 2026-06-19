# Classify Hand Poses and Actions with Create ML
**WWDC21 · Session 10039** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10039/)

_Platforms:_ iOS 14+, iPadOS 14+, macOS Big Sur 11+

## Overview
Create ML introduces two new templates in 2021: Hand Pose Classification and Hand Action Classification. These allow developers to train custom Core ML models that classify single-handed static poses (evaluated on still images) and dynamic hand actions (evaluated on sequences of frames/video clips) using the 21 joint key points detected by Vision's `VNDetectHumanHandPoseRequest`.

The session walks through the full pipeline: collecting and categorizing training data (images for poses, short videos for actions), training with the Create ML app including augmentation, previewing results with the live camera preview, and integrating the resulting Core ML models into an ARKit-based app alongside Vision for real-time key point detection. Vision also gains a new `chirality` property on `VNHumanHandPoseObservation` to distinguish left from right hands.

## Key Topics

### Hand Pose Classification
A static pose is meaningful as a single image (e.g., a "one" finger, "peace" sign, "open palm"). Training data is a set of images organized into class folders. A **Background** class is essential — it should include both random unrelated hand positions and transitional poses (intermediate positions the hand passes through when moving toward or away from a recognized pose).

The Create ML app's **Live Preview** tab shows real-time predictions from the FaceTime camera against the trained model before export. Augmentations (flip, rotation, etc.) extend training data to improve robustness.

Model input at inference: the `keypointsMultiArray()` from a `VNHumanHandPoseObservation` (shape: 3 × 21, x/y coordinates + confidence per joint).

### Hand Action Classification
An action requires movement over time. Training data is short video clips organized into class folders (or a flat folder + annotation CSV/JSON with filename, class, start/end times). A Background class of unrelated motions is essential.

Training parameters: video frame rate, action duration (determines prediction window size), training iterations. Augmentations include **Time Interpolate** and **Frame Drop** for video variation.

At inference, the model expects a sequence (FIFO queue) of `keypointsMultiArray` values. Queue size = `actionDuration × frameRate` (e.g., 1.5 s at 30 fps = 45 frames). When using ARKit (60 fps), downsample to match training frame rate (skip every other frame). The queue is sampled at a configurable interval to balance responsiveness vs. compute.

### Chirality (New in Vision)
`VNHumanHandPoseObservation.chirality` — `.left`, `.right`, `.unknown` — identifies each hand separately; predictions for each hand are independent.

### Data & Quality Guidelines
- Hand Pose Classifier: ~500 images per class.
- Hand Action Classifier: ~100 videos per class.
- Keep hands within ~11 ft (3.5 m) of camera.
- Avoid extreme lighting; avoid bulky gloves.
- Confidence thresholds (~0.9) reduce jitter; Background class enables high-threshold filtering.

## APIs & Frameworks

**Create ML** (`import CreateML`)
- `MLHandPoseClassifier` **[NEW]** — trains hand pose classifier from image dataset
  - `MLHandPoseClassifier.DataSource.labeledDirectories(at:)` — class-per-folder image dataset
  - `MLHandPoseClassifier.ModelParameters` — training parameters (augmentation options)
- `MLHandActionClassifier` **[NEW]** — trains hand action classifier from video dataset
  - `MLHandActionClassifier.DataSource.labeledDirectories(at:)` — class-per-folder video dataset
  - `MLHandActionClassifier.DataSource.labeledFiles(at:annotationFile:)` — flat folder + annotation file
  - `MLHandActionClassifier.ModelParameters` — includes `videoFrameRate`, `predictionWindowSize`, `augmentationOptions`
  - `MLHandActionClassifier.ModelParameters.AugmentationOptions` — `.timeInterpolate` **[NEW]**, `.frameDrop` **[NEW]**
- Create ML app: Live Preview for Hand Pose Classifier **[NEW]** — real-time FaceTime camera inference

**Vision** (`import Vision`)
- `VNDetectHumanHandPoseRequest` — detects hand poses and returns joint positions
  - `.maximumHandCount` — max hands to detect (default 2)
  - `.revision` — `VNDetectHumanHandPoseRequestRevision1`
  - `.results` → `[VNHumanHandPoseObservation]`
- `VNHumanHandPoseObservation` — single hand detection result
  - `.keypointsMultiArray()` — `MLMultiArray` of shape [3, 21] (x, y, confidence per joint)
  - `.recognizedPoint(_:)` — returns `VNRecognizedPoint` for a named joint
  - `.chirality` **[NEW]** — `VNChirality`: `.left`, `.right`, `.unknown`
- `VNHumanHandPoseObservation.JointName` — named joint constants
  - `.indexTip`, `.middleTip`, `.wrist`, etc. (21 joints total)
- `VNRecognizedPoint.confidence` — float confidence of joint detection
- `VNRecognizedPoint.location` — normalized CGPoint (origin lower-left)
- `VNImageRequestHandler(cvPixelBuffer:options:)` — performs requests on a pixel buffer

**Core ML** (`import CoreML`)
- `MLMultiArray(concatenating:axis:dataType:)` — concatenates a queue of per-frame arrays into the sequence input for action classification
- Generated model `.prediction(poses:)` — inference on a single pose (hand pose) or sequence (hand action)
- `.label`, `.labelProbabilities` — prediction output

**ARKit** (`import ARKit`)
- `ARSession` / `ARFrame` — provides `capturedImage` pixel buffer at 60 fps
- Used as camera source; downsample to 30 fps for training-matched inference

## Code Highlights

Hand Pose detection and classification per frame:
```swift
let handPoseRequest = VNDetectHumanHandPoseRequest()
handPoseRequest.maximumHandCount = 1
let handler = VNImageRequestHandler(cvPixelBuffer: pixelBuffer, options: [:])
try handler.perform([handPoseRequest])

guard let observation = handPoseRequest.results?.first else { return }
let keypointsMultiArray = try observation.keypointsMultiArray()
let prediction = try model.prediction(poses: keypointsMultiArray)
if prediction.labelProbabilities[prediction.label]! > 0.9 {
    renderEffect(for: prediction.label)
}
```

Chirality filtering and hand action classification with FIFO queue:
```swift
// Downsample ARKit 60fps → 30fps to match training
if frameCounter % 2 == 0 {
    for (pose, chirality) in getHands() where chirality == .right {
        queue.append(pose)
        queue = Array(queue.suffix(queueSize)) // FIFO, size = 45
        if queue.count == queueSize && samplingCounter % samplingInterval == 0 {
            let poses = MLMultiArray(concatenating: queue, axis: 0, dataType: .float32)
            let prediction = try handActionModel.prediction(poses: poses)
            if let conf = prediction.labelProbabilities[prediction.label], conf > 0.85 {
                renderHandActionEffect(name: prediction.label)
            }
        }
    }
}
```

## Takeaways
- Hand Pose Classifier uses static images; Hand Action Classifier uses video clips — both require a Background class for robust real-world performance.
- At inference, hand action models require a FIFO queue of key point arrays sampled at the same frame rate used during training; mismatched rates cause incorrect predictions.
- Vision's new `chirality` property enables independent left/right hand handling, allowing simultaneous use of both classifiers with different hands.
- The Create ML app's Live Preview provides instant in-camera feedback on trained hand pose models before export to Core ML.

---
_Source: WWDC21 Session 10039 page (abstract, chapter summaries, code samples, and resource links)._
