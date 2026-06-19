# Build a Multi-Device Workout App
**WWDC23 · Session 10023** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10023/)

_Platforms:_ iOS 17, iPadOS 17, watchOS 10

## Overview
This session introduces new HealthKit APIs that let workout apps span Apple Watch, iPhone, and iPad. The centerpiece is a new **mirrored workout session** system: a workout running on Apple Watch can be mirrored to a companion iPhone app, keeping session state (running, paused, stopped) automatically in sync and enabling bidirectional data exchange between the devices. This allows developers to build cycling-computer-style experiences where Watch sensors drive the workout while iPhone provides a larger display interface.

The session also covers new cycling-specific HealthKit data types (speed, power, cadence, functional threshold power), the automatic Bluetooth sensor pipeline for collecting those metrics, and HealthKit's expansion to iPad — including the new authorization flow required for iPad's multi-window scene environment.

## Key Topics

### Mirrored Workout Sessions **[NEW]**
- A primary workout session runs on Apple Watch; a mirrored session is handed to the companion iPhone app when mirroring starts.
- If the iPhone app is not running, HealthKit launches it in the background and gives it 10 seconds to call a handler and start a live activity.
- The iPhone app registers a handler via `HKHealthStore` at launch to receive the mirrored session — this must happen every launch (foreground or background).
- Session state is kept automatically in sync: pausing the primary session pauses the mirrored session; resuming the mirrored session resumes the primary session.
- Both sessions expose a delegate (`HKWorkoutSessionDelegate`) that delivers state changes and events (pause, resume, etc.) on both devices.

### Bidirectional Data Exchange
- `HKWorkoutSession.sendData(toRemoteWorkoutSession:data:)` — send arbitrary `Data` from Watch to iPhone or iPhone to Watch.
- Receiving side: `HKWorkoutSessionDelegate.workoutSession(_:didReceiveDataFromRemoteDevice:)`.
- Use case (Watch → iPhone): package cycling metrics (speed, cadence, power) collected from Bluetooth sensors and display them on iPhone.
- Use case (iPhone → Watch): send water intake logged on iPhone to Apple Watch for storage.

### Cycling Data Types **[NEW]**
- `HKQuantityType` new types: cycling speed, cycling power, cycling cadence, functional threshold power (FTP).
- Bluetooth cycling sensors (power meters, cadence sensors) connect directly to Apple Watch and write data to HealthKit — no app code needed for the connection.
- HealthKit on Apple Watch automatically calculates and saves FTP from collected power data.
- Collected data can be sent to iPhone via `sendData(toRemoteWorkoutSession:data:)` for live display.

### HealthKit on iPad **[NEW]**
- HealthKit and the Health app now available on iPad; health data syncs via iCloud.
- Authorization flow on iPad differs: iPad apps can have multiple window scenes, so the authorization sheet must be presented over the correct scene.
- SwiftUI: use the new `.healthDataAccessRequest(store:readTypes:shareTypes:trigger:completion:)` view modifier from the `HealthKitUI` framework.
- UIKit: set `HKHealthStore.authorizationViewControllerPresenter` then call `requestAuthorization(toShare:read:completion:)`.

### Controlling Mirroring Lifecycle
- `HKWorkoutSession.startMirroringToCompanionDevice(completion:)` — call on Watch to start mirroring; launches iPhone app in background.
- `HKWorkoutSession.stopMirroringToCompanionDevice(completion:)` — call on Watch to stop; triggers `didDisconnectFromRemoteDeviceWithError` on iPhone's mirrored session delegate.
- `HKWorkoutBuilder.beginCollection(withStart:completion:)` — start collecting cycling metrics.
- `HKWorkoutBuilder.finishWorkout(completion:)` — save the workout on Apple Watch; automatically syncs to all signed-in devices including iPad.

## APIs & Frameworks

### HealthKit (watchOS 10 / iOS 17) **[NEW unless noted]**
- `HKWorkoutSession` — manages workout lifecycle (existing)
- `HKWorkoutSession.startMirroringToCompanionDevice(completion:)` **[NEW]** — start Watch→iPhone session mirroring
- `HKWorkoutSession.stopMirroringToCompanionDevice(completion:)` **[NEW]** — stop mirroring
- `HKWorkoutSession.sendData(toRemoteWorkoutSession:data:)` **[NEW]** — send arbitrary data between devices
- `HKWorkoutSessionDelegate.workoutSession(_:didReceiveDataFromRemoteDevice:)` **[NEW]** — receive data from paired device
- `HKWorkoutSessionDelegate.workoutSession(_:didDisconnectFromRemoteDeviceWithError:)` **[NEW]** — mirroring ended
- `HKHealthStore` mirroring start handler — set on iPhone to receive mirrored session **[NEW]**
- `HKWorkoutConfiguration` — configure activity type (e.g., `.cycling`) (existing)
- `HKHealthStore.startWatchApp(with:completion:)` — launch Watch app with configuration from iPhone (existing)
- `HKWorkoutBuilder` — builds and saves workout samples (existing)
- `HKWorkoutBuilder.beginCollection(withStart:completion:)` — start metric collection (existing)
- `HKWorkoutBuilder.finishWorkout(completion:)` — save workout (existing)

### New HealthKit Quantity Types **[NEW]**
- `HKQuantityType` for cycling speed — **[NEW]**
- `HKQuantityType` for cycling power — **[NEW]**
- `HKQuantityType` for cycling cadence — **[NEW]**
- `HKQuantityType` for functional threshold power (FTP) — **[NEW]**

### HealthKit on iPad **[NEW]**
- HealthKit framework — now available on iPadOS 17 **[NEW]**
- Health app on iPad — data synced via iCloud **[NEW]**
- `HKHealthStore.authorizationViewControllerPresenter` — specify window scene for UIKit auth sheet **[NEW]**
- `HKHealthStore.requestAuthorization(toShare:read:completion:)` — request HealthKit auth (existing)

### HealthKitUI **[NEW]**
- `HealthKitUI` framework — UI components for HealthKit authorization on iPadOS **[NEW]**
- `.healthDataAccessRequest(store:readTypes:shareTypes:trigger:completion:)` — SwiftUI view modifier for authorization sheet **[NEW]**

## Code Highlights

iPhone app: register mirroring handler at launch and set up session delegate:
```swift
// In iPhone app startup — call every launch
healthStore.workoutSessionMirroringStartHandler = { mirroredSession in
    mirroredSession.delegate = self
    self.mirroredSession = mirroredSession  // keep a strong reference
}
```

Watch app: start mirroring then start the primary session:
```swift
session.startMirroringToCompanionDevice { success, error in
    // HealthKit launches iPhone app in background
}
try session.startActivity(with: Date())
```

Sending cycling metrics from Watch to iPhone:
```swift
let data = encodeCyclingMetrics(speed: speed, cadence: cadence, power: power)
primarySession.sendData(toRemoteWorkoutSession: data) { success, error in }

// On iPhone mirrored session delegate:
func workoutSession(_ session: HKWorkoutSession, didReceiveDataFromRemoteDevice data: Data) {
    let metrics = decodeCyclingMetrics(data)
    updateUI(with: metrics)
}
```

SwiftUI authorization sheet for iPad:
```swift
import HealthKitUI

.healthDataAccessRequest(
    store: healthStore,
    readTypes: [energyType, cyclingSpeedType, heartRateType, workoutType],
    shareTypes: [],
    trigger: needsAuth
) { result in
    // handle result
}
```

## Takeaways
- Use `startMirroringToCompanionDevice` to hand a Watch workout session to a companion iPhone app — HealthKit handles state sync, launch, and reconnection automatically.
- Use `sendData(toRemoteWorkoutSession:data:)` for live bidirectional data exchange; this enables cycling computers, hydration trackers, and custom metrics displays.
- Four new cycling quantity types (speed, power, cadence, FTP) are now available; FTP is computed and saved automatically from Bluetooth sensor data.
- HealthKit is now on iPad; adopt `HealthKitUI`'s `.healthDataAccessRequest` modifier to correctly present the auth sheet in multi-window SwiftUI apps.

---
_Source: WWDC23 Session 10023 page (abstract, chapter summaries, code samples, and resource links)._
