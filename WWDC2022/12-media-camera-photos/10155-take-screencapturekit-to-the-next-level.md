# Take ScreenCaptureKit to the Next Level
**WWDC22 · Session 10155** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10155/)

_Platforms:_ macOS Ventura 13

## Overview
This advanced companion to "Meet ScreenCaptureKit" (10156) dives into fine-grained content filters, per-frame metadata interpretation, stream configuration for different use cases, and a practical window-picker pattern with live previews. The session demonstrates how to capture a single desktop-independent window, create inclusion/exclusion display-based filters for windows and apps, and live-adjust stream properties without recreating the stream.

A real-world highlight is the integration of ScreenCaptureKit into OBS Studio, which saw up to 15% RAM reduction and up to 50% CPU savings versus `CGWindowListCreateImage`, while achieving a smooth 60 fps output — demonstrating the framework's hardware-accelerated capture and encoding pipeline.

## Key Topics

### Single Window Filter
`SCContentFilter(desktopIndependentWindow:)` captures one specific window regardless of which display it's on. The output is always positioned at the top-left corner; pop-up and child windows are excluded. Audio capture always operates at the app level (captures all audio from the owning app). When the window is minimized, the stream pauses and resumes automatically.

### Display-Based Inclusion Filters
`SCContentFilter(display:including:)` for specific windows, or `SCContentFilter(display:including:exceptingWindows:)` for entire apps. New windows/child windows from included apps are automatically added to the stream output.

### Display-Based Exclusion Filters
`SCContentFilter(display:excludingApplications:exceptingWindows:)` captures everything on a display and removes specified apps — ideal for avoiding the "mirror hall effect" in conferencing apps. Excluded apps' audio is also removed.

### Per-Frame Metadata from CMSampleBuffer
Each output frame delivers metadata via `CMSampleBufferGetSampleAttachmentsArray` with `SCStreamFrameInfo` keys:
- `dirtyRects` — regions updated since the previous frame; enables partial encoding
- `contentRect` — region of captured window content within the output frame
- `contentScale` — how much the content was scaled to fit the output
- `scaleFactor` — display backing-pixel scale factor; used to correctly size content on target display

### Stream Configuration Properties
`SCStreamConfiguration` supports: `width`/`height`, `minimumFrameInterval`, `pixelFormat` (BGRA, YUV 420v/f), `sourceRect`, `destinationRect`, `backgroundColor`, `showsCursor`, `capturesAudio`, `queueDepth` (3–8; default 3).

### Live Stream Configuration Updates
`stream.updateConfiguration(_:)` updates resolution, frame rate, or content filters on the fly without restarting the stream, enabling adaptive quality based on network conditions.

### Surface Queue Depth and Performance
Queue depth (3–8 surfaces) affects frame latency and loss. Rule of thumb: release surfaces within `MinimumFrameInterval × (queueDepth - 1)` seconds to avoid frame loss.

### Window Picker with Live Previews
Creating one per-window stream with `SCContentFilter(desktopIndependentWindow:)` at thumbnail size (e.g., 284×182) and low frame rate (5 fps) produces an efficient live-preview picker, all backed by hardware-accelerated capture.

## APIs & Frameworks

**ScreenCaptureKit** (all **[NEW]** in macOS 12.3, deepened in macOS 13)

_Content_
- `SCShareableContent` — enumerates all capturable content
- `SCShareableContent.excludingDesktopWindows(_:onScreenWindowsOnly:)` **[NEW]** — async class method
- `SCWindow` — represents a capturable window
- `SCDisplay` — represents a capturable display
- `SCRunningApplication` — represents a running application

_Filters_
- `SCContentFilter(desktopIndependentWindow:)` **[NEW]** — single window, display-independent
- `SCContentFilter(display:including:)` **[NEW]** — display + specific windows
- `SCContentFilter(display:including:exceptingWindows:)` **[NEW]** — display + apps, with per-window exceptions
- `SCContentFilter(display:excludingApplications:exceptingWindows:)` **[NEW]** — exclusion-based filter

_Configuration_
- `SCStreamConfiguration` **[NEW]** — mutable configuration object
  - `.width`, `.height` — output pixel dimensions
  - `.minimumFrameInterval: CMTime` — controls maximum frame rate
  - `.pixelFormat` — `kCVPixelFormatType_32BGRA`, `kCVPixelFormatType_420YpCbCr8BiPlanarVideoRange`, etc.
  - `.sourceRect: CGRect` — sub-region of source display to capture
  - `.destinationRect: CGRect` — destination region within output frame
  - `.backgroundColor: CGColor`
  - `.showsCursor: Bool`
  - `.capturesAudio: Bool`
  - `.queueDepth: Int` — number of surfaces in pool (3–8)

_Stream_
- `SCStream(filter:configuration:delegate:)` **[NEW]** — creates a stream
- `SCStream.addStreamOutput(_:type:sampleHandlerQueue:)` **[NEW]** — registers output handler; `type` is `.screen` or `.audio`
- `SCStream.startCapture()` **[NEW]** — async
- `SCStream.stopCapture()` **[NEW]** — async
- `SCStream.updateConfiguration(_:)` **[NEW]** — live-updates stream config without restarting
- `SCStream.updateContentFilter(_:)` **[NEW]** — live-updates content filter

_Metadata keys (SCStreamFrameInfo)_
- `SCStreamFrameInfo.dirtyRects` **[NEW]**
- `SCStreamFrameInfo.contentRect` **[NEW]**
- `SCStreamFrameInfo.contentScale` **[NEW]**
- `SCStreamFrameInfo.scaleFactor` **[NEW]**
- `SCStreamFrameInfo.status` **[NEW]** — frame status (complete, idle, blank, suspended)

## Code Highlights

Single window filter with audio:
```swift
let shareableContent = try await SCShareableContent.excludingDesktopWindows(false, onScreenWindowsOnly: false)
guard let window = shareableContent.windows.first(where: { $0.windowID == windowID }) else { return }
let contentFilter = SCContentFilter(desktopIndependentWindow: window)
let streamConfig = SCStreamConfiguration()
streamConfig.capturesAudio = true
let stream = SCStream(filter: contentFilter, configuration: streamConfig, delegate: self)
try stream.addStreamOutput(self, type: .screen, sampleHandlerQueue: serialQueue)
try await stream.startCapture()
```

Live-downgrade from 4K 60fps to 720p 15fps:
```swift
streamConfiguration.width = 1280
streamConfiguration.height = 720
streamConfiguration.minimumFrameInterval = CMTime(value: 1, timescale: 15)
try await stream.updateConfiguration(streamConfiguration)
```

Reading per-frame metadata:
```swift
guard let attachmentsArray = CMSampleBufferGetSampleAttachmentsArray(sampleBuffer, createIfNecessary: false)
    as? [[SCStreamFrameInfo: Any]],
    let attachments = attachmentsArray.first else { return }
let contentRect = attachments[.contentRect]
let contentScale = attachments[.contentScale]
let scaleFactor = attachments[.scaleFactor]
```

## Takeaways
- Use `SCContentFilter(desktopIndependentWindow:)` for single-window capture; the window is tracked across displays and correctly handles occlusion, minimize, and off-screen states automatically.
- `dirtyRects` metadata enables partial-frame encoding for significant bandwidth savings in video conferencing and streaming.
- `stream.updateConfiguration(_:)` enables adaptive quality (resolution + frame rate) without restarting the stream — essential for network-adaptive streaming apps.
- A live-preview window picker is straightforward to build: one thumbnail-sized, low-fps single-window stream per eligible window, all hardware-accelerated.

---
_Source: WWDC22 Session 10155 page (abstract, chapter summaries, code samples, and resource links)._
