# Enhancing Your Camera Experience with Capture Controls
**WWDC25 · Session 253** · [Watch](https://developer.apple.com/videos/play/wwdc2025/253/)

_Platforms:_ iOS 26, iPadOS 26

## Overview
This session covers two areas of capture control: physical button capture (volume up/down, Action button, Camera Control) via `AVCaptureEventInteraction`, and Camera Control hardware sliders and pickers on iPhone 16 via `AVCaptureControl`. The headline new feature for iOS 26 is AirPods remote camera capture — pressing either H2-chip AirPod stem now triggers the primary capture event, complete with customizable audio feedback.

The session walks through adding the `onCameraCaptureEvent` SwiftUI modifier and creating system-provided or custom `AVCaptureControl` instances for Camera Control on iPhone 16.

## Key Topics

### Physical Capture (AVCaptureEventInteraction)
- `AVCaptureEventInteraction` — UIKit API; `onCameraCaptureEvent` — SwiftUI equivalent.
- Maps physical button presses to camera actions.
- **Event phases**: `began` (prepare camera), `cancelled` (app went to background or capture unavailable), `ended` (initiate capture).
- **Primary vs. secondary actions**: volume down / Action button / Camera Control → primary handler; volume up → secondary handler. Secondary handler is optional; if absent, volume up also triggers primary.
- System only sends events to apps with an active `AVCaptureSession`; backgrounded apps receive default system behavior.
- Apps **must always handle received events** — failing to do so leaves users with a non-functional button.

### AirPods Remote Capture (New in iOS 26)
- **[NEW]** AirPods with H2 chip support remote camera capture via stem clicks. Configurable in Settings > Remote Camera Capture.
- Works automatically for any app already using `AVCaptureEventInteraction` — zero extra code required for basic support.
- **Audio feedback API** — **[NEW]** `event.shouldPlaySound` is `true` only when the capture event was triggered by an AirPod stem click.
- `event.play(.cameraShutter)` — play a sound from the app's bundle on the AirPod.
- `defaultSoundDisabled: true` parameter on `onCameraCaptureEvent` — suppress the built-in default tone when providing a custom sound.

### Camera Control (AVCaptureControl, iPhone 16)
- Camera Control supports: launch app (Lock Screen capture extension), shutter button (free via `AVCaptureEventInteraction`), and hardware sliders/pickers for adjusting settings.
- **Control types**:
  - `AVCaptureSystemZoomSlider` — system-provided continuous zoom; drives `videoZoomFactor` on device automatically.
  - `AVCaptureSystemExposureBiasSlider` — system-provided discrete exposure bias slider.
  - `AVCaptureContinuousSlider` — custom continuous slider for any numeric range.
  - `AVCaptureIndexPicker` — custom picker for named states (e.g., filters, reaction effects).
- Add controls to `AVCaptureSession` after `beginConfiguration`; check `supportsControls` and `canAddControl`.
- `session.maxControlsCount` — limits total controls; `canAddControl` returns false when exceeded.
- Controls with custom behaviors accept a `setActionQueue(_:)` + handler closure; system controls accept optional KVO or a closure for UI sync.
- **Disable, don't remove** — set `control.isEnabled = false` when a control is contextually unavailable; removing causes another control to be selected silently.

## APIs & Frameworks

### AVKit
- `AVCaptureEventInteraction` — UIKit interaction for physical button capture.
- `onCameraCaptureEvent(defaultSoundDisabled:)` — **[NEW `defaultSoundDisabled` param]** SwiftUI view modifier.
- `AVCaptureEvent.phase` (`.began`, `.cancelled`, `.ended`)
- `AVCaptureEvent.shouldPlaySound` — **[NEW]** true only for AirPod stem triggers.
- `AVCaptureEvent.play(_:)` — **[NEW]** play audio on AirPods.

### AVFoundation
- `AVCaptureSession` — `supportsControls`, `maxControlsCount`, `canAddControl(_:)`, `addControl(_:)`
- `AVCaptureControl` — abstract base.
- `AVCaptureSystemZoomSlider(device:)` / `AVCaptureSystemZoomSlider(device:action:)` **[NEW optional action param]**
- `AVCaptureSystemExposureBiasSlider(device:)`
- `AVCaptureContinuousSlider` — custom continuous slider.
- `AVCaptureIndexPicker(_:symbolName:localizedIndexTitles:)` — custom picker with SF Symbol.
- `AVCaptureIndexPicker.isEnabled` — disable without removing.
- `AVCaptureIndexPicker.setActionQueue(_:action:)` — custom action handler.

### LockedCameraCapture (referenced)
- Lock Screen capture extension — required to launch app via Camera Control from Lock Screen.

## Code Highlights

```swift
// SwiftUI: capture on physical button press + AirPods with custom sound
import AVKit
CameraView()
    .onCameraCaptureEvent(defaultSoundDisabled: true) { event in
        if event.phase == .ended {
            if event.shouldPlaySound {
                event.play(.cameraShutter)
            }
            camera.capturePhoto()
        }
    }
```

```swift
// Add system zoom control to Camera Control
captureSession.beginConfiguration()
if captureSession.supportsControls {
    let zoomControl = AVCaptureSystemZoomSlider(device: device) { [weak self] zoomFactor in
        self?.updateUI(zoomFactor: zoomFactor)
    }
    if captureSession.canAddControl(zoomControl) {
        captureSession.addControl(zoomControl)
    }
}
captureSession.commitConfiguration()
```

```swift
// Custom reaction effects picker
let reactions = device.availableReactionTypes.sorted { $0.rawValue < $1.rawValue }
let picker = AVCaptureIndexPicker("Reactions", symbolName: "face.smiling.inverted",
    localizedIndexTitles: reactions.map { localizedTitle(reaction: $0) })
picker.isEnabled = device.canPerformReactionEffects
picker.setActionQueue(sessionQueue) { index in
    device.performEffect(for: reactions[index])
}
```

## Takeaways
- Apps that already use `AVCaptureEventInteraction` / `onCameraCaptureEvent` automatically receive AirPods remote capture on iOS 26 with no additional code changes.
- Add custom audio feedback using `event.shouldPlaySound` and `event.play(_:)` when a shutter sound appropriate to your app's tone differs from the default.
- Always disable unavailable Camera Control items with `isEnabled = false` rather than removing them — removing causes silent fallback to another control, confusing users.
- Sync your in-app zoom UI with Camera Control zoom by using the action closure on `AVCaptureSystemZoomSlider`; otherwise pinch-to-zoom will jump to a stale zoom factor after Camera Control is used.

---
_Source: WWDC25 Session 253 page (abstract, chapters, full transcript, and code samples)._
