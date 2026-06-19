# What's New in Core Motion
**WWDC15 · Session 705** · [Watch](https://developer.apple.com/videos/play/wwdc2015/705/)

_Platforms:_ iOS 9, watchOS 2

## Overview
This session covers the Core Motion updates for iOS 9 and the first introduction of Core Motion APIs on Apple Watch (watchOS 2). Presenters Anil Kandangath and Gabrielle Badie walk through Watch-specific considerations (background limitations, graceful suspension), a brand-new CMSensorRecorder API for historical accelerometer data on Watch, pedometer improvements (GPS fusion, new pace and cadence APIs), altimeter behavior, and a demo music app that uses motion context to automatically switch playlists based on activity, pace, cadence, and altitude.

The session introduces the concept of using Core Motion not just in fitness/gaming apps but as a cross-category tool for context-aware intelligence: detecting what a user is doing, engaging them with real-time updates, and reflecting historical data back to them in meaningful ways.

## Key Topics

### Core Motion on Apple Watch **[NEW]**
- Apple Watch has a motion coprocessor + accelerometer; M-series coprocessor enables 24/7 background processing.
- Most Core Motion APIs available on both iOS and watchOS; behavior is consistent across both platforms.
- **Motion activity states on watchOS**: walking, running, cycling, stationary (automotive not available on Watch).
- **Accelerometer on Watch**: accessible via `CMAccelerometer` API; app only gets processing time while on screen. Screen may turn off due to timeout or wrist orientation.
- Best practice: design Watch apps to expect sensor data only while the app is on screen; handle task suspension gracefully using `NSProcessInfo.performExpiringActivity(withReason:using:)`.

### CMSensorRecorder — Historical Accelerometer Data on Watch **[NEW]**
- Records accelerometer data at 50 Hz; queryable for up to 3 days.
- Data available even when app is not running (unlike live CMAccelerometer streaming).
- Must explicitly initiate recording — unlike pedometer/activity history which is always collected.
- Steps: call `recordAccelerometer(forDuration:)` → app may be suspended → later call `accelerometerData(from:to:)` to retrieve the recorded window.
- Returns a sequence of `CMRecordedAccelerometerData` objects, each with an `startDate` property.
- Use `startDate` as a cursor: on next app launch, query from last processed `startDate` onward.
- Handle task suspension with `NSProcessInfo.performExpiringActivity` when processing large data blocks.
- Best practice: record and query only minimum required duration; drop samples if your algorithm doesn't need the full 50 Hz rate.

### Pedometer Updates in iOS 9
- **GPS fusion** **[NEW]**: When the app is already subscribing to location, `CMPedometer` automatically incorporates GPS data into distance estimation when GPS is reliable, and falls back to stride-based estimation in degraded GPS environments (e.g., urban canyons). Net result: more consistent and accurate distance.
- **`currentPace`** (new property) **[NEW]**: Instantaneous pace in seconds per meter (time-over-distance format matching runners' expectation). Available only in live queries (`startUpdates`), not historical. Smoothed by the pedometer to eliminate jitter while remaining responsive to pace changes.
- **`currentCadence`** (new property) **[NEW]**: Steps per second (landing rate). Available in live queries. Important for runners; replaces manual computation.
- Existing APIs: `floorsAscended`, `floorsDescended` — require a minimum ascent rate and step threshold (earned floors, not elevator/escalator floors).
- Pedometer is available on both iPhone and Apple Watch.

### Altimeter (CMAltimeter)
- Available since iPhone 6 (M8 coprocessor) via `CMAltimeter` API.
- Provides `pressure` (filtered raw sensor reading) and `relativeAltitude` (relative to first sample; first sample = 0.0 m).
- **Suitable for**: floor-level changes.
- **Not suitable for**: human-scale motion like raising an arm; ambient environment changes (cold fronts can cause ~15m false altitude shift); long-duration trends; sealed waterproof cases (block pressure sensor).
- First sample takes ~2.6 seconds; subsequent samples arrive at ~1.3 second intervals.
- `startRelativeAltitudeUpdates(to:withHandler:)` / `stopRelativeAltitudeUpdates()`.
- Check availability before querying: `CMAltimeter.isRelativeAltitudeAvailable()`.

### Context-Aware App Design (Demo: Music Motion App)
Three-part framework for using Core Motion beyond fitness/gaming:
1. **Detect**: Use `CMMotionActivityManager` live updates + `CMPedometer` live updates to infer user context (driving → podcasts, running → workout playlist, walking → relaxed music). Apply app-specific smoothing to avoid rapid playlist switching at activity transitions.
2. **Engage**: Use real-time pedometer `currentPace`, `currentCadence`, and altimeter `relativeAltitude` updates to respond to in-session changes (e.g., play motivational song when user climbs a big hill).
3. **Reflect**: Use `CMMotionActivityManager.queryActivityStarting(from:to:in:withHandler:)` and `CMPedometer.queryPedometerData(from:to:withHandler:)` for historical queries to build interesting summaries (weekly activity breakdown, paired with playlist history).

## APIs & Frameworks

- `CoreMotion` framework
- `CMMotionManager` — live accelerometer, gyro, device motion
- `CMAccelerometerData` — raw accelerometer readings
- `CMSensorRecorder` **[NEW]**
  - `recordAccelerometer(forDuration:)` **[NEW]**
  - `accelerometerData(from:to:)` → `CMSensorDataList` **[NEW]**
  - `CMRecordedAccelerometerData` — `startDate`, `acceleration` **[NEW]**
- `CMMotionActivityManager` — motion activity (walking, running, cycling, automotive, stationary)
  - `startActivityUpdates(to:withHandler:)` — live activity updates
  - `queryActivityStarting(from:to:in:withHandler:)` — historical queries (up to 7 days)
  - `CMMotionActivity` — `walking`, `running`, `cycling`, `automotive`, `stationary`, `confidence`
  - `CMMotionActivityManager.isActivityAvailable()` — availability check
- `CMPedometer` — step counting, distance, floors, pace, cadence
  - `startUpdates(from:withHandler:)` — live pedometer updates
  - `queryPedometerData(from:to:withHandler:)` — historical pedometer queries
  - `CMPedometerData`:
    - `numberOfSteps`, `distance`, `floorsAscended`, `floorsDescended` (existing)
    - `currentPace` **[NEW]** — `NSMeasurement` in s/m, live only
    - `currentCadence` **[NEW]** — steps/second, live only
  - `CMPedometer.isStepCountingAvailable()`, `isDistanceAvailable()`, `isFloorCountingAvailable()`, `isPaceAvailable()` **[NEW]**, `isCadenceAvailable()` **[NEW]**
- `CMAltimeter` — pressure and relative altitude
  - `isRelativeAltitudeAvailable()` — availability check
  - `startRelativeAltitudeUpdates(to:withHandler:)` — live altitude updates
  - `stopRelativeAltitudeUpdates()` — stop live updates
  - `CMAltitudeData` — `relativeAltitude` (meters), `pressure` (kPa)
- `NSProcessInfo.performExpiringActivity(withReason:using:)` — graceful task suspension handling on Watch and during background sensor processing

## Code Highlights

Start CMSensorRecorder recording (Watch):
```swift
let recorder = CMSensorRecorder()
recorder.recordAccelerometer(forDuration: 2 * 3600) // 2 hours at 50 Hz
```

Query historical accelerometer data:
```swift
let recorder = CMSensorRecorder()
if let data = recorder.accelerometerData(from: startDate, to: endDate) {
    NSProcessInfo.processInfo().performExpiringActivity(withReason: "Process sensor data") { expired in
        for sample in data {
            guard !expired else { break }
            // store sample.startDate as cursor for next launch
        }
    }
}
```

Live pedometer update with pace and cadence:
```swift
let pedometer = CMPedometer()
pedometer.startUpdates(from: Date()) { data, error in
    guard let data = data, error == nil else { return }
    let pace = data.currentPace   // seconds per meter
    let cadence = data.currentCadence  // steps per second
}
```

Start and stop altimeter updates based on activity:
```swift
let altimeter = CMAltimeter()
if CMAltimeter.isRelativeAltitudeAvailable() {
    altimeter.startRelativeAltitudeUpdates(to: .main) { data, error in
        guard let data = data else { return }
        let altitude = data.relativeAltitude  // meters from start
    }
}
// When activity changes away from running/walking:
altimeter.stopRelativeAltitudeUpdates()
```

## Takeaways
- `CMSensorRecorder` is the first API to offer historical raw accelerometer access on Apple Watch — enabling custom algorithms over long time windows without keeping the app in the foreground.
- The pedometer's new `currentPace` and `currentCadence` properties are smoothed and responsive, making them far superior to computing pace manually from distance deltas.
- The altimeter is purpose-built for floor-level altitude changes, not fine-grained motion; environmental conditions and sealed cases can cause false readings over long durations.
- Core Motion's detect/engage/reflect pattern makes it useful in any app category — not just fitness — by enabling automatic context awareness without any user input.

---
_Source: WWDC15 Session 705 page (abstract, chapter summaries, code samples, and resource links)._
