# Discover Streamlined Location Updates
**WWDC23 · Session 10180** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10180/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, tvOS 17, watchOS 10, visionOS 1

## Overview
Core Location engineer Siraj introduces `CLLocationUpdate` — a new Swift-native class designed from the ground up for modern Swift concurrency. It replaces the delegate-based `CLLocationManager` update flow with an `AsyncSequence` that can be iterated with `for/try/await`, bringing idiomatic concurrency support to location updates and eliminating the need for explicit start/stop calls.

The session also covers automatic pause/resume for battery efficiency and the new `CLBackgroundActivitySession` that enables background location access for apps that do not yet have a Live Activity.

## Key Topics

### CLLocationUpdate and liveUpdates **[NEW]**
The new `CLLocationUpdate` class exposes a single static factory method, `liveUpdates()`, which returns an `AsyncSequence` called `Updates`. Each element yielded is a `CLLocationUpdate` value containing:
- **`location`** (`CLLocation?`) — the current location, or `nil` if unavailable.
- **`isStationary`** (`Bool`) — `true` when the device has been stationary long enough for the system to automatically pause updates.

Because it is a standard `AsyncSequence`, all standard operations work: `first(where:)`, `filter`, `prefix`, `map`, etc. Stopping updates is as simple as `break`ing out of the `for` loop — no explicit `stopUpdatingLocation()` call needed.

### LiveConfiguration **[NEW]**
`CLLocationUpdate.liveUpdates(_:)` takes an optional `LiveConfiguration` enum argument, replacing `CLActivityType`:
- `.default` — general-purpose (used when no argument is passed)
- `.automotiveNavigation` — optimized for in-vehicle navigation
- `.otherNavigation` — non-automotive navigation (cycling, etc.)
- `.fitness` — walking/running activity
- `.airborne` — aviation use

If the app previously used `CLActivityType`, pick the corresponding `LiveConfiguration` member when migrating.

### Background Location Updates
Two paths for background location:

1. **Live Activity** — the preferred approach. While a Live Activity is active, the app receives location updates in the background with no additional setup.

2. **CLBackgroundActivitySession [NEW]** — for apps without a Live Activity. Creating a `CLBackgroundActivitySession` instance displays the blue background location indicator and keeps the app "effectively in-use," allowing background location access. Rules:
   - Hold the session object as a property (not a local variable) — deallocation invalidates the session.
   - Must be started from the foreground; can be rejoined from the background.
   - App still needs `location` in its `UIBackgroundModes` array.
   - Call `invalidate()` to end the session explicitly, or let the object be destroyed.

### Automatic Pause and Resume
When the device is stationary for a sufficient time, the system automatically pauses updates. The last update before pausing has `isStationary = true`. When the device moves again, updates automatically resume with `isStationary = false`. No app code required — simply check `isStationary` in the loop to react if needed.

### App Lifecycle with Background Updates
While receiving background updates, the app may be:
- **Background running** → **Suspended**: happens during automatic pause or when no fix is available. The system resumes the app automatically when updates are available.
- **Suspended** → **Terminated**: possible due to user close or resource constraints.
- **Terminated** → **Background running** (system launch): the system re-launches the app when updates are available. On re-launch, the app must: (1) restart `liveUpdates()`, and (2) if using `CLBackgroundActivitySession`, recreate it (this rejoins the existing session, not starts a new one). Both should be done in `application(_:didFinishLaunchingWithOptions:)`.

## APIs & Frameworks

### Core Location — CLLocationUpdate **[NEW]**
- `CLLocationUpdate` — new class representing a single location update **[NEW]**
- `CLLocationUpdate.liveUpdates()` — static method returning `CLLocationUpdate.Updates` async sequence **[NEW]**
- `CLLocationUpdate.liveUpdates(_ configuration: LiveConfiguration)` — variant with explicit configuration **[NEW]**
- `CLLocationUpdate.location` — the location value (`CLLocation?`) **[NEW]**
- `CLLocationUpdate.isStationary` — automatic pause/resume signal (`Bool`) **[NEW]**
- `CLLocationUpdate.Updates` — `AsyncSequence` of `CLLocationUpdate` elements **[NEW]**
- `CLLocationUpdate.LiveConfiguration` — enum for pre-baked configurations **[NEW]**
  - `.default` **[NEW]**
  - `.automotiveNavigation` **[NEW]**
  - `.otherNavigation` **[NEW]**
  - `.fitness` **[NEW]**
  - `.airborne` **[NEW]**

### Core Location — CLBackgroundActivitySession **[NEW]**
- `CLBackgroundActivitySession` — maintains background location visibility indicator **[NEW]**
- `CLBackgroundActivitySession.init()` — creates and starts a new session (foreground only) **[NEW]**
- `CLBackgroundActivitySession.invalidate()` — ends the session **[NEW]**

### Core Location — Existing (for context)
- `CLLocationManager` — legacy delegate-based location manager
- `CLLocation` — location value type (unchanged)
- `CLActivityType` — replaced by `LiveConfiguration` in new API

### Concurrency
- `AsyncSequence` — the type returned by `liveUpdates()`; supports `for/try/await` iteration, `first(where:)`, `filter`, `map`, etc.

## Code Highlights

```swift
// Minimal: start foreground location updates
import CoreLocation

for try await update in CLLocationUpdate.liveUpdates() {
    guard let location = update.location else { continue }
    print("Current location: \(location)")
    if update.isStationary { break }  // stop when device has been still
}
```

```swift
// With explicit configuration
for try await update in CLLocationUpdate.liveUpdates(.automotiveNavigation) {
    guard let location = update.location else { continue }
    updateNavigationUI(with: location)
}
```

```swift
// AsyncSequence composition: find first high-speed update
let fastUpdate = try await CLLocationUpdate.liveUpdates()
    .first(where: { ($0.location?.speed ?? 0) > 30 })
```

```swift
// Background location via CLBackgroundActivitySession
class AppDelegate: UIApplicationDelegate {
    var backgroundActivity: CLBackgroundActivitySession?

    func enableBackgroundLocation() {
        // Must hold as a property — local var would be immediately deallocated
        backgroundActivity = CLBackgroundActivitySession()
    }

    // Re-create on background launch to rejoin existing session
    func application(_ app: UIApplication,
                     didFinishLaunchingWithOptions options: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        backgroundActivity = CLBackgroundActivitySession()
        Task {
            for try await update in CLLocationUpdate.liveUpdates() {
                guard let location = update.location else { continue }
                handleBackgroundLocation(location)
            }
        }
        return true
    }
}
```

## Takeaways
- `CLLocationUpdate.liveUpdates()` is the modern replacement for `CLLocationManager` delegate callbacks — one line of code starts location updates with full Swift concurrency support.
- Stop updates by breaking out of the loop; automatic pause/resume handles stationary devices without any app code.
- Use `CLBackgroundActivitySession` (stored as a property, not a local variable) to keep background location access alive for apps without a Live Activity.
- On background re-launch, immediately recreate both the `liveUpdates()` iteration and the `CLBackgroundActivitySession` in `didFinishLaunchingWithOptions` to restore background location continuity.

---
_Source: WWDC23 Session 10180 page (abstract, chapter summaries, code samples, and resource links)._
