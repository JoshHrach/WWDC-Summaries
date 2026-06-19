# Explore 3D Body Pose and Person Segmentation in Vision
**WWDC23 · Session 111241** · [Watch](https://developer.apple.com/videos/play/wwdc2023/111241/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14

## Overview
This session introduces two major new person-centric capabilities in the Vision framework: 3D Human Body Pose detection and per-instance person segmentation. The 3D body pose request expands on the existing 2D request to return joint positions in meters in real-world space without requiring ARKit or an AR session. The person instance mask request builds on the existing single-mask person segmentation to support up to four individually selectable person masks.

The session also covers Vision's new support for `AVDepthData` as an input to image request handlers, which enables more accurate body height estimation and unlocks richer 3D reconstruction. Detailed code examples show how to access 3D joint positions, compute joint angles, project 3D points back into 2D image coordinates, and retrieve the camera's position from the observation.

## Key Topics

- **VNDetectHumanBodyPose3DRequest** — New request returning a `VNHumanBodyPose3DObservation` with 17 joints positioned in meters relative to a root joint at the center of the hip; coordinates follow the same `simd_float4x4` convention as ARKit; processes the most prominent person in the frame.
- **3D skeleton structure** — Head group (center/top of head), torso group (shoulders, spine, root/hip, hip joints), left/right arm groups (shoulder, elbow, wrist), left/right leg groups (hip, knee, ankle); joints accessible individually or by group.
- **Depth data in Vision** — New `VNImageRequestHandler` initializers accepting `AVDepthData` alongside `CVPixelBuffer` or `CMSampleBuffer`; files with embedded depth (Portrait mode) loaded automatically; enables accurate body height measurement vs. reference height of 1.8 m.
- **VNHumanBodyPose3DObservation properties** — `recognizedPoint(_:)` / `recognizedPoints(_:)` for joint access; `bodyHeight` (estimated in meters); `heightEstimation` (`.measured` or `.reference`); `cameraOriginMatrix` (camera position as `simd_float4x4`); `pointInImage(_:)` to project 3D joints to 2D image coordinates; `localPosition` on each point (position relative to parent joint).
- **Person instance segmentation** — New `VNGeneratePersonInstanceMaskRequest` returning up to four individual `VNInstanceMaskObservation` objects each with a confidence score; instance 0 = background; supports selecting specific individuals; falls back to `VNGeneratePersonSegmentationRequest` for crowded scenes.

## APIs & Frameworks

**Vision**
- `VNDetectHumanBodyPose3DRequest` **[NEW]** — requests 3D human body pose
- `VNHumanBodyPose3DObservation` **[NEW]** — observation containing 3D skeleton
- `VNHumanBodyPose3DObservation.recognizedPoint(_:)` **[NEW]** — get a single `VNHumanBodyRecognizedPoint3D` by joint name
- `VNHumanBodyPose3DObservation.recognizedPoints(_:)` **[NEW]** — get joint collection by group name
- `VNHumanBodyPose3DObservation.bodyHeight` **[NEW]** — estimated subject height in meters
- `VNHumanBodyPose3DObservation.heightEstimation` **[NEW]** — `.measured` (with depth) or `.reference` (1.8 m default)
- `VNHumanBodyPose3DObservation.cameraOriginMatrix` **[NEW]** — `simd_float4x4` camera position/orientation
- `VNHumanBodyPose3DObservation.pointInImage(_:)` **[NEW]** — projects 3D joint to 2D normalized image coordinates
- `VNPoint3D` **[NEW]** — base class; stores position as `simd_float4x4`
- `VNRecognizedPoint3D` **[NEW]** — inherits `VNPoint3D`; adds `identifier`
- `VNHumanBodyRecognizedPoint3D` **[NEW]** — inherits `VNRecognizedPoint3D`; adds `localPosition` (relative to parent joint) and `parentJoint`
- `VNHumanBodyPose3DObservation.JointName` **[NEW]** — typed joint name enum (e.g., `.leftWrist`, `.rightAnkle`, `.centerShoulder`, `.root`)
- `VNHumanBodyPose3DObservation.JointsGroupName` **[NEW]** — group names: `.head`, `.torso`, `.leftArm`, `.rightArm`, `.leftLeg`, `.rightLeg`
- `VNGeneratePersonInstanceMaskRequest` **[NEW]** — per-instance person segmentation returning up to 4 masks
- `VNInstanceMaskObservation` **[NEW]** — individual person mask with confidence score; instance 0 = background
- `VNGeneratePersonSegmentationRequest` — existing; still used for scenes with more than 4 people
- `VNImageRequestHandler` — existing; new initializers accepting `AVDepthData` parameter **[NEW]**
- `VNImageRequestHandler(cvPixelBuffer:depthData:orientation:options:)` **[NEW]**
- `VNImageRequestHandler(cmSampleBuffer:depthData:orientation:options:)` **[NEW]**

**AVFoundation**
- `AVDepthData` — container for depth map (disparity or depth format) and camera calibration metadata; passed to new `VNImageRequestHandler` initializers

## Code Highlights

Running the 3D body pose request:
```swift
let request = VNDetectHumanBodyPose3DRequest()
let handler = VNImageRequestHandler(cvPixelBuffer: pixelBuffer,
                                    depthData: avDepthData,
                                    orientation: .up,
                                    options: [:])
try handler.perform([request])
guard let observation = request.results?.first else { return }

let leftWrist = try observation.recognizedPoint(.leftWrist)
print("Height:", observation.bodyHeight)         // meters
print("Estimation:", observation.heightEstimation) // .measured or .reference
```

Accessing joint model position and local position:
```swift
let wristPosition = leftWrist.position          // simd_float4x4, relative to root
let wristLocal = leftWrist.localPosition        // simd_float4x4, relative to parent (elbow)
let yAboveHip = wristPosition.columns.3.y       // ~0.9 m for raised arm pose
```

Per-instance person segmentation:
```swift
let instanceRequest = VNGeneratePersonInstanceMaskRequest()
// To get all people: instanceRequest.instanceCount = .all
try handler.perform([instanceRequest])
if let obs = instanceRequest.results?.first {
    // obs.allInstances returns array of instance IDs
    // instance 0 = background, 1..4 = individuals
    let mask = try obs.generateScaledMaskForImage(forInstances: [1], from: handler)
}
```

## Takeaways

- `VNDetectHumanBodyPose3DRequest` provides 3D body pose from a still image without ARKit — pass `AVDepthData` from a Portrait photo or LiDAR capture for accurate metric measurements.
- Use `localPosition` on a `VNHumanBodyRecognizedPoint3D` when working with a specific body region or computing joint angles; use `position` (relative to root/hip) for full-body scene placement.
- `VNGeneratePersonInstanceMaskRequest` returns up to four individually selectable person masks — use `VNGeneratePersonSegmentationRequest` as a fallback when more than four people are present.
- Project 3D joints back to 2D with `pointInImage(_:)` to overlay skeleton visualization on the original image.

---
_Source: WWDC23 Session 111241 page (abstract, chapter summaries, code samples, and resource links)._
