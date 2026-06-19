# Create Camera Extensions with Core Media IO
**WWDC22 · Session 10022** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10022/)

_Platforms:_ macOS Ventura 13

## Overview
This session introduces Camera Extensions, a modern replacement for the deprecated DAL (Device Abstraction Layer) plug-in architecture in macOS. Camera Extensions are built on top of the System Extensions framework and run in a sandboxed daemon process, eliminating the security vulnerabilities of DAL plug-ins that loaded untrusted code directly into app processes. They are fully compatible with AVFoundation and appear in all camera-using apps including Apple first-party apps like FaceTime, QuickTime Player, and PhotoBooth.

The session walks through three primary use cases: pure software cameras (e.g., test patterns, pre-rendered content), hardware camera drivers (USB/Bluetooth/Wi-Fi hardware with optional DriverKit extension), and creative cameras that intercept an existing camera stream and apply effects. A live demo builds a software camera from scratch using Xcode's Camera Extension target template, then extends it into a Core Image filter camera.

The Core Media IO Extension API is covered in depth: the four main classes (`CMIOExtensionProvider`, `CMIOExtensionDevice`, `CMIOExtensionStream`, and their corresponding source protocols), the property system (standard and custom properties bridged to the legacy C API), and output (sink) streams for pro video monitoring use cases.

## Key Topics

### Security Architecture
Extensions run as the `_cmiodalassistants` role user under a custom sandbox profile. A proxy service (`registerassistantservice`) handles TCC camera permission consent on behalf of the extension. Buffer validation occurs before delivery to any client app. Extensions cannot fork/exec child processes, access the window server, or register arbitrary Mach services.

### Deployment via System Extensions
Extensions are embedded in an app bundle and installed/uninstalled via `OSSystemExtensionRequest`. Deleting the host app automatically uninstalls the extension. App Store distribution is supported.

### Three Core Classes
- **CMIOExtensionProvider** — the root object; adds/removes devices, receives client connection events, published properties: `manufacturer`, `name`
- **CMIOExtensionDevice** — manages streams; `localizedName` and `deviceID` (UUID) map directly to `AVCaptureDevice.localizedName` and `AVCaptureDevice.uniqueIdentifier`; optional properties: `deviceModel`, `isSuspended`, `transportType`, `linkedDevices`
- **CMIOExtensionStream** — publishes `CMIOExtensionStreamFormat` objects (→ `AVCaptureDeviceFormat`); sends `CMSampleBuffer` to clients; supports `source` (output to clients) and `sink` (input from clients) directions

### Custom Properties
Custom properties use a naming scheme bridging to the legacy CoreMedia IO C property address (`4cc_<selector>_<scope>_<element>`). Custom property access requires the CoreMedia IO C API directly; AVFoundation does not expose them.

### Hardware Camera Support
Running a hardware camera (accessing another `AVCaptureDevice`) inside an extension requires macOS Ventura and the `com.apple.security.device.camera` entitlement plus `NSCameraUsageDescription` in `Info.plist`. USB hardware can be addressed via DriverKit (DEXT) or, for class-compliant cameras, Apple's built-in UVC extension (overridable via `IOKitPersonalities`).

### Output (Sink) Streams
Sink streams receive `CMSampleBuffer` objects from apps via a queue; the extension calls `notifyScheduledOutputChanged` after consuming each buffer. Used for print-to-tape, real-time monitoring, and similar pro video workflows.

### DAL Plug-in Deprecation
DAL plug-ins are deprecated as of macOS 12.3 and will be disabled entirely in the release after macOS Ventura. Developers should migrate existing DAL plug-ins to Camera Extensions.

## APIs & Frameworks

### CoreMediaIO — Camera Extension Classes
- `CMIOExtensionProvider` **[NEW]** — root extension object; call `startService(provider:)` from `main.swift`
- `CMIOExtensionProviderSource` protocol **[NEW]** — implement `manufacturer` and `name`
- `CMIOExtensionDevice` **[NEW]** — represents a logical camera device
- `CMIOExtensionDeviceSource` protocol **[NEW]** — optional properties: `deviceModel`, `isSuspended`, `transportType`, `linkedDevices`
- `CMIOExtensionStream` **[NEW]** — publishes formats and sends sample buffers
- `CMIOExtensionStreamSource` protocol **[NEW]** — implement to publish formats and handle format changes
- `CMIOExtensionStreamFormat` **[NEW]** — wraps a `CMFormatDescription` with valid frame durations
- `CMIOExtensionStream.send(_:discontinuity:hostTimeInNanoseconds:)` **[NEW]** — sends a `CMSampleBuffer` to clients
- `CMIOExtensionStream.consumeSampleBuffer(from:completionHandler:)` **[NEW]** — sink stream method to read buffers from clients
- `CMIOExtensionStream.notifyScheduledOutputChanged(_:)` **[NEW]** — notifies client after consuming a sink buffer
- `CMIOExtensionStreamDirection` **[NEW]** — `.source` (camera output) or `.sink` (output device input)
- `CMIOExtensionProperty` — property key type for standard and custom properties
- Custom property naming: `4cc_<selector>_<scope>_<element>` string format

### SystemExtensions Framework
- `OSSystemExtensionManager.shared.submitRequest(_:)` — installs or removes the extension
- `OSSystemExtensionRequest.activationRequest(forExtensionWithIdentifier:queue:)` — install request
- `OSSystemExtensionRequest.deactivationRequest(forExtensionWithIdentifier:queue:)` — uninstall request
- `OSSystemExtensionRequestDelegate` — receives activation/deactivation callbacks

### Required Entitlements & Info.plist Keys
- `com.apple.security.app-sandbox` — required
- `com.apple.security.application-groups` — app group shared between app and extension
- `CMIOExtensionMachServiceName` in `Info.plist` — required for `registerassistantservice` to launch the extension
- `com.apple.security.device.camera` — required if extension accesses another camera (macOS Ventura+)
- `NSCameraUsageDescription` — required if accessing another camera

## Code Highlights

Extension entry point:
```swift
// main.swift
let providerSource = ExtensionProviderSource()
CMIOExtensionProvider.startService(provider: providerSource.provider)
CFRunLoopRun()
```

Installing the extension from the host app:
```swift
let request = OSSystemExtensionRequest.activationRequest(
    forExtensionWithIdentifier: "com.example.ExampleCam.Extension",
    queue: .main)
request.delegate = self
OSSystemExtensionManager.shared.submitRequest(request)
```

Sending a frame to clients:
```swift
try stream.send(sampleBuffer, discontinuity: [], hostTimeInNanoseconds: hostTime)
```

## Takeaways
- Camera Extensions replace DAL plug-ins with a sandboxed, App Store-distributable architecture compatible with all macOS camera apps including Apple first-party apps.
- The Xcode "Camera Extension" target template provides a fully working software camera in minutes; customization is limited to implementing `CMIOExtensionProviderSource`, `CMIOExtensionDeviceSource`, and `CMIOExtensionStreamSource`.
- DAL plug-ins will be disabled entirely after macOS Ventura — begin porting existing plug-ins now.
- Hardware camera access within an extension requires macOS Ventura and the `com.apple.security.device.camera` entitlement; for custom USB protocols, override Apple's UVC extension via `IOKitPersonalities`.

---
_Source: WWDC22 Session 10022 page (abstract, chapter summaries, code samples, and resource links)._
