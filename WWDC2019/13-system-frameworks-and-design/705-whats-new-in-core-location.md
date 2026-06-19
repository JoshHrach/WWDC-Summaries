# What's New in Core Location
**WWDC19 · Session 705** · [Watch](https://developer.apple.com/videos/play/wwdc2019/705/)

_Platforms:_ iOS 13, iPadOS 13, watchOS 6, macOS Catalina 10.15, tvOS 13

## Overview
iOS 13 brings significant changes to the Core Location authorization model, giving users finer control over how and when apps access their location. The two major pillars are a revamped authorization flow that introduces "provisional always authorization" and the new "Allow Once" (temporary) authorization option, along with expanded capabilities for When In Use authorization.

The session also introduces a fully redesigned beacon ranging API. Beacon ranging no longer requires Always authorization—When In Use authorization is now sufficient—and a new `CLBeaconIdentityConstraint` type replaces the overloaded use of `CLBeaconRegion` for expressing which beacons an app is interested in.

## Key Topics

### Revised Always Authorization Flow
- In iOS 13, all location authorization prompts start with a single prompt offering "Allow While Using App," "Allow Once," and "Don't Allow." There is no longer an "Allow Always" option on the initial prompt.
- When an app calls `requestAlwaysAuthorization()`, the user sees the standard prompt. If they choose "Allow While Using App," the app enters **provisional always authorization**: it behaves as if it has Always authorization and can set up geofences, but events are held by the system until a follow-up prompt asks the user to upgrade or downgrade.
- The upgrade prompt (with "Allow Always" button) is shown at a system-chosen moment when the user is not busy. Until then, events may be dropped or displaced by newer ones.
- This process can only be initiated once per app.

### Expanded When In Use Authorization
- In iOS 12, region monitoring, significant-location-change monitoring, and visit monitoring required Always authorization.
- In iOS 13, all Core Location APIs are available under When In Use authorization—they simply do not deliver events to the app when the app is not in use.
- "In use" covers the foreground period, a brief grace period after entering background, and the background-location-updates indicator period (when `allowsBackgroundLocationUpdates = true`).
- Local notifications with `UNLocationNotificationTrigger` can bridge the gap: the user taps the notification, the app enters the foreground, and the app becomes "in use" right where location context is needed.

### Temporary (Allow Once) Authorization
- Selecting "Allow Once" grants temporary When In Use authorization that resets to `.notDetermined` when the app is no longer in use.
- The app can and should re-request authorization the next time the user performs an action that requires it.
- Using `allowsBackgroundLocationUpdates = true` extends the in-use period across a background session, keeping temporary authorization active (e.g., for a run-tracking or navigation app).

### Revamped Beacon Ranging API
- Introduced `CLBeaconIdentityConstraint` **[NEW]** to express beacon interest independently of region shape.
- `CLBeaconRegion` is now initialized with a `CLBeaconIdentityConstraint` rather than UUID/major/minor directly.
- `startRangingBeacons(satisfying:)` and `stopRangingBeacons(satisfying:)` **[NEW]** replace the old ranging methods.
- Beacon ranging now works with When In Use authorization, removing the previous requirement for Always authorization.

## APIs & Frameworks

### Core Location
- `CLLocationManager` — central manager class
- `CLLocationManagerDelegate` — delegate protocol
- `CLLocationManager.requestWhenInUseAuthorization()` — request When In Use auth
- `CLLocationManager.requestAlwaysAuthorization()` — request Always auth (now triggers provisional flow)
- `CLAuthorizationStatus.authorizedWhenInUse` — When In Use status
- `CLAuthorizationStatus.authorizedAlways` — Always status (may be provisional)
- `CLAuthorizationStatus.notDetermined` — temporary auth resets here
- `CLLocationManager.allowsBackgroundLocationUpdates` — extend in-use period with background indicator
- `CLBeaconIdentityConstraint` **[NEW]** — expresses UUID / major / minor combination with wildcard support
- `CLBeaconRegion.init(beaconIdentityConstraint:identifier:)` **[NEW]** — create region from identity constraint
- `CLBeaconRegion.beaconIdentityConstraint` **[NEW]** — retrieve constraint from region
- `CLLocationManager.startRangingBeacons(satisfying:)` **[NEW]** — start ranging with identity constraint
- `CLLocationManager.stopRangingBeacons(satisfying:)` **[NEW]** — stop ranging with identity constraint
- `CLLocationManagerDelegate.locationManager(_:didRange:satisfying:)` **[NEW]** — ranging update callback
- `CLLocationManagerDelegate.locationManager(_:didFailRangingFor:error:)` **[NEW]** — ranging failure callback
- `CLBeacon` — object representing a detected beacon with proximity info
- `CLRegion` / `CLCircularRegion` — geographic region types
- Region monitoring: `startMonitoring(for:)`, `stopMonitoring(for:)`
- `locationManager(_:didDetermineState:for:)` — region state callback (inside/outside/unknown)

### User Notifications (related usage)
- `UNLocationNotificationTrigger` — trigger a local notification when user enters a geographic region (works with When In Use authorization)

## Code Highlights

Setting up When In Use authorization and beacon ranging without Always authorization:

```swift
// Request When In Use (sufficient for ranging in iOS 13)
locationManager.requestWhenInUseAuthorization()

// Create identity constraint (UUID only = wildcard on major & minor)
let constraint = CLBeaconIdentityConstraint(uuid: museumUUID)

// Create region using the constraint
let beaconRegion = CLBeaconRegion(beaconIdentityConstraint: constraint,
                                  identifier: "MuseumRegion")
locationManager.startMonitoring(for: beaconRegion)

// In delegate: react to region state
func locationManager(_ manager: CLLocationManager,
                     didDetermineState state: CLRegionState,
                     for region: CLRegion) {
    guard let beaconRegion = region as? CLBeaconRegion else { return }
    if state == .inside {
        manager.startRangingBeacons(satisfying: beaconRegion.beaconIdentityConstraint)
    } else {
        manager.stopRangingBeacons(satisfying: beaconRegion.beaconIdentityConstraint)
    }
}
```

Persistent folder access with security-scoped bookmarks (related pattern for location data persistence):
```swift
// Store bookmark
let bookmarkData = try url.bookmarkData(options: .minimalBookmark)

// Restore URL
var stale = false
let resolvedURL = try URL(resolvingBookmarkData: bookmarkData,
                          bookmarkDataIsStale: &stale)
```

## Takeaways
- Always authorization now uses a provisional two-step flow; apps should plan for dropped events during the provisional period and design accordingly.
- When In Use authorization in iOS 13 covers all Core Location services—apps should evaluate whether Always authorization is truly necessary before requesting it.
- The new "Allow Once" option returns to `.notDetermined` after use; apps should re-request authorization contextually rather than eagerly.
- `CLBeaconIdentityConstraint` decouples beacon identification from region definition, and beacon ranging now works entirely with When In Use authorization.

---
_Source: WWDC19 Session 705 page (abstract, chapter summaries, code samples, and resource links)._
