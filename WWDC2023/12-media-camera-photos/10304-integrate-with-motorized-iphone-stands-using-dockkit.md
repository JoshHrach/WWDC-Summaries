# Integrate with Motorized iPhone Stands Using DockKit
**WWDC23 · Session 10304** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10304/)

_Platforms:_ iOS 17

## Overview
DockKit is a new framework introduced in 2023 that enables iPhone to serve as the compute hub for compatible motorized camera stands. These stands provide 360-degree pan and 90-degree tilt, and DockKit handles subject tracking entirely on-device using the existing camera processing pipeline — meaning any app that uses iOS camera APIs automatically benefits from subject tracking without additional code.

For developers who want to go further, DockKit provides APIs to customize framing alignment, set regions of interest, supply custom inference observations (using Vision, Create ML, or any other ML model), directly control the stand's motors, and trigger built-in or custom animations that use motor movement as an expressive affordance.

## Key Topics

### How DockKit Works Out of the Box
- Motorized stands pair with iPhone via the DockKit protocol; DockKit daemon manages communication.
- Camera frames analyzed via ISP inference at 30 fps; bounding boxes for faces and bodies generated and fed to a multi-subject tracker.
- Statistical EKF filter smooths gaps and errors from inference.
- System-level tracker determines the primary subject and drives motors to keep them centered.
- Switching between front and rear cameras causes the stand to rotate 180 degrees automatically.
- Any camera-using app benefits without code changes (demo: FILMICPRO from the App Store).

### Obtaining a Dock Reference and State
- Register for dock accessory state changes via `DockAccessoryManager`.
- `DockAccessory.StateEvent` — `.docked` and `.undocked` states; must be in `.docked` state before customizing behavior.
- State events also expose the stand's tracking button state.

### Framing Control
- `DockAccessory.FramingMode` — `.left`, `.center` (default), `.right` — controls alignment of automatic framing.
- Region of interest: set a normalized `CGRect` on the dock accessory to constrain the framing area (useful for square-crop video conferencing apps).
- Origin for region of interest: upper-left corner of the iPhone display; normalized coordinates.

### Custom Inference / Observations
- Disable system tracking before supplying custom observations: `dockAccessory.trackingEnabled = false`.
- `DockAccessory.Observation` — created from a normalized `CGRect` bounding box (origin: lower-left); specify type as `.humanFace` or `.object`.
- Using `.humanFace` type preserves system-level multi-person framing optimizations.
- Pass observations with camera info to `dockAccessory.observations` (or equivalent method).
- Vision framework bounding boxes use the same coordinate system as DockKit — pass directly with device orientation set to `.corrected`.
- Supported Vision requests: `VNDetectHumanBodyPoseRequest`, `VNDetectAnimalBodyPoseRequest`, `VNDetectHumanHandPoseRequest`, `VNDetectBarcodesRequest`, face detection, etc.

### Direct Motor Control
- Disable system tracking first.
- Motor axes: Pitch (X-axis, tilt) and Yaw (Y-axis, pan).
- Velocity vector specifies rotation speed in radians per second for each axis.
- `dockAccessory.setVelocity(_:)` — sends a velocity command to the stand's motors.

### Device Animations
- Built-in named animations: `.yes`, `.no`, `.wakeup`, `.kapow`.
- `dockAccessory.startAnimation(_:)` — initiates an animation from the stand's current position; executes asynchronously.
- Custom animations: achieve via sequences of motor control calls.
- Re-enable system tracking after animation completes.
- Combine with Create ML hand action classifier to trigger animations on specific gestures.

## APIs & Frameworks
- `DockKit` framework **[NEW]** — iPhone-to-motorized-stand communication and subject tracking
- `DockAccessoryManager` **[NEW]** — singleton manager for dock state and accessory reference
- `DockAccessoryManager.StateEvent` **[NEW]** — async stream of dock state events (`.docked`, `.undocked`)
- `DockAccessory` **[NEW]** — represents a connected motorized stand
- `DockAccessory.trackingEnabled` **[NEW]** — boolean to enable/disable system tracking
- `DockAccessory.FramingMode` **[NEW]** — framing alignment enum: `.left`, `.center`, `.right`
- `DockAccessory.regionOfInterest` **[NEW]** — normalized `CGRect` constraining the tracked region
- `DockAccessory.Observation` **[NEW]** — bounding box observation; initialized with normalized `CGRect` and type (`.humanFace` or `.object`)
- `DockAccessory.Observation.ObservationType.humanFace` **[NEW]** — enables system multi-person framing optimizations
- `DockAccessory.Observation.ObservationType.object` **[NEW]** — generic object tracking
- `DockAccessory.CameraInformation` **[NEW]** — camera orientation/transform metadata passed with observations
- `DockAccessory.CameraInformation.Orientation.corrected` **[NEW]** — no coordinate transform needed (bottom-left origin)
- `DockAccessory.setVelocity(_:)` **[NEW]** — sends motor velocity command (radians/second for pitch and yaw)
- `DockAccessory.startAnimation(_:)` **[NEW]** — triggers a built-in animation
- `DockAccessory.Animation` **[NEW]** — built-in animation enum: `.yes`, `.no`, `.wakeup`, `.kapow`
- `Vision` framework — custom inference; `VNDetectHumanHandPoseRequest`, `VNDetectHumanBodyPoseRequest`, `VNDetectAnimalBodyPoseRequest`, `VNDetectBarcodesRequest`
- `Create ML` — train custom hand action classifier models for gesture-triggered animations
- `AVFoundation` — camera capture; DockKit integrates with standard camera pipeline

## Code Highlights

Direct motor control (pan right and tilt down, then reverse):
```swift
// Move right and tilt down
var velocity = DockAccessory.MotorVelocity(yaw: 0.2, pitch: -0.1)
try await dockAccessory.setVelocity(velocity)
try await Task.sleep(for: .seconds(2))

// Reverse direction
velocity = DockAccessory.MotorVelocity(yaw: -0.2, pitch: 0.1)
try await dockAccessory.setVelocity(velocity)
```

Custom hand-tracking observations fed to DockKit:
```swift
let request = VNDetectHumanHandPoseRequest()
let handler = VNImageRequestHandler(cvPixelBuffer: frame)
try handler.perform([request])

let thumbTip = try request.results?.first?.recognizedPoint(.thumbTip)
let boundingBox = CGRect(x: thumbTip.x, y: thumbTip.y, width: 0.05, height: 0.05)
let observation = DockAccessory.Observation(boundingBox: boundingBox, type: .humanFace)
let cameraInfo = DockAccessory.CameraInformation(orientation: .corrected)
try await dockAccessory.track([observation], cameraInformation: cameraInfo)
```

Triggering a built-in animation on a custom gesture:
```swift
dockAccessory.trackingEnabled = false
try await dockAccessory.startAnimation(.kapow)
dockAccessory.trackingEnabled = true
```

## Takeaways
- Any camera app gains 360-degree automated subject tracking for free when the user connects a DockKit stand — no code changes needed.
- DockKit's framing mode and region of interest APIs are simple one-liners that enable pixel-perfect composition for custom overlays and non-standard aspect ratios.
- Custom observations from Vision (or any other source) can replace or supplement the system tracker, enabling tracking of hands, animals, barcodes, or arbitrary objects.
- Motor animations (`.yes`, `.no`, `.wakeup`, `.kapow`) can serve as expressive affordances — triggered by gestures via a Create ML hand action classifier.

---
_Source: WWDC23 Session 10304 page (abstract, chapter summaries, code samples, and resource links)._
