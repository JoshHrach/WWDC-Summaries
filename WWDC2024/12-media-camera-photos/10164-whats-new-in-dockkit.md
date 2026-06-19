# What's New in DockKit
**WWDC24 · Session 10164** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10164/)

_Platforms:_ iOS 18, iPadOS 18

## Overview
DockKit gains three major additions in iOS 18: **Intelligent Subject Tracking** — an ML pipeline that automatically selects and frames the most relevant person in a scene; **accessory button controls** — hardware events from buttons on a new class of DockKit accessories (gimbals) that apps can handle to trigger custom actions; and **expanded camera mode support** — tracking now works in Photo, Panorama, and Cinematic modes in the system Camera app. Battery state monitoring for accessories is also introduced.

The first DockKit-powered stands are now available for purchase at Apple Stores. Any app that uses the camera now automatically tracks subjects when a DockKit device is connected, without any code changes.

## Key Topics

### Intelligent Subject Tracking
Built on the multi-person tracker introduced in iOS 17 (which estimated trajectories of multiple subjects), iOS 18 adds a full **Intelligent Tracking Pipeline** with three stages:
1. **Subject Selection ML Model** — analyzes body pose, face pose, attention, and speaking confidence to determine the most relevant subject in the scene
2. **Subject Framing Module** — determines the most visually appealing composition for the selected subjects
3. **Motor Control** — uses positional and velocity feedback to issue actuator commands to the accessory

**Watch Control** was also introduced for all DockKit devices, letting users manually select or override the tracked subject from Apple Watch — tapping a face on Watch selects that person; swiping enables manual dock control.

### ML Signals Exposed to Apps
Apps using DockKit intelligent tracking can query the tracking summary via `trackingStates` (an async sequence). Each `TrackingState` includes a timestamp and a list of `TrackedSubject` values. Each `TrackedSubject` exposes:
- `identifier` — unique ID for the subject
- `faceRect` — bounding rect of the face
- `salientRank` — importance rank (1 = most salient, higher = less important)
- `speakingConfidence` (persons only) — 0.0 (not speaking) to 1.0 (speaking)
- `lookingAtCameraConfidence` (persons only) — 0.0 to 1.0

Apps can call `selectSubjects(_:)` with a filtered subset (e.g., only active speakers) to direct tracking.

### Button Controls for DockKit Accessories
Apps receive accessory button events via an async sequence `accessoryEvents`. Three built-in event types: **shutter** (capture photo/video), **flip** (switch cameras), and **zoom** (relative zoom factor — 2.0 doubles size, halves FOV). A **custom button** event type includes a button ID and a boolean pressed state for app-defined behaviors.

This enables a new category of accessories: **gimbals** — hand-held stabilizers for action/sports photography. Apps can use button events to start/stop dock rotation, begin panorama sweeps, or any other custom behavior.

### New Camera Modes
DockKit tracking is now integrated into the iOS Camera app's **Photo mode** (track subject while capturing stills), **Panorama mode** (one button press to autonomously sweep for a panorama), and **Cinematic mode** (cinematically track the person in focus).

### Battery State Monitoring
Apps can subscribe to `batteryStates` (async sequence) to monitor connected accessory battery. Each battery state reports a name (identifier), current percentage, and charging state (e.g., charging, discharging).

## APIs & Frameworks

**DockKit**
- `DockAccessory.trackingStates` — async sequence of `TrackingState` **[NEW]**
- `TrackingState` **[NEW]**
  - `.time` — capture timestamp
  - `.trackedSubjects: [TrackedSubject]` — list of subjects
- `TrackedSubject` **[NEW]**
  - `.identifier`
  - `.faceRect: CGRect`
  - `.salientRank: Int` — 1 = most salient
  - `.speakingConfidence: Double` (persons) **[NEW]**
  - `.lookingAtCameraConfidence: Double` (persons) **[NEW]**
- `DockAccessory.selectSubjects(_:)` — direct tracking to a specific subset **[NEW]**
- `DockAccessory.accessoryEvents` — async sequence of `AccessoryEvent` **[NEW]**
- `AccessoryEvent` **[NEW]**
  - `.shutter` — toggle event (no value)
  - `.flip` — toggle event (no value)
  - `.zoom(factor: Double)` — relative zoom factor
  - `.customButton(id: Int, isPressed: Bool)` — app-defined button
- `DockAccessory.batteryStates` — async sequence of battery status **[NEW]**
- Battery state — `.name`, `.percentage`, `.chargingState` **[NEW]**
- Photo mode tracking support **[NEW]**
- Panorama mode tracking support **[NEW]**
- Cinematic mode tracking support **[NEW]**
- Watch Control integration (no code required; system feature) **[NEW]**

## Code Highlights

Query tracked subjects and select only active speakers:
```swift
// Subscribe to tracking state
for await trackingState in dock.trackingStates {
    let activeSpeakers = trackingState.trackedSubjects
        .filter { $0.speakingConfidence > 0.8 }
    await dock.selectSubjects(activeSpeakers)
}
```

Handle custom gimbal button to start/stop panorama rotation:
```swift
for await event in dock.accessoryEvents {
    switch event {
    case .customButton(let id, let isPressed) where id == 5:
        if isPressed {
            startPanoramaRotation()
        } else {
            stopPanoramaRotation()
        }
    default: break
    }
}
```

Monitor accessory battery:
```swift
for await batteryState in dock.batteryStates {
    print("\(batteryState.name): \(batteryState.percentage)% - \(batteryState.chargingState)")
}
```

## Takeaways
- Intelligent Subject Tracking is a system-level ML pipeline; apps that simply use the camera benefit automatically — add `selectSubjects(_:)` calls only when you need custom logic (e.g., always track active speakers).
- The `salientRank`, `speakingConfidence`, and `lookingAtCameraConfidence` signals give apps the same ML inputs the system uses, enabling truly custom tracking heuristics.
- Handle `accessoryEvents` to build hands-free gimbal controls — button events are the key interaction model for the new class of hand-held DockKit gimbals.
- Monitor `batteryStates` to show proactive status messages so users aren't caught off guard by a dead accessory mid-session.

---
_Source: WWDC24 Session 10164 page (abstract, chapter summaries, and resource links)._
