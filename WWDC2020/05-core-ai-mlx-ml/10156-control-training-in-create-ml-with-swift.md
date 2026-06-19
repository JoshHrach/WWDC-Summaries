# Control Training in Create ML with Swift
**WWDC20 · Session 10156** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10156/)

_Platforms:_ macOS Big Sur 11

## Overview
The Create ML framework gains a new asynchronous training API in macOS Big Sur that gives Swift developers programmatic control over the entire ML training lifecycle. Rather than calling a blocking model constructor and waiting for completion, developers call a `train(...)` method that returns an `MLJob` — an object providing Combine publishers for progress monitoring, checkpoint events, and final results. Training can be paused (`job.cancel()`), resumed (call `train` again with the same session directory), and extended (increase the iteration count in subsequent calls).

The session uses an Xcode Playground to demonstrate Style Transfer model training with live SwiftUI visualization: each checkpoint renders the stylized validation image inline in the Playground Live View, giving real-time visual feedback on model quality. This is the same technology that powers the Create ML app's checkpoint preview feature.

Checkpoints are automatically saved to a session directory; the `MLTrainingSession` type allows querying all checkpoint metadata (phase, iteration, loss values) and selectively removing old checkpoints to reclaim disk space.

## Key Topics

**Why Training Control Matters**
Long-running tasks (object detection training can take 5+ hours) may reach their iteration limit while still improving. Without control, the only option is to restart from scratch. With the new API, training can be resumed from the last checkpoint and extended. Custom stopping criteria can be implemented in the checkpoint handler — stop early if a target accuracy is reached, or stop based on a time budget.

**Asynchronous Training with MLJob**
The `.train()` class method on task types (`MLStyleTransfer`, `MLActivityClassifier`, `MLObjectDetector`, etc.) accepts `sessionParameters: MLTrainingSessionParameters` and returns `MLJob<ModelType>` instead of blocking. `MLJob` exposes three Combine publishers:
- `result` — fires once with `Result<Model, Error>` when training completes
- `progress` — fires at each progress report interval; backed by a standard `Foundation.Progress` instance
- `checkpoints` — fires at each checkpoint interval with an `MLCheckpoint`

**MLTrainingSessionParameters**
Configures the session: `sessionDirectory` (reuse for resume; delete to start fresh), `reportInterval` (iterations between progress publisher events), `checkpointInterval` (iterations between checkpoint saves), `iterations` (maximum iteration count; increase in later calls to extend training).

**Progress Monitoring with Combine**
`job.progress` is a standard `Foundation.Progress` instance. Use `KVO publisher` via `.publisher(for: \.fractionCompleted)` to observe completion percentage. Use `MLProgress(progress: job.progress)` to access Create ML-specific metrics (`itemCount`, `totalItemCount`, `.metrics[.accuracy]`, `.metrics[.loss]`, `.metrics[.styleLoss]`, `.metrics[.contentLoss]`, `.metrics[.stylizedImageURL]`).

**Checkpoints and Sessions**
`MLCheckpoint` — represents a point-in-time model snapshot; has `.phase` (`.featureExtraction` or `.training`), `.iteration`, and `.metrics`. Only training-phase checkpoints can be used to generate a model (`ModelType(checkpoint:)`). `MLTrainingSession` provides access to all saved checkpoints, loss history, session metadata, and checkpoint deletion (`removeCheckpoints(where:)`). Retrieve with `ModelType.restoreTrainingSession(sessionParameters:)`.

**Task Support Matrix**
- Feature extraction checkpoints: image classification, sound classification, action classification
- Training checkpoints: action classification, object detection, Style Transfer, activity classification

**Pause, Resume, Extend**
- Pause: `job.cancel()`
- Resume: call `ModelType.train(...)` with the same `sessionDirectory` — Create ML reads existing checkpoints and continues from the last one
- Extend: call `train(...)` again with a higher `iterations` value in `sessionParameters`

## APIs & Frameworks

### Create ML (macOS Big Sur) **[NEW]**
- `MLJob<Output>` **[NEW]** — active training job returned from `.train()`
  - `MLJob.result: AnyPublisher<Output, Error>` **[NEW]** — fires once on completion
  - `MLJob.progress: Progress` **[NEW]** — `Foundation.Progress` instance for the active job
  - `MLJob.checkpoints: AnyPublisher<MLCheckpoint, Error>` **[NEW]** — fires on each checkpoint
  - `MLJob.cancel()` **[NEW]** — stops training (can be resumed later)
- `MLTrainingSessionParameters` **[NEW]** — configures the session
  - `init(sessionDirectory:reportInterval:checkpointInterval:iterations:)` **[NEW]**
  - `sessionDirectory: URL` — where session state is persisted; reuse to resume
  - `reportInterval: Int` — iterations between progress events
  - `checkpointInterval: Int` — iterations between checkpoint saves
  - `iterations: Int` — maximum iterations
- `MLCheckpoint` **[NEW]** — snapshot of model state at an iteration
  - `MLCheckpoint.phase: MLPhase` **[NEW]** — `.featureExtraction` or `.training`
  - `MLCheckpoint.iteration: Int` **[NEW]**
  - `MLCheckpoint.metrics: [MLMetricKey: Any]` **[NEW]**
- `MLProgress` **[NEW]** — wrapper around `Foundation.Progress` providing ML-specific metrics
  - `MLProgress(progress:)` **[NEW]** — may return `nil` if metrics aren't available yet
  - `MLProgress.itemCount: Int` **[NEW]** — current iteration or file index
  - `MLProgress.totalItemCount: Int?` **[NEW]**
  - `MLProgress.metrics: [MLMetricKey: Any]` **[NEW]**
- `MLMetricKey` **[NEW]** — `.accuracy`, `.loss`, `.styleLoss`, `.contentLoss`, `.stylizedImageURL`, and more; task-specific
- `MLTrainingSession<ModelType>` **[NEW]** — represents a saved session
  - `checkpoints: [MLCheckpoint]` **[NEW]**
  - `removeCheckpoints(where:)` **[NEW]** — deletes matching checkpoints from disk
- `ModelType.train(..., sessionParameters:)` **[NEW]** — async train on: `MLStyleTransfer`, `MLActivityClassifier`, `MLObjectDetector`, `MLSoundClassifier`, `MLImageClassifier`, etc.
- `ModelType.restoreTrainingSession(sessionParameters:)` **[NEW]** — access saved session
- `ModelType(checkpoint:)` **[NEW]** — create a model from a training checkpoint (training phase only)

### Combine (used for monitoring)
- `Publisher.sink(receiveCompletion:receiveValue:)` — attaches subscriber
- `AnyCancellable` / `.store(in: &subscriptions)` — manages subscription lifetime
- `Progress.publisher(for: \.fractionCompleted)` — KVO-backed publisher on `Foundation.Progress`
- `Publisher.compactMap(_:)` — filters nil values in checkpoint pipeline
- `Publisher.receive(on:)` — switch to main queue for UI updates

### Xcode Playgrounds
- `PlaygroundSupport.PlaygroundPage.current.setLiveView(_:)` — renders SwiftUI view inline in the Live View panel

## Code Highlights

Setting up session parameters and starting async training:
```swift
let sessionParameters = MLTrainingSessionParameters(
    sessionDirectory: sessionDirectory,
    reportInterval: 10,
    checkpointInterval: 100,
    iterations: 1000
)

let job = try MLStyleTransfer.train(
    trainingData: dataSource,
    parameters: trainingParameters,
    sessionParameters: sessionParameters
)
```

Receiving the final model via Combine:
```swift
var subscriptions = [AnyCancellable]()

job.result.sink { result in
    if case .failure(let error) = result { print(error) }
} receiveValue: { model in
    try? model.write(to: outputURL)
}.store(in: &subscriptions)
```

Monitoring progress with MLProgress:
```swift
job.progress
    .publisher(for: \.fractionCompleted)
    .sink { [weak job] _ in
        guard let job = job,
              let progress = MLProgress(progress: job.progress) else { return }
        print("Iteration \(progress.itemCount) / \(progress.totalItemCount ?? 0)")
        print("Style loss: \(progress.metrics[.styleLoss] ?? 0)")
    }.store(in: &subscriptions)
```

Generating a model from a checkpoint:
```swift
job.checkpoints.sink { checkpoint in
    guard checkpoint.phase == .training else { return }
    let model = try? MLActivityClassifier(checkpoint: checkpoint)
    try? model?.write(to: outputURL)
}.store(in: &subscriptions)
```

Pause and resume:
```swift
job.cancel()  // pause

// Resume later — same sessionDirectory picks up from last checkpoint:
let resumedJob = try MLStyleTransfer.train(
    trainingData: dataSource,
    parameters: trainingParameters,
    sessionParameters: sessionParameters  // same sessionDirectory
)
```

Accessing and pruning a saved session:
```swift
let session = MLObjectDetector.restoreTrainingSession(sessionParameters: sessionParameters)
let losses = session.checkpoints.compactMap { $0.metrics[.loss] as? Double }
session.removeCheckpoints { $0.iteration < 500 }  // free disk space
```

## Takeaways
- The new `MLJob` API transforms Create ML from a blocking, fire-and-forget tool into a fully interactive workflow: monitor loss in real time, stop early when satisfied, resume from checkpoints without restarting, and extend training by increasing the iteration cap in a subsequent call.
- Reuse the same `sessionDirectory` across multiple calls to `train(...)` to resume automatically from the last checkpoint; delete the directory or use a new path to start a fresh session.
- Use `MLProgress(progress: job.progress)` inside the `progress` publisher handler to access task-specific metrics (`.accuracy`, `.styleLoss`, `.stylizedImageURL`) — `nil` return from the initializer signals that the current phase has no metrics yet (e.g., session startup or phase transition).
- The checkpoint publisher is the right place to implement custom stopping criteria: evaluate the checkpoint's loss or accuracy, generate and inspect a model, and call `job.cancel()` if the threshold is met — without needing to specify the iteration count upfront.

---
_Source: WWDC20 Session 10156 page (transcript, code samples, and resource links)._
