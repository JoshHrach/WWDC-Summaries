# Build an Action Classifier with Create ML
**WWDC20 · Session 10043** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10043/)

_Platforms:_ iOS 14, macOS Big Sur 11

## Overview
This session introduces Action Classification, a new Create ML template for training models that recognize human body actions from video. Unlike sensor-based Activity Classification, Action Classification uses Vision's human body pose estimation under the hood to identify 18 body landmarks per frame, then classifies sequences of poses over a configurable time window.

The session is split into two halves: data collection and training in the Create ML app, followed by integration of the resulting Core ML model into an iOS fitness app. The fitness app demo recognizes jumping jacks, lunges, squats, and other exercises in real time from a camera stream, providing an interactive coaching experience without requiring the user to touch the device.

Best practices for capturing training videos, configuring the prediction window duration, selecting a single person from multi-person frames, and counting action repetitions are covered in detail.

## Key Topics

**Action Classification Template in Create ML**
A new template in Create ML on macOS Big Sur 11. Training data consists of labeled video folders (one folder per class). Montage videos (multiple actions in one file) are supported via a JSON/CSV annotation file specifying start and end timestamps per action segment.

**Feature Extraction via Vision**
Before model training begins, Create ML runs `VNDetectHumanBodyPoseRequest` on every frame of every training video to extract 18 body landmark positions (x, y, confidence). This is the dominant time cost during training.

**Action Duration / Prediction Window**
The key hyperparameter is the prediction window size (seconds or number of frames). Short repeating actions like exercise reps use ~2 seconds; complex moves like dance can require up to 10 seconds. Frame rate affects effective window length, so training and inference video frame rates should match.

**Augmentation**
Horizontal flip augmentation generates mirrored training examples to improve orientation robustness without recording additional videos.

**Model Integration in iOS**
The exported `MLModel` takes a `[windowSize, 3, 18]` `MLMultiArray` of concatenated pose arrays as input and outputs a label string plus a probability dictionary. Poses are obtained from Vision and converted using `keypointsMultiArray()`.

## APIs & Frameworks

### Create ML **[NEW]**
- `MLCreateML` app — Action Classification template **[NEW]**
- Training parameters: action duration (seconds), prediction window size (frames), augmentations (horizontal flip)
- Annotation file support (CSV/JSON) for montage video labeling **[NEW]**
- Preview tab for testing model on new videos with pose skeleton overlay

### Vision **[NEW]**
- `VNDetectHumanBodyPoseRequest` **[NEW]** — detects 18 human body landmarks per frame
- `VNRecognizedPointsObservation` **[NEW]** — result type holding landmark positions
- `VNRecognizedPointsObservation.keypointsMultiArray()` **[NEW]** — convenience method returning `[1, 3, 18]` `MLMultiArray`
- `VNVideoProcessor(url:)` **[NEW]** — processes pose requests across an entire video file
- `VNVideoProcessor.add(_:)` **[NEW]**
- `VNVideoProcessor.analyze(with:)` **[NEW]**
- `VNImageRequestHandler(url:)` — single-image pose extraction from camera frames
- `VNImageRequestHandler.perform(_:)` — executes the pose request

### Core ML
- `MLMultiArray(concatenating:axis:dataType:)` **[NEW]** — concatenates per-frame pose arrays into a prediction window
- `MLMultiArray` — model input type (`[60, 3, 18]` for a 60-frame window)
- Generated model class (e.g., `FitnessClassifier`) — `.prediction(poses:)` returning label and `labelProbabilities`

### AVFoundation
- `CMSampleBuffer` — camera frame type passed to pose extraction
- `CMTime`, `CMTimeRange` — time range for `VNVideoProcessor`

## Code Highlights

Extracting poses from a video file:
```swift
let request = VNDetectHumanBodyPoseRequest(completionHandler: { request, error in
    let poses = request.results as! [VNRecognizedPointsObservation]
})
let processor = VNVideoProcessor(url: videoURL)
try processor.add(request)
try processor.analyze(with: CMTimeRange(start: .zero, end: .indefinite))
```

Building the prediction window and running the model:
```swift
let poseMultiArrays: [MLMultiArray] = try posesWindow.map { person in
    guard let person = person else { return zeroPaddedMultiArray() }
    return try person.keypointsMultiArray()
}
let modelInput = MLMultiArray(concatenating: poseMultiArrays, axis: 0, dataType: .float)
let predictions = try fitnessClassifier.prediction(poses: modelInput)
// predictions.label, predictions.labelProbabilities
```

Annotation file for montage videos (JSON):
```json
[{ "file_name": "Montage1.mov", "label": "Squats", "start_time": 4.5, "end_time": 8 }]
```

## Takeaways
- Action Classification in Create ML uses Vision body pose estimation to classify human movements from video; it works on single persons and is not suitable for animals or objects.
- The prediction window size (action duration) must be tuned to match the actual length of the action being classified — this is the most critical training parameter.
- Use `VNRecognizedPointsObservation.keypointsMultiArray()` to convert Vision pose results directly into Core ML-compatible input arrays, simplifying integration significantly.
- For robust models: collect ~50 videos per class with diverse performers, use a stable camera, and include a negative "other" class for when no target action is occurring.

---
_Source: WWDC20 Session 10043 page (abstract, chapter summaries, code samples, and resource links)._
