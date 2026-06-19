# Meet Core Location for Spatial Computing
**WWDC23 · Session 10146** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10146/)

_Platforms:_ visionOS 1 (also covers compatible iPad/iPhone app behavior)

## Overview
This session explains how Core Location works on visionOS — what data is available, how authorization and privacy are handled, and how the spatial computing context changes the definition of "in use" for location access. It also addresses how existing iPhone and iPad apps that use Core Location will behave when running in compatibility mode on visionOS.

The fundamentals of Core Location's API (requesting authorization, using `CLLocationUpdate` for live updates) are unchanged. What visionOS introduces is a new model for determining when an app is "in use" and therefore eligible to receive location data, driven by where the user is actively looking rather than which app is foregrounded in the traditional sense.

## Key Topics

### Location Accuracy on visionOS
- A standalone visionOS device achieves accuracy similar to a Mac: approximately 100 meters.
- Suitable for coarse location uses: finding nearby restaurants, parks, weather by city, etc.
- When a nearby iPhone is available, the visionOS device leverages existing inter-device connections to achieve iPhone-level accuracy.
- This enables precision use cases: precise weather, geotagging, navigation, etc.

### Authorization Model
- Location access requires explicit user permission before any data is delivered — unchanged from iOS.
- Request authorization with `CLLocationManager.requestWhenInUseAuthorization()`.
- Provide `NSLocationWhenInUseUsageDescription` in `Info.plist`; this string appears in the permission prompt.
- User can grant: "Allow Once," "While Using the App," or deny; can also choose precise vs. approximate location.
- Ask for authorization in a context where the app genuinely needs location — users are more likely to grant access when the request is clearly relevant.

### "While In Use" Semantics on visionOS
The definition of "in use" differs significantly from iPhone:
- **Fully immersive app** (Full Space): as long as the user is running the app, it is considered in use.
- **Windowed app** (Shared Space): an app is in use only when the **user has recently looked at it** (i.e., directed their gaze/attention toward the app window).
  - If the user looks away, Core Location stops delivering updates to that app.
  - A short **grace period** (a few seconds) allows both apps to remain "in use" briefly when the user shifts gaze from one to another.
  - If both windows are in view simultaneously and the user is looking at both, both can receive updates.
- Apps do not receive location updates while not running on visionOS.
- Monitoring APIs (region monitoring, visit monitoring, `CLMonitor`) do **not** deliver events on visionOS.

### Compatible iPhone/iPad Apps on visionOS
- Apps that request `.authorizedAlways` will have their authorization request **redirected to `.authorizedWhenInUse`** — "Always" authorization is not available.
- "Always" will not appear as an option in the Settings location permission UI.
- Apps relying on background region monitoring or `CLMonitor` will not receive events on visionOS.
- Developers should audit compatible apps for assumptions that region/visit monitoring always delivers events.

### Using CLLocationUpdate (Live Updates)
- The modern, Swift-concurrency-friendly API for receiving location updates.
- `CLLocationUpdate.liveUpdates()` — async sequence of location updates.
- Pair with `CLLocationManager.requestWhenInUseAuthorization()` before starting updates.
- Details covered in the companion session "Discover Streamlined Location Updates."

## APIs & Frameworks
- `CoreLocation` framework — geographic and beacon location services
- `CLLocationManager` — manages location services and authorization
- `CLLocationManager.requestWhenInUseAuthorization()` — requests while-in-use location permission
- `NSLocationWhenInUseUsageDescription` (`Info.plist`) — usage description string shown in authorization prompt
- `CLAuthorizationStatus` — `.authorizedWhenInUse`, `.denied`, `.notDetermined`, `.restricted`
- `CLLocationUpdate` **[visionOS 1]** — modern Swift async location update type
- `CLLocationUpdate.liveUpdates()` **[NEW]** — async sequence delivering continuous location updates
- `CLLocation` — location value with coordinate, altitude, accuracy, timestamp
- `CLLocationCoordinate2D` — latitude/longitude coordinate
- `CLAccuracyAuthorization` — `.fullAccuracy` (precise), `.reducedAccuracy` (approximate)
- Region monitoring (`CLCircularRegion`, `CLRegion`) — not supported on visionOS (no events delivered)
- `CLMonitor` — not supported on visionOS (no events delivered; see session 10147)
- `CLVisit` / visit monitoring — not supported on visionOS

## Code Highlights

Request authorization and start live location updates:
```swift
import CoreLocation

class LocationViewModel: ObservableObject {
    private let manager = CLLocationManager()
    @Published var location: CLLocation?

    init() {
        manager.requestWhenInUseAuthorization()
        Task {
            for await update in CLLocationUpdate.liveUpdates() {
                self.location = update.location
            }
        }
    }
}
```

Info.plist usage description (required):
```xml
<key>NSLocationWhenInUseUsageDescription</key>
<string>This app uses your location to show nearby points of interest.</string>
```

Checking authorization status before using location:
```swift
let manager = CLLocationManager()
switch manager.authorizationStatus {
case .authorizedWhenInUse:
    // begin receiving updates
case .denied, .restricted:
    // explain to user
case .notDetermined:
    manager.requestWhenInUseAuthorization()
default:
    break
}
```

## Takeaways
- Core Location works on visionOS with the same authorization API as iOS; only the definition of "in use" changes — a windowed app is in use only when the user is actively looking at it.
- Standalone visionOS achieves ~100m accuracy; proximity to an iPhone improves this to iPhone-level precision automatically.
- "Always" authorization is not available on visionOS; all requests are redirected to "When In Use."
- Background monitoring APIs (region monitoring, `CLMonitor`, visit monitoring) do not fire events on visionOS — audit compatible apps for assumptions that rely on these deliveries.

---
_Source: WWDC23 Session 10146 page (abstract, transcript, and resource links)._
