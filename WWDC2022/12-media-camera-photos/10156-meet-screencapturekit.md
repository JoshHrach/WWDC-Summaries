# Meet ScreenCaptureKit
**WWDC22 · Session 10156** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10156/)

_Platforms:_ macOS Ventura 13

## Overview
ScreenCaptureKit is a brand-new macOS framework introduced at WWDC22 for building high-performance screen capture experiences in screen sharing, video conferencing, game streaming, and content creation apps. It replaces the older `CGDisplayStream` and `CGWindowList` APIs (which will be deprecated) with a modern, Swift-native, async API that delivers video at up to native display resolution and frame rate and audio at up to 48 kHz stereo, using GPU acceleration with lower CPU overhead than existing capture methods.

The framework gives developers fine-grained control over what is captured (specific displays, windows, and applications can be included or excluded), how it is captured (resolution, frame rate, pixel format, color space, cursor visibility, audio sample rate, channel count), and all of these parameters can be changed live while a stream is running.

## Key Topics
- **`SCShareableContent`** — enumerates all capturable content on the system (displays, running applications, windows); async class method with filtering options for on-screen-only windows and desktop windows
- **`SCDisplay`** — represents a display; properties: `displayID`, `width`, `height`
- **`SCRunningApplication`** — represents a running app; properties: `bundleIdentifier`, `applicationName`, `processID`
- **`SCWindow`** — represents a window; properties: `windowID`, `frame`, `title`, `isOnScreen`, `isMinimized`, `owningApplication`
- **`SCContentFilter`** — specifies what to capture; two main modes: (1) display-independent window capture (follows window across displays), (2) display-dependent capture with application/window include or exclude lists; audio filtering is application-level only
- **`SCStreamConfiguration`** — controls quality and format; key properties: `width`/`height` (output resolution), `minimumFrameInterval` (`CMTime`, controls frame rate), `showsCursor` (Bool), `capturesAudio` (Bool), `sampleRate` (Int), `channelCount` (Int); all can be updated on a running stream
- **`SCStream`** — the central capture stream object; initialized with `SCContentFilter`, `SCStreamConfiguration`, and an optional error-handling delegate; started with `startCapture()` (async); stopped with `stopCapture()`
- **`SCStreamOutput` protocol** — add one or more output objects to an `SCStream` with `addStreamOutput(_:type:sampleHandlerQueue:)`; receives `CMSampleBuffer` objects for `.screen` and `.audio` types; `sampleHandlerQueue` allows delivery on a specific dispatch queue
- **`SCStreamOutputType`** — `.screen` for video frames, `.audio` for audio samples
- **`SCStreamFrameInfo`** — `CMSampleBuffer` attachment dictionary providing frame-level metadata; key field: `SCStreamFrameInfoStatus` — `.complete` means new video frame with new `IOSurface`, `.idle` means content unchanged (no new surface)
- **CMSampleBuffer / IOSurface** — video sample buffers are `IOSurface`-backed; audio sample buffers are standard `CMSampleBuffer`; standard CoreMedia utilities apply
- **Privacy model** — requires Screen Recording permission (stored in System Preferences → Security & Privacy); system prompts on first use; applies globally to all apps using ScreenCaptureKit

## APIs & Frameworks
**ScreenCaptureKit** **[NEW]** — `import ScreenCaptureKit`
- `SCShareableContent` **[NEW]**
  - `SCShareableContent.excludingDesktopWindows(_:onScreenWindowsOnly:) async throws -> SCShareableContent` — enumerates capturable content
  - `content.displays: [SCDisplay]`
  - `content.applications: [SCRunningApplication]`
  - `content.windows: [SCWindow]`
- `SCDisplay` **[NEW]** — `displayID: CGDirectDisplayID`, `width: Int`, `height: Int`
- `SCRunningApplication` **[NEW]** — `bundleIdentifier: String`, `applicationName: String`, `processID: pid_t`
- `SCWindow` **[NEW]** — `windowID: CGWindowID`, `frame: CGRect`, `title: String?`, `isOnScreen: Bool`, `isMinimized: Bool`, `owningApplication: SCRunningApplication?`
- `SCContentFilter` **[NEW]**
  - `init(desktopIndependentWindow: SCWindow)` — capture a single window regardless of display
  - `init(display: SCDisplay, excludingApplications: [SCRunningApplication], exceptingWindows: [SCWindow])` — capture display minus specified apps/windows
  - `init(display: SCDisplay, including applications: [SCRunningApplication], exceptingWindows: [SCWindow])` — capture only specified apps
- `SCStreamConfiguration` **[NEW]**
  - `width: Int`, `height: Int` — output video resolution
  - `minimumFrameInterval: CMTime` — minimum time between frames; `CMTime(value:1, timescale:60)` = 60 fps
  - `showsCursor: Bool` — include mouse cursor in capture
  - `capturesAudio: Bool` — enable audio capture
  - `sampleRate: Int` — audio sample rate (e.g., 48000)
  - `channelCount: Int` — audio channel count (e.g., 2 for stereo)
  - `pixelFormat: OSType`, `colorSpaceName: CFString` — pixel and color configuration
- `SCStream` **[NEW]**
  - `init(filter: SCContentFilter, configuration: SCStreamConfiguration, delegate: SCStreamDelegate?)` — create stream
  - `startCapture() async throws` — begin capture
  - `stopCapture() async throws` — end capture
  - `updateContentFilter(_ filter: SCContentFilter) async throws` — live filter update
  - `updateConfiguration(_ configuration: SCStreamConfiguration) async throws` — live configuration update
  - `addStreamOutput(_ output: SCStreamOutput, type: SCStreamOutputType, sampleHandlerQueue: DispatchQueue?) throws`
  - `removeStreamOutput(_ output: SCStreamOutput, type: SCStreamOutputType) throws`
- `SCStreamDelegate` protocol **[NEW]** — `stream(_:didStopWithError:)` for error handling
- `SCStreamOutput` protocol **[NEW]** — `stream(_:didOutputSampleBuffer:of:)` for receiving samples
- `SCStreamOutputType` **[NEW]** — `.screen`, `.audio`
- `SCStreamFrameInfo` **[NEW]** — `CMSampleBuffer` attachment keys; `SCStreamFrameInfoStatus` enum with `.complete`, `.idle`, `.started`, `.stopped`

**Deprecated (replacing these)**
- `CGDisplayStream` — deprecated in favor of ScreenCaptureKit
- `CGWindowList` / `CGWindowListCreateImage` — deprecated in favor of ScreenCaptureKit

## Code Highlights
Enumerating shareable content and creating a display filter that excludes the capture app:
```swift
let content = try await SCShareableContent.excludingDesktopWindows(false, onScreenWindowsOnly: true)

let excludedApps = content.applications.filter { app in
    Bundle.main.bundleIdentifier == app.bundleIdentifier
}

let filter = SCContentFilter(display: display,
                             excludingApplications: excludedApps,
                             exceptingWindows: [])
```

Configuring a stream for high-motion content at 1080p/60fps with audio:
```swift
let streamConfig = SCStreamConfiguration()
streamConfig.width = 1920
streamConfig.height = 1080
streamConfig.minimumFrameInterval = CMTime(value: 1, timescale: 60)
streamConfig.showsCursor = false
streamConfig.capturesAudio = true
streamConfig.sampleRate = 48000
streamConfig.channelCount = 2
```

Creating, starting, and adding outputs:
```swift
let stream = SCStream(filter: filter, configuration: streamConfig, delegate: self)

try stream.addStreamOutput(self, type: .screen, sampleHandlerQueue: screenQueue)
try stream.addStreamOutput(self, type: .audio,  sampleHandlerQueue: audioQueue)

try await stream.startCapture()
```

Receiving sample buffers:
```swift
func stream(_ stream: SCStream, didOutputSampleBuffer sampleBuffer: CMSampleBuffer,
            of type: SCStreamOutputType) {
    switch type {
    case .screen: handleLatestScreenSample(sampleBuffer)
    case .audio:  handleLatestAudioSample(sampleBuffer)
    @unknown default: break
    }
}
```

Checking frame status on a video sample buffer:
```swift
guard let attachments = CMSampleBufferGetSampleAttachmentsArray(sampleBuffer, createIfNecessary: false)
        as? [[SCStreamFrameInfo: Any]],
      let statusRawValue = attachments.first?[.status] as? Int,
      let status = SCFrameStatus(rawValue: statusRawValue),
      status == .complete else { return }
// New IOSurface available — process the frame
```

## Takeaways
- ScreenCaptureKit replaces `CGDisplayStream` and `CGWindowListCreateImage` with a modern Swift API offering live-updatable filters, lower CPU overhead, and GPU-accelerated delivery at native display resolution and frame rate.
- The filter/configuration separation makes it straightforward to update what is captured or how it is captured independently, without restarting the stream.
- Video samples are `IOSurface`-backed `CMSampleBuffer`s; always check `SCStreamFrameInfoStatus` before processing — `.idle` frames contain no new surface and should be skipped.
- All capture requires Screen Recording permission; no special entitlement is needed beyond user consent granted in System Preferences.

---
_Source: WWDC22 Session 10156 page (abstract, chapter summaries, code samples, and resource links)._
