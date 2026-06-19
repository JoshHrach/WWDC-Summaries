# What's new in ScreenCaptureKit
**WWDC23 · Session 10136** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10136/)

_Platforms:_ macOS Sonoma 14

## Overview
ScreenCaptureKit, introduced in macOS 12.3, gains three major new capabilities in macOS Sonoma: Presenter Overlay, a system-wide screen sharing picker, and a screenshot API. Together these features allow apps to deeply integrate with macOS screen-sharing infrastructure rather than building everything from scratch.

Presenter Overlay is a new video effect that composites the presenter's camera feed on top of shared content — either as a small movable window or as a large immersive overlay where screen content appears layered between the presenter's face and body. Any app using both SCStream and AVCaptureSession automatically gains access to this effect via the system Video menu bar item.

The new SCContentSharingPicker provides a singleton interface to a system-level content picker, letting users choose what to share directly from windows, apps, or displays without requiring each app to build its own picker UI. A brand-new SCScreenshotManager class rounds out the release with async, one-off screen captures using the same filter and configuration model as streaming.

## Key Topics

**Presenter Overlay**
- New macOS video effect that embeds the presenter's camera into an active SCStream
- Two modes: small (movable window, segmentation algorithm) and large (background separation, screen content layered between presenter)
- Notified via new `SCStreamDelegate` callback `outputEffectDidStart`
- When active, AVCaptureSession pauses its live camera feed; apps should update UI, adjust A/V sync, and hide the camera tile

**Screen Sharing Picker (SCContentSharingPicker)**
- Singleton picker that replaces manual SCSharableContent enumeration for filter creation
- Integrated into the macOS Video menu bar item alongside Presenter Overlay controls
- Supports selection by window, app, or display; delivers SCContentFilter via observer callbacks
- Per-stream SCContentSharingPickerConfiguration controls allowed picking modes, excluded window/bundle IDs, and whether repicking is allowed

**Screenshot API (SCScreenshotManager)**
- New class with two async class methods: `captureSampleBuffer` and `captureImage`
- Takes the same SCContentFilter and SCStreamConfiguration used for streaming
- Supports all SCStreamConfiguration options: pixel formats, color spaces, cursor visibility, etc.
- Drop-in upgrade path from `CGWindowListCreateImage`

## APIs & Frameworks

**ScreenCaptureKit**
- `SCStream` — existing streaming class **[updated]**
- `SCStreamDelegate` — protocol updated with new callback:
  - `stream(_:outputEffectDidStart:)` **[NEW]** — notified when Presenter Overlay starts/stops
- `SCContentSharingPicker` **[NEW]** — singleton system picker
  - `SCContentSharingPicker.shared()` — access the singleton
  - `picker.addObserver(_:)` — register observer
  - `picker.active` — activate picker so system recognizes it
  - `picker.present(for:using:)` — present system picker with a content style
  - `picker.setConfiguration(_:for:)` — apply per-stream configuration
- `SCContentSharingPickerConfiguration` **[NEW]**
  - `allowedPickingModes` — restrict to windows/apps/displays
  - `excludedWindowIDs` — exclude specific windows
  - `excludedBundleIDs` — exclude specific app bundles
  - `allowsRepicking` — whether user can change selection
- `SCContentSharingPickerObserver` **[NEW]** protocol with callbacks:
  - `contentSharingPicker(_:didUpdateWith:for:)` — new or updated SCContentFilter
  - `contentSharingPicker(contentSharingPickerStartDidFailWith:)` — picker failed
  - `contentSharingPicker(_:didCancel:for:)` — picker cancelled
- `SCScreenshotManager` **[NEW]**
  - `captureSampleBuffer(contentFilter:configuration:)` async throws → `CMSampleBuffer`
  - `captureImage(contentFilter:configuration:)` async throws → `CGImage`
- `SCContentFilter` — updated to be creatable via SCContentSharingPicker
- `SCStreamConfiguration` — provides all screenshot configuration options (pixel format, color space, cursor)
- `SCSharableContent` — existing enumeration API (still available)

## Code Highlights

Detecting Presenter Overlay start/stop:
```swift
let stream = SCStream(filter: filter, configuration: config, delegate: self)

func stream(_ stream: SCStream, outputEffectDidStart didStart: Bool) {
    if didStart {
        presentBanner()
        turnOffCamera()
    } else {
        turnOnCamera()
    }
}
```

Setting up SCContentSharingPicker:
```swift
let picker = SCContentSharingPicker.shared()
picker.addObserver(self)
picker.active = true

func showSystemPicker(sender: UIButton!) {
    picker.present(for: nil, using: .window)
}

func contentSharingPicker(_ picker: SCContentSharingPicker, didUpdateWith filter: SCContentFilter, for stream: SCStream?) {
    if let stream = stream {
        stream.updateContentFilter(filter)
    } else {
        let stream = SCStream(filter: filter, configuration: config, delegate: self)
    }
}
```

Taking a screenshot:
```swift
let myContentFilter = SCContentFilter(display: display, excludingApplications: [], exceptingWindows: [])
let myConfiguration = SCStreamConfiguration()

if let screenshot = try? await SCScreenshotManager.captureSampleBuffer(
    contentFilter: myContentFilter, configuration: myConfiguration) {
    print("Fetched screenshot.")
}
```

## Takeaways
- Any app using SCStream automatically participates in Presenter Overlay and the system Video menu bar — no extra code needed.
- Replace custom content picker UIs with SCContentSharingPicker for a consistent, system-integrated experience.
- Use SCScreenshotManager to replace CGWindowListCreateImage with a more capable, privacy-aware screenshot API.
- Configure per-stream SCContentSharingPickerConfiguration to lock down or open up the system picker behavior on a stream-by-stream basis.

---
_Source: WWDC23 Session 10136 page (abstract, chapter summaries, code samples, and resource links)._
