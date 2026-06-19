# Build Location-Aware Enterprise Apps
**WWDC20 · Session 10140** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10140/)

_Platforms:_ iOS 14, iPadOS 14

## Overview
This session uses Apple's own Caffè Macs campus cafeteria app as a case study for building location-aware enterprise apps. The session covers three layers of location awareness: user-controlled preferences (no location permission required), on-site micro-location using iBeacons with Core Location, and international macro-location through `DateFormatter` and `NumberFormatter`.

The Caffè Macs app allows employees to browse menus, order food, and pay with Apple Pay. Location-awareness ensures the app shows the correct cafeteria menu based on where the employee is standing among multiple nearby venues. The session also demonstrates how apps can gracefully fall back to user preferences when location permission is denied or unavailable.

Privacy is a core theme throughout: the app requests only "When in Use" authorization, explains why location is needed, and gives users explicit control over their default location preference independently of system location services.

## Key Topics

**User Preference Layer (No Location Required)**
A simple setting lets users choose "Closest to Me" (uses Core Location) or a specific Caffè (no location needed). The preference is stored in `UserDefaults` and read at launch to set the initial app state. This allows full control and graceful degradation without any location permission.

**Core Location Authorization**
The app requests `.authorizedWhenInUse` (not `.authorizedAlways`) since it only needs location while actively in use. The `NSLocationWhenInUseUsageDescription` Info.plist key provides the purpose string. The `locationManager(_:didChangeAuthorization:)` delegate method handles status changes, including denial, restriction, and the new "Allow Once" grant. Device support is checked with `CLLocationManager.isMonitoringAvailable(for:)` and `CLLocationManager.isRangingAvailable()` before activating beacon features.

**iBeacon Region Monitoring (Stage 1)**
Each beacon broadcasts a proximity UUID, major value, and minor value. `CLBeaconIdentityConstraint` and `CLBeaconRegion` are used to register for monitoring with `locationManager.startMonitoring(for:)`. When the device enters or exits the beacon region, `locationManager(_:didEnterRegion:)` and `didExitRegion:` are called.

**iBeacon Ranging (Stage 2)**
After entering a region, `startRangingBeacons(satisfying:)` determines relative proximity. The `didRangeBeacons(_:in:)` delegate method delivers an array of `CLBeacon` objects sorted by proximity. Major and minor values distinguish between venue sub-locations. `CLProximity` values (`.immediate`, `.near`, `.far`, `.unknown`) drive the UI decision.

**Internationalization**
- `DateFormatter` with `dateStyle` and `timeStyle` produces locale-appropriate date/time strings.
- `NumberFormatter` with `.currency` style plus explicit `currencyCode` and `locale` formats prices correctly for the physical location of the business, independent of the device's locale.

## APIs & Frameworks

### Core Location
- `CLLocationManager` — manages location services
- `CLLocationManager.requestWhenInUseAuthorization()` — requests "When in Use" permission
- `CLLocationManagerDelegate.locationManager(_:didChangeAuthorization:)` — handles auth status changes
- `CLAuthorizationStatus` — `.notDetermined`, `.restricted`, `.denied`, `.authorizedWhenInUse`, `.authorizedAlways`
- `CLLocationManager.isMonitoringAvailable(for:)` — checks beacon region monitoring support
- `CLLocationManager.isRangingAvailable()` — checks beacon ranging support
- `CLBeaconIdentityConstraint(uuid:)` **[NEW in iOS 13]** — identifies beacons by UUID
- `CLBeaconIdentityConstraint(uuid:major:)`, `CLBeaconIdentityConstraint(uuid:major:minor:)` — more specific constraints
- `CLBeaconRegion(beaconIdentityConstraint:identifier:)` **[NEW in iOS 13]** — monitoring region for beacons
- `CLLocationManager.startMonitoring(for:)` — starts region monitoring
- `CLLocationManager.stopMonitoring(for:)` — stops region monitoring
- `CLLocationManagerDelegate.locationManager(_:didEnterRegion:)` — beacon region entered
- `CLLocationManagerDelegate.locationManager(_:didExitRegion:)` — beacon region exited
- `CLLocationManager.startRangingBeacons(satisfying:)` **[NEW in iOS 13]** — starts beacon ranging
- `CLLocationManager.stopRangingBeacons(satisfying:)` **[NEW in iOS 13]** — stops ranging
- `CLLocationManagerDelegate.locationManager(_:didRangeBeacons:in:)` — delivers ranged beacons
- `CLBeacon` — represents a detected beacon with `uuid`, `major`, `minor`, `proximity`, `accuracy`, `rssi`
- `CLProximity` — `.immediate`, `.near`, `.far`, `.unknown`
- `CLBeaconMajorValue`, `CLBeaconMinorValue` — typedefs for major/minor values
- Info.plist key: `NSLocationWhenInUseUsageDescription`

### Foundation
- `UserDefaults.standard.set(_:forKey:)` — persists user location preference
- `UserDefaults.standard.integer(forKey:)` — reads saved preference
- `DateFormatter` — localizes date/time strings
  - `dateFormatter.dateStyle` — `.short`, `.medium`, `.long`, `.full`
  - `dateFormatter.timeStyle` — `.short`, `.medium`, etc.
  - `dateFormatter.string(from:)` — produces locale-appropriate string
- `NumberFormatter` — formats numbers with locale and currency awareness
  - `formatter.numberStyle = .currency` — currency formatting
  - `formatter.currencyCode` — ISO 4217 currency code (e.g., `"USD"`, `"CAD"`)
  - `formatter.locale` — determines decimal separator and grouping separator
  - `formatter.string(from:)` — produces formatted currency string

## Code Highlights

Storing and reading a location preference:
```swift
UserDefaults.standard.set(defaultLocation.id, forKey: "defaultLocationId")
let defaultLocationId = UserDefaults.standard.integer(forKey: "defaultLocationId")
```

Requesting location authorization and handling changes:
```swift
locationManager.requestWhenInUseAuthorization()

func locationManager(_ manager: CLLocationManager,
                     didChangeAuthorization status: CLAuthorizationStatus) {
    switch status {
    case .restricted, .denied: disableLocationFeatures()
    case .authorizedWhenInUse, .authorizedAlways: enableLocationFeatures()
    case .notDetermined: break
    }
}
```

Region monitoring for iBeacons:
```swift
func monitorBeacons() {
    guard CLLocationManager.isMonitoringAvailable(for: CLBeaconRegion.self) else { return }
    let constraint = CLBeaconIdentityConstraint(uuid: proximityUUID)
    let region = CLBeaconRegion(beaconIdentityConstraint: constraint, identifier: beaconID)
    locationManager.startMonitoring(for: region)
}
```

Ranging and acting on proximity:
```swift
func locationManager(_ manager: CLLocationManager,
                     didRangeBeacons beacons: [CLBeacon], in region: CLBeaconRegion) {
    guard let nearest = beacons.first else { return }
    let major = CLBeaconMajorValue(truncating: nearest.major)
    let minor = CLBeaconMinorValue(truncating: nearest.minor)
    switch nearest.proximity {
    case .near, .immediate: displayInformation(for: major, and: minor)
    default: handleUnknownOrFarBeacon(for: major, and: minor)
    }
}
```

Currency formatting with explicit locale:
```swift
let formatter = NumberFormatter()
formatter.currencyCode = "USD"
formatter.locale = Locale(identifier: "en_US")
formatter.numberStyle = .currency
let result = formatter.string(from: amount) // "$12.99"
```

## Takeaways
- Layer location awareness: user preferences (no permission), then Core Location (When in Use only), then iBeacons — each layer degrades gracefully to the previous when unavailable.
- Request only the minimum location authorization needed: `requestWhenInUseAuthorization()` suffices for beacon-based venue detection; always handle denial and "Allow Once" scenarios explicitly.
- Use two-stage iBeacon detection — region monitoring for presence detection, then ranging for proximity — to preserve battery by only ranging when a beacon region is active.
- Set `NumberFormatter.currencyCode` explicitly based on the business location, not the device locale, to display prices in the correct currency even when the device is set to a different region.

---
_Source: WWDC20 Session 10140 page (abstract, chapter summaries, code samples, and resource links)._
