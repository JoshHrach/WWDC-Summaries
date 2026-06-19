# Meet Core Location Monitor
**WWDC23 · Session 10147** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10147/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, watchOS 10

## Overview
`CLMonitor` is a new top-level Swift actor that replaces and supersedes `CLLocationManager`'s region monitoring and beacon ranging APIs. It provides a unified, Swift-concurrency-native interface for monitoring geographic and beacon conditions, delivering state-change events via an async sequence. Events are persisted across app launches — if a monitored condition changes state while the app is terminated, Core Location will relaunch the app in the background and deliver the event.

The key abstraction is a **Condition** (geographic region or beacon identity) that CLMonitor tracks and for which it maintains a **Record** containing the most recent observed **Event** (state: satisfied/unsatisfied/unknown, timestamp, and optional refinement for wildcard beacon conditions).

## Key Topics

### CLMonitor Overview
- A Swift `actor` — all property access and mutations must be `await`ed; thread safety is built in.
- One `CLMonitor` instance per name at a time — multiple monitors with different names are allowed.
- Created/opened with `CLMonitor(name:)` — if an existing monitor with that name exists, it is opened; otherwise a new one is created.
- Conditions and their states persist across app launches.

### Conditions
**CircularGeographicCondition:**
- Replaces `CLCircularRegion`.
- Defined by a `CLLocationCoordinate2D` center and a radius in meters.
- State is `.satisfied` when inside the radius; `.unsatisfied` when outside.

**BeaconIdentityCondition:**
- Replaces `CLBeaconIdentityConstraint` / `CLBeaconRegion`.
- Matches iBeacon devices by UUID alone, UUID + major, or UUID + major + minor.
- Omitting minor (or major + minor) creates a wildcard that matches any value for those fields.
- Use cases: detect entry to any company site (UUID only), a specific site (UUID + major), or a specific section of a site (UUID + major + minor).

### Adding and Removing Conditions
- `monitor.add(condition, identifier: String)` — associates a condition with an alphanumeric identifier; replaces any existing condition with the same identifier.
- Optional `assuming:` parameter: set the initial assumed state (`.satisfied`, `.unsatisfied`) if the app already has context — Core Location will correct this if the assumption is wrong.
- `monitor.remove(identifier: String)` — removes the condition and its record.

### Records
- A **Record** stores the most recently observed **Event** for a condition.
- `monitor.record(for: identifier)` — returns `CLMonitor.Record?` for a given identifier.
- `monitor.identifiers` — the list of all currently monitored identifiers; no need to maintain your own list.
- `Record.condition` — the monitored condition.
- `Record.lastEvent` — the most recently delivered event.
- `Event.state` — `.satisfied`, `.unsatisfied`, `.unknown`.
- `Event.date` — when the state was observed.
- `Event.refinement` — for wildcard `BeaconIdentityCondition` matches: a fully-specified condition with the actual UUID/major/minor of the detected beacon. `nil` when condition is unsatisfied.

### Receiving Events
- `monitor.events` — an `AsyncSequence` of `CLMonitor.Event` that streams state changes.
- Pattern: wrap a `for await event in monitor.events` loop inside a `Task`.
- Events are only promoted to `lastEvent` after the app handles them from the sequence.
- Always keep a `Task` awaiting `monitor.events` during the app's lifecycle.

### Background Launching
- If the app is terminated and a monitored condition changes state, Core Location will relaunch the app in the background (requires the app to have location authorization).
- On relaunch: reinitialize the monitor by name and begin awaiting events immediately.
- Best place: `application(_:didFinishLaunchingWithOptions:)` delegate callback.
- Do not use `CLMonitor` from widgets or extensions — this would launch the host app instead and complicate the single-instance-per-name requirement.

### Key Requirements and Recommendations
- Only one instance per name may be open at a time.
- Always have a `Task` awaiting `monitor.events` so events are not missed.
- Re-init and re-await on every app launch.
- Trust CLMonitor's persisted state rather than maintaining a parallel state table; if a separate representation is needed (e.g., for SwiftUI), keep it for UI only and don't use it for reasoning about expected events.
- Only available on iOS, iPadOS, macOS, watchOS — **not** on visionOS (see session 10146).

## APIs & Frameworks
- `CoreLocation` framework
- `CLMonitor` **[NEW]** — Swift actor for unified condition monitoring
- `CLMonitor.init(name:)` **[NEW]** — creates or opens a named monitor
- `CLMonitor.add(_:identifier:)` **[NEW]** — adds a condition for monitoring
- `CLMonitor.add(_:identifier:assuming:)` **[NEW]** — adds with an initial assumed state
- `CLMonitor.remove(_:)` **[NEW]** — removes a condition by identifier
- `CLMonitor.record(for:)` **[NEW]** — retrieves a `Record` by identifier
- `CLMonitor.identifiers` **[NEW]** — list of all monitored identifiers
- `CLMonitor.events` **[NEW]** — `AsyncSequence` of `CLMonitor.Event` state changes
- `CLMonitor.Record` **[NEW]** — persisted condition state container
- `CLMonitor.Record.condition` **[NEW]** — the associated condition
- `CLMonitor.Record.lastEvent` **[NEW]** — most recently delivered event
- `CLMonitor.Event` **[NEW]** — state change event
- `CLMonitor.Event.state` **[NEW]** — `.satisfied`, `.unsatisfied`, `.unknown`
- `CLMonitor.Event.date` **[NEW]** — timestamp of the observed state
- `CLMonitor.Event.refinement` **[NEW]** — fully-specified beacon identity when wildcard matched
- `CLMonitor.Event.identifier` **[NEW]** — identifier of the condition that changed
- `CLMonitor.Condition` **[NEW]** — base protocol for monitorable conditions
- `CLMonitor.CircularGeographicCondition` **[NEW]** — geographic circle condition; center + radius
- `CLMonitor.BeaconIdentityCondition` **[NEW]** — beacon identity condition; UUID, major, minor (optionally wildcarded)
- `CLMonitor.State` **[NEW]** — `.satisfied`, `.unsatisfied`, `.unknown`
- `CLLocationCoordinate2D` — latitude/longitude center for `CircularGeographicCondition`

## Code Highlights

Create a monitor and add a geographic condition:
```swift
let monitor = await CLMonitor("AppMonitor")
let appleHQ = CLLocationCoordinate2D(latitude: 37.3318, longitude: -122.0312)
let condition = CLMonitor.CircularGeographicCondition(center: appleHQ, radius: 100)
await monitor.add(condition, identifier: "AppleHQ")
```

Add a beacon condition with wildcard major/minor (any beacon with UUID):
```swift
let uuid = UUID(uuidString: "12345678-1234-1234-1234-1234567890AB")!
let beaconCondition = CLMonitor.BeaconIdentityCondition(uuid: uuid)
await monitor.add(beaconCondition, identifier: "AnyAppleBeacon")
```

Add a beacon condition matching a specific site (UUID + major):
```swift
let siteCondition = CLMonitor.BeaconIdentityCondition(uuid: uuid, major: 1)
await monitor.add(siteCondition, identifier: "AppleParkSite", assuming: .unsatisfied)
```

Await events and respond to state changes:
```swift
Task {
    for await event in monitor.events {
        switch event.state {
        case .satisfied:
            print("Condition \(event.identifier) is now satisfied")
            if let refinement = event.refinement as? CLMonitor.BeaconIdentityCondition {
                print("Matched beacon major: \(refinement.major ?? -1), minor: \(refinement.minor ?? -1)")
            }
        case .unsatisfied:
            print("Condition \(event.identifier) is no longer satisfied")
        case .unknown:
            break
        @unknown default:
            break
        }
    }
}
```

Access persisted records on launch:
```swift
let monitor = await CLMonitor("AppMonitor")
for identifier in await monitor.identifiers {
    if let record = await monitor.record(for: identifier) {
        print("\(identifier): \(record.lastEvent?.state ?? .unknown)")
    }
}
```

## Takeaways
- `CLMonitor` replaces delegate-based region monitoring and beacon ranging with a clean async sequence API; state is persisted across app launches with no extra bookkeeping.
- Always re-init and re-await `monitor.events` on every app launch — Core Location will background-launch the app when monitored conditions change state.
- Use wildcard `BeaconIdentityCondition` (UUID only, or UUID + major) for multi-level beacon hierarchies; the `Event.refinement` property tells you which specific beacon triggered the match.
- `CLMonitor` is not supported on visionOS — background event delivery and region monitoring are unavailable on that platform.

---
_Source: WWDC23 Session 10147 page (abstract, transcript, and resource links)._
