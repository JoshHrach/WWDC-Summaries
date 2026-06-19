# Compose Advanced Models with Create ML Components
**WWDC22 · Session 10020** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10020/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13

## Overview
This session builds on the introduction to Create ML Components from "Get to know Create ML Components" (Session 10019) and dives into advanced usage with temporal data such as video and audio. The presenter demonstrates how to compose transformers and temporal transformers to build a human action repetition counter that works both on pre-recorded video and in a live camera feed.

The second major topic is training a custom sound classifier using Create ML Components, with a focus on workflow efficiency. By separating feature extraction from model fitting, developers can avoid redundant computation when iterating on model design or adding new training data.

The session closes with a deep dive into incremental fitting — training models in batches to accommodate memory constraints, implementing early stopping based on validation accuracy, and checkpointing model state during training so that long-running jobs can be paused and resumed.

## Key Topics

### Temporal Data and AsyncSequence
Video and audio data are inherently sequential. Create ML Components uses Swift's `AsyncSequence` to represent streams of frames or audio buffers, enabling composable, asynchronous processing pipelines.

### Temporal Transformers
- **VideoReader** — reads a video file as an `AsyncSequence` of frames.
- **Downsampler** — reduces the frame rate of an async sequence.
- **SlidingWindowTransformer** — groups consecutive frames into overlapping or non-overlapping windows; configured via `windowLength` and `stride`.
- **AudioFeaturePrint** — a temporal transformer that windows an async sequence of audio buffers and extracts audio features.

### Human Action Repetition Counter
By composing `HumanBodyPoseExtractor`, `PoseSelector` (with selection strategies like `rightMostJointLocation`), `SlidingWindowTransformer`, and `HumanBodyActionCounter`, the session builds a pipeline that produces a floating-point cumulative repetition count. The same pipeline is extended to a live camera feed using `readCamera`.

### Custom Sound Classifier with Incremental Fitting
A custom sound classifier is built from `AudioFeaturePrint` (feature extractor) composed with a classifier. The session explains how to preprocess features once and then fit different classifiers — or re-fit with new data — without repeating expensive feature extraction.

Incremental fitting requires an updatable classifier such as `FullyConnectedNeuralNetworkClassifier`. A training loop iterates for a fixed number of epochs, calling `update` on each batch. Early stopping is implemented by evaluating validation accuracy each iteration and breaking when a threshold is reached.

## APIs & Frameworks

### CreateMLComponents
- `VideoReader` **[NEW]** — reads video files as `AsyncSequence<Frame>`
- `Downsampler` **[NEW]** — temporal transformer to downsample frame sequences
- `SlidingWindowTransformer` **[NEW]** — groups frames into windows; `windowLength`, `stride` parameters
- `HumanBodyPoseExtractor` **[NEW]** — extracts `HumanBodyPose` array from an image using Vision
- `PoseSelector` **[NEW]** — selects a single pose from an array; `SelectionStrategy` (e.g., `.rightMostJointLocation`)
- `HumanBodyActionCounter` **[NEW]** — temporal transformer returning cumulative `Float` repetition count
- `AudioFeaturePrint` **[NEW]** — temporal transformer extracting audio features from `AsyncSequence<AVAudioBuffer>`
- `FullyConnectedNeuralNetworkClassifier` **[NEW]** — updatable classifier for incremental fitting
- `LogisticRegressionClassifier` — non-updatable classifier option
- `.preprocess(_:)` method — separates feature extraction from estimator fitting
- `.update(_:)` method — updates an updatable estimator with a batch of preprocessed features
- `mapFeatures(_:)` — maps validation data through learned transformers for evaluation
- `readCamera(configuration:)` **[NEW]** — reads a live camera stream as `AsyncSequence<Frame>`

### Swift Algorithms Package
- `chunks(ofCount:)` — batches a collection into fixed-size chunks for incremental training

### Vision (used internally by HumanBodyPoseExtractor)
- Human body pose detection pipeline

## Code Highlights

Composing a repetition-counting pipeline:
```swift
let task = HumanBodyPoseExtractor()
    .appending(PoseSelector(strategy: .rightMostJointLocation))
    .appending(SlidingWindowTransformer(windowLength: 90, stride: 90))
    .appending(HumanBodyActionCounter())
```

Incremental fitting loop with early stopping:
```swift
var model = FullyConnectedNeuralNetworkClassifier()
let features = try await featureExtractor.preprocess(trainingData)
for _ in 0..<maxIterations {
    for batch in features.chunks(ofCount: batchSize) {
        model = try await model.update(with: batch)
    }
    let predictions = try await classifier.mapFeatures(validationFeatures, model: model)
    let accuracy = computeAccuracy(predictions)
    if accuracy >= 0.95 { break }
}
try model.write(to: outputURL)
```

## Takeaways
- Temporal transformers (Downsampler, SlidingWindowTransformer, AudioFeaturePrint) extend the Create ML Components composition model to time-series data like video and audio.
- Separating feature extraction from classifier fitting enables efficient iteration — extract once, retrain many times with different parameters or new data.
- Incremental fitting with `FullyConnectedNeuralNetworkClassifier` supports batched training, early stopping, and mid-training checkpointing.
- The same temporal pipeline used for offline video analysis can be adapted to live camera input with minimal code changes using `readCamera`.

---
_Source: WWDC22 Session 10020 page (abstract, chapter summaries, code samples, and resource links)._
