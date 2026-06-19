# Detect Body and Hand Pose with Vision
**WWDC20 · Session 10653** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10653/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, tvOS 14 (Vision; not watchOS)

## Overview
Vision framework gains two major new capabilities in iOS 14: hand pose detection and human body pose detection. Both APIs follow the same Vision request-handler pattern and return `VNRecognizedPointsObservation` objects containing per-landmark locations and confidence values. These features open the door to gesture-driven interactions, fitness/sports analysis, action classification with Create ML, and creative photo/video effects.

Hand pose returns 21 landmarks across fingers, thumb, and wrist, enabling apps to recognize custom gestures such as pinch-to-draw. Body pose returns up to 19 landmarks across the face, arms, torso, and legs, enabling applications such as ergonomics coaching, action-shot selection, and stromotion photography. Body pose from Vision is explicitly designed for offline image/video analysis and on any camera, contrasting with ARKit body pose which targets live rear-camera AR sessions.

## Key Topics

### Hand Pose Detection (VNDetectHumanHandPoseRequest)
- Returns up to `maximumHandCount` observations (default 2; tunable for performance).
- 21 landmarks per hand: wrist, 4 points per finger (TIP, DIP, PIP, MCP), 4 points for thumb (TIP, IP, MP, CMC).
- Landmarks grouped by `VNRecognizedPointGroupKey` (e.g., `.handLandmarkRegionKeyIndexFinger`, `.handLandmarkRegionKeyThumb`).
- Individual landmarks retrieved by `VNRecognizedPointKey` (e.g., `.handLandmarkKeyIndexTIP`, `.handLandmarkKeyThumbTIP`).
- Coordinates in Vision's normalized lower-left-origin space; convert to AVFoundation coordinates for camera overlays.
- `VNTrackObjectRequest` can be used after an initial hand detection to track hand location efficiently.
- Accuracy considerations: edge-of-frame occlusion, hands parallel to camera, gloves, and potential confusion with feet.

### Human Body Pose Detection (VNDetectHumanBodyPoseRequest)
- Analyzes multiple people simultaneously.
- Landmarks: face (nose, left/right eye, left/right ear), right arm (shoulder, elbow, wrist), left arm (shoulder, elbow, wrist), torso (neck, left/right shoulder, left/right hip, root), right leg (hip, knee, ankle), left leg (hip, knee, ankle).
- Shoulders appear in both arm and torso groups; hips appear in both torso and leg groups.
- Per-point confidence values provided (ARKit body pose does not provide confidence).
- Can be used on still images, video libraries, or live camera streams — not limited to AR sessions.
- Accuracy limitations: bent-over/upside-down subjects, occluding clothing, mutual occlusion between people, subjects near frame edges.

### Vision vs ARKit for Body Pose
- Vision: all supported platforms (except watchOS), any camera, still images or video, offline processing, per-point confidence.
- ARKit: live motion capture, rear camera only, supported iOS/iPadOS devices, within AR session, no per-point confidence.
- For ARKit use cases use ARKit; for all other use cases use Vision.

### Action Classification with Create ML
- Body pose observations from Vision can be fed to a Create ML action classifier.
- Collect 60-frame observation windows as a ring buffer; convert each `VNRecognizedPointsObservation` to `MLMultiArray` via `keypointsMultiArray()`.
- Pad with zeros if fewer than 60 frames are available.
- Concatenate arrays and pass to the action classifier's `prediction(from:)` method.
- Train and infer using Vision body pose consistently — mixing ARKit body pose with a Vision-trained model produces undefined results.
- On older devices, stagger classification inferences (a few times per second) to avoid starving the camera stream.
- Crop training videos to a single subject to avoid the default "largest person" selection behavior.

## APIs & Frameworks

### Vision **[NEW in iOS 14]**
- `VNDetectHumanHandPoseRequest` **[NEW]**
- `VNDetectHumanBodyPoseRequest` **[NEW]**
- `VNImageRequestHandler` — init with `CMSampleBuffer`, `CGImage`, or `URL`
- `VNRecognizedPointsObservation` — `.recognizedPoints(forGroupKey:)`, `.keypointsMultiArray()` **[NEW]**, `.confidence`
- `VNRecognizedPoint` — `.location`, `.confidence`
- `VNDetectedPoint` — subclass of `VNPoint` with confidence
- `VNPoint` — `.location: CGPoint`, `.x`, `.y`
- `VNRecognizedPointGroupKey` **[NEW]** — `.handLandmarkRegionKeyThumb`, `.handLandmarkRegionKeyIndexFinger`, `.handLandmarkRegionKeyMiddleFinger`, `.handLandmarkRegionKeyRingFinger`, `.handLandmarkRegionKeyLittleFinger`, `.all`; body groups: `.bodyLandmarkRegionKeyFace`, `.bodyLandmarkRegionKeyRightArm`, `.bodyLandmarkRegionKeyLeftArm`, `.bodyLandmarkRegionKeyTorso`, `.bodyLandmarkRegionKeyRightLeg`, `.bodyLandmarkRegionKeyLeftLeg`
- `VNRecognizedPointKey` **[NEW]** — hand: `.handLandmarkKeyThumbTIP`, `.handLandmarkKeyThumbIP`, `.handLandmarkKeyThumbMP`, `.handLandmarkKeyThumbCMC`, `.handLandmarkKeyIndexTIP`, `.handLandmarkKeyIndexDIP`, `.handLandmarkKeyIndexPIP`, `.handLandmarkKeyIndexMCP`, (and equivalent for middle, ring, little fingers), `.handLandmarkKeyWrist`; body: `.bodyLandmarkKeyNose`, `.bodyLandmarkKeyLeftEye`, `.bodyLandmarkKeyRightEye`, `.bodyLandmarkKeyLeftEar`, `.bodyLandmarkKeyRightEar`, `.bodyLandmarkKeyLeftShoulder`, `.bodyLandmarkKeyRightShoulder`, `.bodyLandmarkKeyNeck`, `.bodyLandmarkKeyLeftElbow`, `.bodyLandmarkKeyRightElbow`, `.bodyLandmarkKeyLeftWrist`, `.bodyLandmarkKeyRightWrist`, `.bodyLandmarkKeyLeftHip`, `.bodyLandmarkKeyRightHip`, `.bodyLandmarkKeyRoot`, `.bodyLandmarkKeyLeftKnee`, `.bodyLandmarkKeyRightKnee`, `.bodyLandmarkKeyLeftAnkle`, `.bodyLandmarkKeyRightAnkle`
- `VNDetectHumanHandPoseRequest.maximumHandCount` — default 2
- `VNTrackObjectRequest` — for tracking detected hands across frames

### Create ML
- Action classifier trained on body pose observations
- `MLMultiArray` — shape `[1, 3, 18]`, dataType `.double` for body pose keypoints
- `MLMultiArray(concatenating:axis:dataType:)` — merge per-frame arrays into window
- `MLModel.prediction(from:)` — run inference

### AVFoundation
- `CMSampleBuffer` — camera frame input to `VNImageRequestHandler`
- Coordinate conversion from Vision (lower-left origin) to AVFoundation (upper-left origin)

## Code Highlights

Performing hand pose detection on a camera frame:
```swift
let handler = VNImageRequestHandler(cmSampleBuffer: sampleBuffer, orientation: .up, options: [:])
try handler.perform([handPoseRequest])
guard let observation = handPoseRequest.results?.first as? VNRecognizedPointsObservation else { return }
let thumbPoints = try observation.recognizedPoints(forGroupKey: .handLandmarkRegionKeyThumb)
let indexFingerPoints = try observation.recognizedPoints(forGroupKey: .handLandmarkRegionKeyIndexFinger)
guard let thumbTipPoint = thumbPoints[.handLandmarkKeyThumbTIP],
      let indexTipPoint = indexFingerPoints[.handLandmarkKeyIndexTIP] else { return }
guard thumbTipPoint.confidence > 0.3 && indexTipPoint.confidence > 0.3 else { return }
```

Preparing 60-frame body pose window for action classification:
```swift
func prepareInputWithObservations(_ observations: [VNRecognizedPointsObservation]) -> MLMultiArray? {
    var multiArrayBuffer = [MLMultiArray]()
    for f in 0 ..< min(observations.count, 60) {
        if let arr = try? observations[f].keypointsMultiArray() {
            multiArrayBuffer.append(arr)
        }
    }
    // Pad with zeros if needed
    while multiArrayBuffer.count < 60 {
        if let zero = try? MLMultiArray(shape: [1, 3, 18], dataType: .double) {
            multiArrayBuffer.append(zero)
        }
    }
    return MLMultiArray(concatenating: multiArrayBuffer, axis: 0, dataType: .double)
}
```

## Takeaways
- `VNDetectHumanHandPoseRequest` and `VNDetectHumanBodyPoseRequest` are new in iOS 14 and follow the same Vision pattern as all other requests.
- Hand pose enables novel gesture-based interactions (pinch detection, gesture classification) without custom ML training.
- Body pose pairs naturally with Create ML action classifiers — always use Vision for both training and inference; never mix with ARKit body pose data.
- Tune `maximumHandCount` for performance; use `VNTrackObjectRequest` for efficient ongoing tracking after initial detection.

---
_Source: WWDC20 Session 10653 page (abstract, transcript, code samples, and resource links)._
