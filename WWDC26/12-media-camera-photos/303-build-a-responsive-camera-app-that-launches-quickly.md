# Build a Responsive Camera App that Launches Quickly
**WWDC26 · Session 303** · [Watch](https://developer.apple.com/videos/play/wwdc2026/303/)

_Platforms:_ iOS, iPadOS

## Overview
The most critical metric for a camera app is the time from app launch to the first live preview frame on screen. This session walks through every stage of that pipeline — app startup overhead, `AVCaptureSession` configuration, preview rendering — and introduces new WWDC26 APIs designed to collapse that latency significantly.

The central new API is deferred start: the ability to split `AVCaptureSession.startRunning()` into two phases so the preview layer is live before expensive outputs like `AVCapturePhotoOutput` finish initializing. Both automatic and manual deferred-start modes are covered, and the session shows how to trigger the deferred phase precisely when the first Metal drawable is presented.

Beyond launch, the session addresses sustained performance: monitoring `hardwareCost`, observing `systemPressureState`, and the brand-new `AVProVideoStorage` API for deterministic file write speeds needed by ProRes video capture.

## Key Topics

### Fast Launch
Minimize work on the main thread during app startup. Create and configure `AVCaptureSession` as early as possible, preferring a background queue. Avoid loading unrelated UI before the preview layer is visible.

### Adopt Deferred Start
`AVCaptureSession.automaticallyRunsDeferredStart` controls whether the system handles the deferred-start timing automatically after the first preview frame, or whether the app calls `runDeferredStartWhenNeeded()` manually. Per-output `isDeferredStartEnabled` flags let you designate which outputs (e.g., `AVCapturePhotoOutput`) are deferred while the preview layer starts immediately.

The delegate `AVCaptureSessionDeferredStartDelegate` provides `sessionWillRunDeferredStart` and `sessionDidRunDeferredStart` callbacks for coordinating UI state.

### Steady Preview
`AVCaptureVideoPreviewLayer` is the simplest path and handles orientation, mirroring, and gravity automatically. `AVCaptureVideoDataOutput` offers more flexibility for custom rendering (e.g., Metal compositing) but requires careful frame-drop handling to avoid backpressure.

### Sustained Performance
`AVCaptureSession.hardwareCost` reports a 0–1 normalized load value. If it exceeds 1.0 after `commitConfiguration()`, the session cannot start and the app must reconfigure to a lower-cost layout. `AVCaptureDevice.systemPressureState` (KVO-observable) provides ongoing thermal/power pressure signals so the app can react at runtime.

### Deterministic File Writing
`AVProVideoStorage` allocates dedicated storage space for high-data-rate captures like ProRes. Call `AVProVideoStorage.isSupported` to gate the feature, check `remainingCapacity` before recording, and set `movieOutput.usesProVideoStorage = true` on `AVCaptureMovieFileOutput`. The same flag is available on `AVAssetWriter`.

## APIs & Frameworks

### AVFoundation
- `AVCaptureSession`
  - `automaticallyRunsDeferredStart` — **[NEW]** `true` for automatic, `false` for manual deferred start
  - `runDeferredStartWhenNeeded()` — **[NEW]** manually trigger the deferred phase
  - `setDeferredStartDelegate(_:deferredStartDelegateCallbackQueue:)` — **[NEW]** register deferred-start callbacks
  - `hardwareCost` — **[NEW]** 0–1 hardware utilization; must be ≤1.0 to start
  - `beginConfiguration()` / `commitConfiguration()` / `startRunning()`
- `AVCaptureSessionDeferredStartDelegate` — **[NEW]** protocol
  - `sessionWillRunDeferredStart(_:)`
  - `sessionDidRunDeferredStart(_:)`
- `AVCaptureVideoPreviewLayer`
  - `isDeferredStartEnabled` — **[NEW]** set to `false` to keep preview in the immediate phase
- `AVCapturePhotoOutput`
  - `isDeferredStartEnabled` — **[NEW]** set to `true` to defer initialization
  - `isResponsiveCaptureEnabled` / `isResponsiveCaptureSupported`
  - `maxPhotoQualityPrioritization`
- `AVCaptureVideoDataOutput`
  - `isDeferredStartEnabled` — **[NEW]**
- `AVCaptureDevice`
  - `systemPressureState` — KVO-observable thermal/power pressure; use `.observe(\.systemPressureState, options: [.initial, .new])`
- `AVCaptureMovieFileOutput`
  - `usesProVideoStorage` — **[NEW]**
  - `isProVideoStorageSupported` — **[NEW]**
  - `startRecording(to:recordingDelegate:)`
- `AVProVideoStorage` — **[NEW]** class
  - `isSupported` — static guard
  - `shared` — singleton accessor
  - `remainingCapacity` — available dedicated storage in bytes
  - `isBusy` — whether another recording is using pro storage
  - `openSettings()` — direct user to storage management UI
- `AVAssetWriter`
  - `usesProVideoStorage` — **[NEW]**

### QuartzCore / Metal
- `CAMetalLayer.nextDrawable()` / `drawable.addPresentedHandler(_:)` — used in manual deferred start to fire `runDeferredStartWhenNeeded()` exactly at first frame presentation

### Sample Code Reference
- [AVCam: Building a camera app](https://developer.apple.com/documentation/AVFoundation/avcam-building-a-camera-app)
- [Performance and metrics](https://developer.apple.com/documentation/Xcode/performance-and-metrics)

## Code Highlights

Automatic deferred start — preview starts immediately, photo output deferred:
```swift
captureSession.automaticallyRunsDeferredStart = true
videoPreviewLayer.isDeferredStartEnabled = false
photoOutput.isDeferredStartEnabled = true
captureSession.setDeferredStartDelegate(delegate, deferredStartDelegateCallbackQueue: sessionQueue)
```

Manual deferred start triggered on first Metal frame:
```swift
drawable.addPresentedHandler { _ in
    captureSession.runDeferredStartWhenNeeded()
}
```

Hardware cost guard before start:
```swift
guard captureSession.hardwareCost <= 1.0 else {
    setupLowCostConfiguration()
    return
}
captureSession.startRunning()
```

## Takeaways
- The new deferred-start API (automatic or manual) is the highest-leverage change for reducing camera launch time; preview starts before photo/video output pipelines are fully initialized.
- Triggering `runDeferredStartWhenNeeded()` inside a Metal drawable's presented handler ensures the deferred phase starts at the exact moment the user sees the first frame.
- Monitor `hardwareCost` after `commitConfiguration()` and `systemPressureState` at runtime to maintain a stable, sustainable capture session.
- `AVProVideoStorage` is required for reliable ProRes capture; check `isSupported` and `remainingCapacity` before every recording.

---
_Source: WWDC26 Session 303 page (abstract, chapter summaries, code samples, and resource links)._
