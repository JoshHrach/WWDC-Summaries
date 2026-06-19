# Designing for Adverse Network and Temperature Conditions
**WWDC19 · Session 422** · [Watch](https://developer.apple.com/videos/play/wwdc2019/422/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15

## Overview
This session addresses a common gap between development environments and real-world usage: apps tested only under fast, climate-controlled conditions may perform poorly for users on 3G networks or in warm outdoor environments. Two new Xcode 11 Device Conditions features — Network Link and Thermal State simulation — let developers reproduce these adverse conditions reliably without custom network hardware or physically warming a device. The session covers the testing Pyramid model (unit → integration → UI tests), how clean-room testing creates blind spots, and provides concrete guidance on both network and thermal state best practices.

## Key Topics

- **Real-world gap** — Developers work in offices with fast Wi-Fi and climate control. Users may be on 3G, in direct sunlight, or charging wirelessly while using a hotspot. This gap produces App Store reviews about slow performance or poor behavior that are easily dismissed as edge cases.
- **Testing Pyramid model** — Unit tests optimize for speed in clean-room conditions; integration tests target subsystem interactions with some real-world variance; UI tests exercise the full app with real network and system state. Apps that focus exclusively on unit tests miss behavioral regressions surfaced only under real conditions.
- **Network Link Device Conditions (new in Xcode 11)** — In the Devices and Simulators window, a new Device Conditions panel lets you select a network type and quality (2G/EDGE, 3G, LTE, various Wi-Fi grades) and activate it system-wide on the device. The condition applies a ceiling on network performance without changing the UI status indicators. Deactivated automatically when device disconnects from Xcode. **[NEW]**
- **Network best practices** — Timeout on lack of progress (not arbitrary elapsed time), use HTTP/2, enable Optimistic DNS and TLS 1.3 where possible, avoid reachability pre-checks (just try the network). Demo showed ~33% connection time improvement on 3G by enabling Optimistic DNS + TLS 1.3.
- **Thermal State Conditions (new in Xcode 11)** — Device Conditions can set the device to report a specific thermal state (Fair, Serious, Critical) without physically warming it. This acts as a floor: if the device actually warms further under load, the state can still rise above the set floor. Ramping takes several seconds; `ProcessInfo.ThermalState` notifications fire normally. **[NEW]**
- **Four thermal states:**
  - **Nominal** — Normal operating temp; no action needed.
  - **Fair** — Start proactive energy-saving (system pauses discretionary background work like Photos analysis).
  - **Serious** — System reduces ARKit/FaceTime frame rates and pauses iCloud restore; app must reduce heavy CPU/GPU/I/O, use lower-quality visual effects.
  - **Critical** — App must stop using camera and peripherals; system in extreme conservation mode.
- **Xcode Energy Gauge** — Shows average energy impact and two new Thermal State tracks: one showing active device condition, one showing actual reported thermal state, both color-coded. **[NEW tracks]**
- **SceneKit/ARKit demo** — Fox 2 SceneKit sample modified to disable HDR, switch to blob shadows, reduce particle density, and lower post-processing quality dynamically at Serious/Critical states; frame rate recovered from ~17 FPS to ~20 FPS under Serious condition.

## APIs & Frameworks

### ProcessInfo
- `ProcessInfo.thermalState` — current `ProcessInfo.ThermalState` enum value: `.nominal`, `.fair`, `.serious`, `.critical`
- `ProcessInfo.thermalStateDidChangeNotification` — `NotificationCenter` notification posted whenever thermal state transitions

### Xcode Tools **[NEW]**
- **Network Link Device Conditions** — Devices and Simulators window panel; network types: 2G, EDGE (good/average), 3G (good/average), LTE, Wi-Fi (good/average); system-wide ceiling on network performance; gray status bar indicator on device **[NEW]**
- **Thermal State Device Conditions** — Fair, Serious, Critical simulated states; acts as floor not ceiling; ramps up/down over ~10 seconds **[NEW]**
- **Energy Gauge Thermal Tracks** — Two tracks in Xcode debugging navigator: active condition track and actual thermal state track, color-coded **[NEW]**
- **Network Link Conditioner (macOS Preference Pane)** — Pre-existing tool for macOS apps; custom presets for bandwidth, packet loss, latency
- **Network Link Conditioner (iOS Developer Settings)** — Pre-existing on-device settings panel for network type simulation

## Code Highlights

Register for thermal state notifications and respond:

```swift
NotificationCenter.default.addObserver(
    self,
    selector: #selector(thermalStateDidChange),
    name: ProcessInfo.thermalStateDidChangeNotification,
    object: nil)

@objc func thermalStateDidChange(_ notification: Notification) {
    let state = ProcessInfo.processInfo.thermalState
    switch state {
    case .nominal, .fair:
        // All features on: face tracking, person segmentation, motion blur
        enableAllFeatures()
    case .serious:
        // Reduce load: disable face tracking and frame semantics
        disableFaceTracking()
        disablePersonSegmentation()
    case .critical:
        // Stop everything including camera
        disableAllHeavyFeatures()
        stopCamera()
    @unknown default:
        break
    }
}
```

SceneKit quality reduction pattern:

```swift
switch ProcessInfo.processInfo.thermalState {
case .nominal, .fair:
    scene.enableHDR = true
    scene.shadowMode = .soft
    scene.postProcessing = .high
case .serious:
    scene.enableHDR = false
    scene.shadowMode = .blob
    scene.postProcessing = .medium
case .critical:
    scene.enableHDR = false
    scene.enableDepthOfField = false
    scene.shadowMode = .none
    scene.postProcessing = .off
@unknown default: break
}
```

## Takeaways

- Testing only in clean-room environments produces apps that work great in labs and fail for real users; make network link and thermal state simulation a mandatory part of integration and UI test runs, not an afterthought.
- Thermal state response should be coded defensively from the start — register for `ProcessInfo.thermalStateDidChangeNotification` and gracefully degrade features at Serious and Critical rather than letting the system throttle your frame rate for you.
- Set timeouts based on lack of progress rather than elapsed time: a user on a 3G network may be willing to wait; an arbitrary 10-second timeout will frustrate them when you could have succeeded with slightly more patience.
- The new Device Conditions tools eliminate the two most common workarounds (running dummy CPU loads, discarding the first hour of results) and make thermal state testing as repeatable as unit tests.

---
_Source: WWDC19 Session 422 page (abstract, full transcript, and resource links)._
