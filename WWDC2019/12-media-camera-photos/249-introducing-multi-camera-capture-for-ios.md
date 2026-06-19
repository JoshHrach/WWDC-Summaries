# Introducing Multi-Camera Capture for iOS
**WWDC19 · Session 249** · [Watch](https://developer.apple.com/videos/play/wwdc2019/249/)

_Platforms:_ iOS 13 (iPhone XS, iPhone XS Max, iPhone XR, iPad Pro — latest generation)

## Overview
iOS 13 introduces multi-camera capture via `AVCaptureMultiCamSession`, a new subclass of `AVCaptureSession` that allows simultaneous video, photo, audio, metadata, and depth capture from multiple cameras and microphones. Previously limited to one active camera per session on iOS (multi-camera was already supported on macOS since Lion), this capability is now available on devices with newer image signal processors that resolve hardware power-sharing limitations.

The session covers three main capabilities: the `AVCaptureMultiCamSession` API for multi-camera pipelines, synchronized streaming from virtual device constituent cameras (dual camera wide + tele simultaneously), and multi-microphone capture with simultaneous front and back beam forming. Hardware cost and system pressure cost APIs provide upfront budgeting to help apps avoid thermal throttling or session interruption.

A companion sample app, AVMultiCamPiP, demonstrates picture-in-picture capture from front and back cameras simultaneously, compositing two streams in real time via Metal and recording to a single movie file.

## Key Topics

**AVCaptureMultiCamSession**
A new subclass of `AVCaptureSession` that allows multiple camera inputs. The existing `AVCaptureSession` is not deprecated and remains preferred for single-camera scenarios. Key differences from `AVCaptureSession`:
- Uses `addInput(withNoConnections:)` and `addOutput(withNoConnections:)` to prevent implicit connection forming, then manually creates `AVCaptureConnection` objects to wire specific input ports to specific outputs.
- Does not support `AVCaptureSession` presets; only `.inputPriority` is supported. Active formats must be configured manually.
- Limitation: one input per physical camera per session; cannot fan out one camera to multiple outputs of the same type; outputs cannot receive from multiple cameras (no mixing).
- Use `AVCaptureMultiCamSession.isMultiCamSupported` class property to check device support.
- Use `AVCaptureDevice.DiscoverySession.supportedMultiCamDeviceSets` to enumerate valid multi-camera combinations.

**Hardware Cost**
The ISP (image signal processor) is a shared, single resource. `AVCaptureMultiCamSession.hardwareCost` (0.0–1.0+) tallies ISP pixel bandwidth. Cost is determined by resolution, format binning mode, and max frame rate of the format (not the active frame rate set on the device). To reduce cost: pick lower resolution, use binned formats (trade image quality for 4x bandwidth reduction), or use `AVCaptureDeviceInput.videoMinFrameDurationOverride` to promise a lower max frame rate and pay only that cost.

**System Pressure Cost**
`AVCaptureMultiCamSession.systemPressureCost` reports the expected thermal impact independent of other system activity. Values: < 1.0 = indefinite; 1–2 = ~15 min; 2–3 = ~10 min; > 3 = short duration. At runtime, reduce frame rate immediately if system pressure elevates. As a last resort, disable individual cameras using `AVCaptureInputPort.isEnabled = false` on all ports of that camera's input without stopping other cameras.

**Virtual Devices and Synchronized Streaming**
"Virtual cameras" are software cameras backed by multiple physical sensors: `DualCamera` (wide + tele), `TrueDepth` (infrared + RGB). New in iOS 13:
- `AVCaptureDevice.isVirtualDevice` property identifies virtual cameras.
- `AVCaptureDevice.constituentDevices` returns the underlying physical cameras.
- Virtual device input exposes "secret" ports (not in `ports` array) accessed by querying `inputPorts(for:sourceDeviceType:position:)` — specifying source device type (e.g., `.builtInWideAngleCamera`, `.builtInTelephotoCamera`).
- These ports deliver hardware-synchronized frames (sensor readout midpoints aligned) with matched exposure, white balance, and focus.
- `AVCaptureDataOutputSynchronizer` consolidates multi-output callbacks into a single callback per timestamp.

**Camera Intrinsics and Extrinsics**
- `AVCaptureConnection.isCameraIntrinsicMatrixDeliverySupported` / `isCameraIntrinsicMatrixDeliveryEnabled` — per-frame 3×3 camera intrinsic matrix as `CVPixelBuffer` attachment (existing since iOS 11).
- New in iOS 13: `AVCaptureDevice.extrinsicMatrix(relativeTo:)` — 3×4 rotation+translation matrix describing pose of one camera relative to another. Used for image rectification and depth-convergence plane adjustment (demonstrated in AVDualCam sample).

**Multi-Microphone Capture**
iPhone microphones are all omnidirectional; directional pickup is achieved via software beam forming (Core Audio). `AVCaptureMultiCamSession` allows simultaneous front and back beam forming for the first time:
- Omni port: first audio port from the device input (`AVMediaType.audio`).
- Back beam port: queried via `inputPorts(for:sourceDeviceType:position:)` with `.back` position.
- Front beam port: queried with `.front` position.
- Beam forming requires built-in microphones; external microphones (USB, AirPods) are passed through to all connected outputs as-is.

**Supported Device Combinations (iPhone XS)**
Six combinations allowed, all involving exactly two physical cameras (DualCamera counts as two). Consult `AVCaptureDevice.DiscoverySession.supportedMultiCamDeviceSets` at runtime.

**Supported Formats**
MultiCam-compatible formats are flagged via `AVCaptureDeviceFormat.isMultiCamSupported`. Supported formats on iPhone XS include: binned formats (640×480 to 1920×1440, up to 60 FPS), 1920×1080 unbinned at 30 FPS, and 1920×1440 unbinned at 30 FPS (the "photo proxy" format enabling 12MP stills).

## APIs & Frameworks

**AVFoundation** (iOS 13) **[NEW]**

Session:
- `AVCaptureMultiCamSession` — new multi-camera session subclass **[NEW]**
- `AVCaptureMultiCamSession.isMultiCamSupported: Bool` (class property) **[NEW]**
- `AVCaptureMultiCamSession.hardwareCost: Float` **[NEW]**
- `AVCaptureMultiCamSession.systemPressureCost: Float` **[NEW]**
- `AVCaptureSession.addInput(withNoConnections:)` **[NEW]**
- `AVCaptureSession.addOutput(withNoConnections:)` **[NEW]**
- `AVCaptureVideoPreviewLayer.setSessionWithNoConnections(_:)` **[NEW]**

Device and format:
- `AVCaptureDevice.isVirtualDevice: Bool` **[NEW]**
- `AVCaptureDevice.constituentDevices: [AVCaptureDevice]` **[NEW]**
- `AVCaptureDeviceFormat.isMultiCamSupported: Bool` **[NEW]**
- `AVCaptureDevice.DiscoverySession.supportedMultiCamDeviceSets: [Set<AVCaptureDevice>]` **[NEW]**
- `AVCaptureDevice.extrinsicMatrix(relativeTo:) -> matrix_float4x3?` **[NEW]**
- `AVCaptureDeviceInput.videoMinFrameDurationOverride: CMTime` **[NEW]**
- `AVCaptureInputPort.isEnabled: Bool` — disable individual camera without stopping session **[existing, now useful for MultiCam power management]**

Input ports (virtual device constituent access):
- `AVCaptureDeviceInput.inputPorts(for:sourceDeviceType:position:) -> [AVCaptureInputPort]` **[NEW]** (access constituent and beam-form ports)

Connection and synchronizer:
- `AVCaptureConnection.init(inputPorts:output:)` — manual connection creation **[NEW]**
- `AVCaptureDataOutputSynchronizer` — single callback for multi-output frames at same timestamp **[existing, newly relevant]**
- `AVCaptureConnection.isCameraIntrinsicMatrixDeliveryEnabled: Bool` (existing since iOS 11)

Output:
- `AVCaptureVideoDataOutput`, `AVCaptureAudioDataOutput`, `AVCapturePhotoOutput`, `AVCaptureMovieFileOutput`, `AVCaptureMetadataOutput`, `AVCaptureDepthDataOutput` — all usable in MultiCam sessions

System pressure (existing since iOS 11, extended):
- `AVCaptureDevice.systemPressureState` — `.nominal`, `.fair`, `.serious`, `.critical`, `.shutdown`

## Code Highlights

Setting up AVCaptureMultiCamSession with front and back cameras:
```swift
let session = AVCaptureMultiCamSession()

// Add inputs without implicit connections
let backInput = try AVCaptureDeviceInput(device: backCamera)
session.addInput(withNoConnections: backInput)

let frontInput = try AVCaptureDeviceInput(device: frontCamera)
session.addInput(withNoConnections: frontInput)

// Add outputs without connections
let backOutput = AVCaptureVideoDataOutput()
session.addOutput(withNoConnections: backOutput)

let frontOutput = AVCaptureVideoDataOutput()
session.addOutput(withNoConnections: frontOutput)

// Manually wire ports to outputs
let backPort = backInput.ports(for: .video, sourceDeviceType: .builtInWideAngleCamera, position: .back).first!
session.addConnection(AVCaptureConnection(inputPorts: [backPort], output: backOutput))

let frontPort = frontInput.ports(for: .video, sourceDeviceType: .builtInWideAngleCamera, position: .front).first!
session.addConnection(AVCaptureConnection(inputPorts: [frontPort], output: frontOutput))
```

Accessing dual camera constituent streams:
```swift
let dualInput = try AVCaptureDeviceInput(device: dualCamera)
let widePort = dualInput.ports(for: .video, sourceDeviceType: .builtInWideAngleCamera, position: .back).first!
let telePort = dualInput.ports(for: .video, sourceDeviceType: .builtInTelephotoCamera, position: .back).first!
```

Overriding frame rate to reduce hardware cost:
```swift
let thirtyFPS = CMTime(value: 1, timescale: 30)
backInput.videoMinFrameDurationOverride = thirtyFPS
```

Setting up front and back beam-form microphone ports:
```swift
let backAudioPort = micInput.ports(for: .audio, sourceDeviceType: .builtInMicrophone, position: .back).first!
let frontAudioPort = micInput.ports(for: .audio, sourceDeviceType: .builtInMicrophone, position: .front).first!
```

## Takeaways
- `AVCaptureMultiCamSession` is a power tool requiring explicit connection management; never add inputs/outputs with implicit connections when using multi-camera.
- Always check `hardwareCost` and `systemPressureCost` before running — exceeding hardware cost 1.0 causes an immediate session stop with no grace period.
- Binned formats are the most cost-effective choice for multi-camera; use `isMultiCamSupported` to enumerate compatible formats rather than assuming.
- Synchronized streaming from virtual device constituent cameras (wide + tele) and simultaneous front/back beam forming open entirely new creative possibilities for camera and AR apps.

---
_Source: WWDC19 Session 249 page (abstract, chapter summaries, code samples, and resource links)._
