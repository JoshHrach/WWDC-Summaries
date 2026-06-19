# What's New in Core Motion
**WWDC23 · Session 10179** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10179/)

_Platforms:_ iOS 14+ / macOS 14 (headphone motion), watchOS 9+ (submersion), watchOS 10 (batched sensor)

## Overview
This session covers three new and expanded Core Motion capabilities in 2023. First, `CMHeadphoneMotionManager` — previously iOS/iPadOS-only — now streams attitude, user acceleration, and rotation rate from AirPods and compatible headphones to **macOS 14**. Second, `CMWaterSubmersionManager` (Apple Watch Ultra) gains a clearer API walkthrough covering depth zones, water temperature uncertainty, and submersion state events. Third, the entirely new `CMBatchedSensorManager` delivers **800 Hz accelerometer** and **200 Hz device motion** data in batches to Apple Watch Series 8 and Ultra during active HealthKit workout sessions — enabling high-fidelity analysis of short-duration, impact-based sports like golf, tennis, and baseball.

The session uses a baseball swing analysis as a concrete worked example: detecting bat-ball impact from the filtered 800 Hz accelerometer Z-axis signal, and identifying swing start from rotation-rate-along-gravity in the 200 Hz device motion stream, to compute time-to-contact.

## Key Topics

### CMHeadphoneMotionManager — Now on macOS 14
- `CMHeadphoneMotionManager` **[macOS NEW]** — streams `CMDeviceMotion` (attitude, user acceleration, rotation rate) from AirPods Pro and other spatial-audio-capable headphones to a connected iPhone, iPad, or now Mac.
- `CMHeadphoneMotionManagerDelegate` — receives `headphoneMotionManagerDidConnect(_:)` and `headphoneMotionManagerDidDisconnect(_:)` events; also fires when Automatic Ear Detection causes a connection change (bud removed/inserted, headphones put on/taken off).
- `CMDeviceMotion.sensorLocation` — identifies which bud (left or right) is currently sourcing the data; streaming switches buds seamlessly when Automatic Ear Detection changes in-ear state.
- Authorization: requires `NSMotionUsageDescription` in Info.plist; check `CMHeadphoneMotionManager.authorizationStatus` before streaming.
- Streaming: check `isDeviceMotionAvailable`, set delegate, call `startDeviceMotionUpdates(to:withHandler:)`.
- Relative attitude: use `CMAttitude.multiply(byInverseOf:)` to compute pose change relative to a stored reference attitude.

### CMWaterSubmersionManager — Apple Watch Ultra
- `CMWaterSubmersionManager` — available on Apple Watch Ultra (watchOS 9+); requires the **Shallow Depth and Pressure** capability in entitlements.
- Depth zones via `CMWaterSubmersionMeasurement.submersionState`:
  - `.notSubmerged` — out of water
  - `.submergedShallow` — 0–1 m depth
  - `.submergedDeep` — 1–6 m depth
  - `.approachingMaxDepth` — nearing the 6 m cap
  - `.pastMaxDepth` — beyond 6 m (data still vended with uncertainty)
  - `.sensorDepthError` — sensor limits exceeded
- `CMWaterSubmersionMeasurement` — delivers depth, pressure, surface pressure, and submersion state at regular intervals; depth is `nil` when not submerged.
- `CMWaterTemperature` — delivers water temperature with an uncertainty value; uncertainty is higher at first submersion and converges as the watch temperature equalizes with the water. Only available when submerged.
- `CMWaterSubmersionEvent` — fires on submersion state transitions (entering/exiting water).
- Configure Auto Launch settings so the app starts automatically when users begin water activities.
- `CMWaterSubmersionManager.maximumDepth` — the monitored depth ceiling (6 m with Shallow Depth and Pressure capability).

### CMBatchedSensorManager — High-Rate Workout Data (NEW)
- `CMBatchedSensorManager` **[NEW]** — delivers high-frequency sensor data in batches (one batch per second) during an active `HKWorkoutSession`.
- Supported rates: **800 Hz accelerometer**, **200 Hz device motion** (vs. 100 Hz max from `CMMotionManager`).
- Platforms: Apple Watch Series 8 and Apple Watch Ultra.
- Requires an active HealthKit workout session; authorization for motion data (`NSMotionUsageDescription`) and HealthKit workout permissions are both required.
- Data delivery via Swift async sequences: iterate over `accelerometerBatches` and `deviceMotionBatches` async sequences; each element is an array of samples for that one-second window.
- Use case guidance: well-suited for workout-centric features that benefit from high-rate data without tight sub-second latency requirements; not a replacement for `CMMotionManager` when real-time low-latency updates are needed.
- Key availability checks: `CMBatchedSensorManager.isAccelerometerSupported`, `CMBatchedSensorManager.isDeviceMotionSupported`.

### High-Rate Data Analysis Pattern (Baseball Swing Example)
1. **Impact detection** — filter the 800 Hz accelerometer Z-axis signal to isolate high-frequency impact reverberation; find the peak as the impact timestamp.
2. **Swing start detection** — scan the 200 Hz device motion rotation-rate-along-gravity buffer backwards from the impact timestamp; find where rotation-rate-along-gravity consistently falls below a threshold (bounds: swing duration guard, accumulated rotation validation).
3. **Time to contact** — `impactTime - swingStartTime`.

## APIs & Frameworks

- `CoreMotion` framework
- `CMHeadphoneMotionManager` — streams headphone device motion; now on macOS 14 **[macOS NEW]**
- `CMHeadphoneMotionManager.isDeviceMotionAvailable` — check before streaming
- `CMHeadphoneMotionManager.authorizationStatus` — motion authorization state
- `CMHeadphoneMotionManager.startDeviceMotionUpdates(to:withHandler:)` — push interface for streaming
- `CMHeadphoneMotionManagerDelegate` — `headphoneMotionManagerDidConnect(_:)`, `headphoneMotionManagerDidDisconnect(_:)`
- `CMDeviceMotion` — attitude, userAcceleration, rotationRate, sensorLocation
- `CMDeviceMotion.sensorLocation` — `.left`, `.right`, `.default`
- `CMAttitude.multiply(byInverseOf:)` — relative attitude computation
- `CMWaterSubmersionManager` — water depth/temperature/state tracking on Apple Watch Ultra
- `CMWaterSubmersionManagerDelegate` — `manager(_:didUpdate:)` for events and measurements
- `CMWaterSubmersionMeasurement` — depth, pressure, surfacePressure, submersionState
- `CMWaterSubmersionMeasurement.submersionState` — `.notSubmerged`, `.submergedShallow`, `.submergedDeep`, `.approachingMaxDepth`, `.pastMaxDepth`, `.sensorDepthError`
- `CMWaterTemperature` — temperature, uncertainty
- `CMWaterSubmersionEvent` — submersion state change event
- `CMWaterSubmersionManager.maximumDepth` — depth ceiling property
- `CMBatchedSensorManager` **[NEW]** — high-rate batched sensor streaming during HKWorkoutSession
- `CMBatchedSensorManager.isAccelerometerSupported` **[NEW]** — availability check for 800 Hz accelerometer
- `CMBatchedSensorManager.isDeviceMotionSupported` **[NEW]** — availability check for 200 Hz device motion
- `CMBatchedSensorManager.accelerometerBatches` **[NEW]** — AsyncSequence of `[CMAccelerometerData]` batches
- `CMBatchedSensorManager.deviceMotionBatches` **[NEW]** — AsyncSequence of `[CMDeviceMotion]` batches
- `CMAccelerometerData` — x/y/z acceleration samples with timestamps
- `HKWorkoutSession` — required active session for `CMBatchedSensorManager`
- `NSMotionUsageDescription` (Info.plist key) — required for motion authorization

## Code Highlights

```swift
// CMHeadphoneMotionManager setup (iOS, iPadOS, macOS 14)
let headphoneManager = CMHeadphoneMotionManager()
headphoneManager.delegate = self

guard headphoneManager.isDeviceMotionAvailable else { return }
headphoneManager.startDeviceMotionUpdates(to: .main) { motion, error in
    guard let motion else { return }
    // Relative attitude
    let current = motion.attitude
    current.multiply(byInverseOf: startingPose)
    // Identify source bud
    let location = motion.sensorLocation // .left or .right
}

// CMHeadphoneMotionManagerDelegate
func headphoneMotionManagerDidConnect(_ manager: CMHeadphoneMotionManager) { /* start streaming */ }
func headphoneMotionManagerDidDisconnect(_ manager: CMHeadphoneMotionManager) { /* pause streaming */ }
```

```swift
// CMBatchedSensorManager — high-rate workout streaming
let batchedManager = CMBatchedSensorManager()
guard CMBatchedSensorManager.isAccelerometerSupported,
      CMBatchedSensorManager.isDeviceMotionSupported else { return }

// In an active HKWorkoutSession:
Task {
    for await batch in batchedManager.accelerometerBatches {
        // batch: [CMAccelerometerData] at 800 Hz
        feed(accelerometerBatch: batch)
        if workoutEnded { break }
    }
}

Task {
    for await batch in batchedManager.deviceMotionBatches {
        // batch: [CMDeviceMotion] at 200 Hz
        buffer.append(contentsOf: batch)
    }
}
```

```swift
// CMWaterSubmersionManager setup
let submersionManager = CMWaterSubmersionManager()
submersionManager.delegate = self

// Delegate callbacks
func manager(_ manager: CMWaterSubmersionManager,
             didUpdate measurement: CMWaterSubmersionMeasurement) {
    let depth = measurement.depth        // nil if not submerged
    let state = measurement.submersionState
    let surfacePressure = measurement.surfacePressure
}

func manager(_ manager: CMWaterSubmersionManager,
             didUpdate event: CMWaterSubmersionEvent) {
    // Submersion state transition (entered/exited water)
}

func manager(_ manager: CMWaterSubmersionManager,
             didUpdate temperature: CMWaterTemperature) {
    let temp = temperature.temperature
    let uncertainty = temperature.uncertainty // higher on initial submersion
}
```

## Takeaways
- `CMHeadphoneMotionManager` now works on macOS 14, making it possible to stream head-tracking motion from AirPods to Mac apps — the same data that powers dynamic spatial audio is now accessible on desktop.
- `CMBatchedSensorManager` provides 8x the accelerometer frequency (800 Hz vs 100 Hz) and 2x the device motion frequency (200 Hz vs 100 Hz) of `CMMotionManager`, unlocking impact detection and fine-grained swing analysis that was previously impossible on Apple Watch.
- The batched delivery model (one batch/second) keeps CPU overhead low — ideal for workout analysis where data is processed asynchronously rather than frame-by-frame.
- `CMWaterSubmersionManager` depth zones make it straightforward to enforce safety boundaries (max 6 m) and display contextual UI during snorkeling or swimming activities on Apple Watch Ultra.

---
_Source: WWDC23 Session 10179 page (abstract, transcript, and resource links)._
