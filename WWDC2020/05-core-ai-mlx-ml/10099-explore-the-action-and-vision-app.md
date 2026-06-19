# Explore the Action & Vision App
**WWDC20 · Session 10099** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10099/)

_Platforms:_ iOS 14, iPadOS 14

## Overview
This session presents the design and implementation of the "Action & Vision" sample app — a real-time sports analysis tool built around a bean bag toss game. The app demonstrates how to combine Object Detection (Create ML), Action Classification (Create ML), Body Pose Estimation, Trajectory Detection, and Contour Detection from Vision into a cohesive live-camera pipeline. The full source code is available as a downloadable sample project.

The app is structured around a state machine (`GameManager`) that progresses through: finding and aligning the game board, confirming scene stability, measuring the board via contours, detecting a player, tracking bean bag trajectories, classifying throw types, and computing release speed and angle. Two custom Create ML models power the app — an Object Detection model for board recognition and an Action Classification model for throw-type prediction.

Live-stream processing best practices are covered extensively: returning camera buffers promptly, using separate queues for parallel analysis, and asynchronously rendering results on the main queue to avoid stalling the camera feed.

## Key Topics

**App Architecture**
- `GameManager` singleton drives state transitions observed by multiple view controllers
- `CameraViewController` owns the camera/video buffer pipeline; delegates via `OutputDelegate`
- `SetupViewController`, `GameViewController`, `SummaryViewController` conform to `GameStateChangeObserver` and `CameraViewControllerOutputDelegate`

**Prerequisites: Board Detection & Scene Stability**
- Custom Object Detection model (Create ML Object Detection template, transfer learning) run via `VNCoreMLRequest`
- Scene stability measured with `VNTranslationalImageRegistrationRequest` across 15 frames; stable when moving average of alignment points < 10 pixels

**Board Measurement via Contour Detection**
- `VNDetectContoursRequest` with `regionOfInterest` set to the detected board bounding box
- Simplified contours used to find board edges and hole
- Known physical board size (2 ft × 4 ft regulation) converted to pixels for real-world measurement

**Player Detection**
- `VNDetectHumanBodyPoseRequest` **[NEW]** identifies player body joints (shoulders, elbows, wrists, legs)
- Body pose observations stored continuously for later action classification input

**Trajectory Detection**
- `VNDetectTrajectoriesRequest` **[NEW]** — stateful request that builds evidence over multiple frames
- Detects parabolic motion; reports `VNTrajectoryObservation` with detected points, projected points, and `EquationCoefficients` (y = ax² + bx + c)
- `frameAnalysisSpacing` reduces computational cost on older devices
- `minimumObjectSize` / `maximumObjectSize` filter out irrelevant moving objects
- Multiple simultaneous trajectories differentiated by UUID; requires stable scene and timestamped `CMSampleBuffer`

**Action Classification**
- Custom Create ML Action Classification model consumes 45 frames of `VNHumanBodyPoseObservation` around throw detection point
- Body poses merged into `MLMultiArray` and fed to Core ML for throw type prediction (overhand, underhand, under-leg, or negative/other class)
- Prediction triggered at end of detected throw, not continuously

**Metrics Calculation**
- Release speed: trajectory length in pixels ÷ `VNTrajectoryObservation.timeRange.duration`, scaled via board pixel-to-meter mapping
- Release angle: angle from elbow to wrist joint relative to horizon at release frame

**Live Stream Best Practices**
- Return `CMSampleBuffer` to camera as soon as possible; process on separate queues
- Use `captureOutput(_:didDrop:)` to diagnose dropped frames
- `AVPlayerItemVideoOutput` + `CADisplayLink` for video playback analysis without stutter
- Render results asynchronously on main queue after releasing buffer

## APIs & Frameworks

### Vision (new in 2020)
- `VNDetectTrajectoriesRequest` **[NEW]** — stateful; detects parabolic trajectories in video
  - `init(frameAnalysisSpacing:trajectoryLength:completionHandler:)`
  - `minimumObjectSize` / `maximumObjectSize` — filter by object size
- `VNTrajectoryObservation` **[NEW]** — trajectory result
  - `detectedPoints` — centroids of moving object per frame
  - `projectedPoints` — five points defining the fitted parabola
  - `equationCoefficients` — coefficients a, b, c of y = ax² + bx + c
  - `timeRange` — when the trajectory occurred
  - `uuid` — unique identifier for multi-trajectory differentiation
- `VNDetectHumanBodyPoseRequest` **[NEW]** — detects body joint positions
- `VNHumanBodyPoseObservation` **[NEW]** — result with joint locations
- `VNDetectContoursRequest` — finds edges; `regionOfInterest` limits to bounding box
- `VNContoursObservation` — hierarchical contour result with `normalizedPath`
- `VNTranslationalImageRegistrationRequest` — measures inter-frame alignment for scene stability
- `VNCoreMLRequest` — runs Core ML model inference via Vision
- `VNSequenceRequestHandler` — performs requests across a series of frames
- `VNImageRequestHandler` — single-image request handler

### Create ML
- Object Detection template (transfer learning) — custom board detector
- Action Classification template — throw type classifier using body pose sequences

### Core ML
- `MLModel` — base model type
- `MLMultiArray` — input format for Action Classification (concatenated body pose observations)
- Auto-generated typed model wrappers from `.mlmodel` files

### AVFoundation
- `AVCaptureOutput.captureOutput(_:didDrop:from:)` — dropped frame diagnostics
- `AVPlayerItemVideoOutput` — video frame access for analysis
- `CADisplayLink` — display-sync timing for video playback analysis
- `CMSampleBuffer` — timestamped camera/video frame; must be released promptly

## Code Highlights

Trajectory detection setup (stateful request):
```swift
let detectTrajectoryRequest = VNDetectTrajectoriesRequest(
    frameAnalysisSpacing: .zero,
    trajectoryLength: 15
) { [weak self] request, error in
    self?.processTrajectoryObservations(request)
}
detectTrajectoryRequest.minimumObjectSize = 0.01
detectTrajectoryRequest.maximumObjectSize = 0.2
```

Action classification from stored body pose observations:
```swift
let observations = storedBodyPoseObservations  // [VNHumanBodyPoseObservation]
let input = prepareInputWithObservations(observations)
let prediction = try actionClassifier.prediction(from: input)
let throwType = prediction.labelProbabilities.max(by: { $0.value < $1.value })?.key
```

## Takeaways
- `VNDetectTrajectoriesRequest` is a stateful Vision request — keep a single instance alive across frames and feed it timestamped `CMSampleBuffer`s; it refines the trajectory observation with each new frame.
- Combining Object Detection, Contour Detection, Body Pose, and Trajectory Detection in a single pipeline enables rich sports analysis without any server-side processing.
- Always add a "negative/other" class to Action Classification models to prevent spurious predictions for non-target actions.
- For live camera pipelines, release `CMSampleBuffer` before rendering results and use dedicated queues for each analysis task to avoid starving the camera buffer pool.

---
_Source: WWDC20 Session 10099 page (abstract, chapter summaries, code samples, and resource links)._
