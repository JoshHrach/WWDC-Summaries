# What's New in Location
**WWDC20 · Session 10660** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10660/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, watchOS 7

## Overview
iOS 14 introduces a second dimension to Core Location authorization: in addition to choosing _when_ an app can access location (never, while using, always), users can now also choose _how precisely_. A new "Precise" toggle appears in the initial authorization prompt and in Settings, allowing users to grant only approximate location rather than exact location. Approximate locations are quantized to regions of roughly 5 km radius in populated areas (up to ~10 km in rural areas) and update about four times per hour. Apps cannot opt out of this feature; they must adapt.

The session covers the full API surface for the new accuracy authorization model—a new `CLAccuracyAuthorization` enum, a new `locationManagerDidChangeAuthorization` delegate method that fires on either authorization status or accuracy changes, and a new `requestTemporaryFullAccuracyAuthorization(withPurposeKey:)` method that allows apps to request a one-time upgrade to exact location for features that require it. The session also addresses how background location (significant change, visit monitoring, region monitoring) interacts with reduced accuracy—region monitoring does not fire for apps without full accuracy and cannot use temporary upgrades—and introduces Core Location considerations for App Clips (no always authorization, "While Using Until Tomorrow" expiration) and WidgetKit (no authorization prompts; inherits parent app's authorization).

## Key Topics
- **`CLAccuracyAuthorization`** — new enum: `.fullAccuracy` (default) / `.reducedAccuracy` **[NEW]**
- **`CLLocationManager.accuracyAuthorization`** — new instance property **[NEW]**
- **`locationManagerDidChangeAuthorization(_:)`** — replaces deprecated `didChangeAuthorization(status:)`; fires on both authorization status and accuracy changes **[NEW]**
- **`requestTemporaryFullAccuracyAuthorization(withPurposeKey:completion:)`** — prompts user to temporarily upgrade to full accuracy for a specific feature **[NEW]**
- **`NSLocationTemporaryUsageDescriptionDictionary`** — Info.plist dictionary mapping purpose keys to localized strings **[NEW]**
- **`CLLocationAccuracy.reduced`** — new constant; set `desiredAccuracy` to this value to explicitly request reduced accuracy locations **[NEW]**
- **`NSLocationDefaultAccuracyReduced`** — Info.plist key; opts app into reduced accuracy by default (hides Precise:On toggle in prompt) **[NEW]**
- **`CLLocationManager.authorizationStatus`** — deprecated class method replaced by instance property **[UPDATED]**
- **App Clips** — cannot receive always authorization; receive "While Using Until Tomorrow" (authorization expires next morning) **[NEW behavior]**
- **Widgets (WidgetKit)** — include `NSWidgetWantsLocation` in widget's Info.plist; cannot show authorization prompts; inherits parent app's authorization **[NEW]**
- **Region monitoring** — requires full accuracy; not satisfied by temporary upgrade
- **Reduced accuracy locations** — quantized (not noise-added); ~5 km radius typical; updated ~4 times/hour; delivered as `CLLocation` with large `horizontalAccuracy`

## APIs & Frameworks

**Core Location**
- `CLAccuracyAuthorization` enum **[NEW]** — `.fullAccuracy`, `.reducedAccuracy`
- `CLLocationManager.accuracyAuthorization: CLAccuracyAuthorization` — current accuracy authorization **[NEW]**
- `CLLocationManager.authorizationStatus: CLAuthorizationStatus` — new instance property (replaces `CLLocationManager.authorizationStatus()` class method) **[NEW; class method deprecated]**
- `CLLocationManagerDelegate.locationManagerDidChangeAuthorization(_ manager: CLLocationManager)` **[NEW]** — called when either `authorizationStatus` or `accuracyAuthorization` changes; replaces deprecated `locationManager(_:didChangeAuthorization:)`
- `CLLocationManager.requestTemporaryFullAccuracyAuthorization(withPurposeKey: String, completion: ((Error?) -> Void)?)` **[NEW]** — presents a one-time prompt to upgrade to full accuracy; requires matching key in `NSLocationTemporaryUsageDescriptionDictionary`
- `CLLocationAccuracy.reduced` — new constant; assign to `desiredAccuracy` to request reduced accuracy **[NEW]**

**Info.plist Keys**
- `NSLocationTemporaryUsageDescriptionDictionary` — dictionary of `String → String` purpose key to localized explanation; required for `requestTemporaryFullAccuracyAuthorization` **[NEW]**
- `NSLocationDefaultAccuracyReduced` — Boolean; set `true` to default new authorization requests to reduced accuracy and hide Precise:On in the prompt **[NEW]**
- `NSWidgetWantsLocation` — in widget extension's Info.plist; required for widgets that use location **[NEW]**

**WatchConnectivity (improvement)**
- Temporary authorization status and accuracy upgrade state now sync between companion iPhone and Apple Watch apps **[NEW]**

## Code Highlights

Implement the new delegate method to handle both authorization status and accuracy changes:
```swift
func locationManagerDidChangeAuthorization(_ manager: CLLocationManager) {
    switch manager.authorizationStatus {
    case .authorizedWhenInUse, .authorizedAlways:
        switch manager.accuracyAuthorization {
        case .fullAccuracy:
            startFullAccuracyFeatures()
        case .reducedAccuracy:
            startReducedAccuracyFeatures()
            // Don't ask for temporary upgrade here; wait until the user invokes a feature that needs it
        @unknown default:
            break
        }
    case .denied, .restricted:
        showLocationDeniedUI()
    case .notDetermined:
        break
    @unknown default:
        break
    }
}
```

Request a temporary full accuracy upgrade when navigating:
```swift
// Info.plist must contain:
// NSLocationTemporaryUsageDescriptionDictionary → WantsToNavigate → "Navigation requires your exact location."

func startNavigation() {
    guard locationManager.accuracyAuthorization == .fullAccuracy else {
        locationManager.requestTemporaryFullAccuracyAuthorization(withPurposeKey: "WantsToNavigate") { [weak self] error in
            guard error == nil else { return }
            self?.startNavigation()
        }
        return
    }
    // Proceed with full accuracy navigation
    locationManager.startUpdatingLocation()
}
```

Opt a location manager into reduced accuracy explicitly:
```swift
locationManager.desiredAccuracy = kCLLocationAccuracyReduced
locationManager.startUpdatingLocation()
// Delivers reduced accuracy locations without asking for more than needed
```

## Takeaways
- Apps must handle `CLAccuracyAuthorization.reducedAccuracy` gracefully; reduced accuracy is a user right, not an edge case—design the app's primary flow to work with approximate location (large `CLLocation.horizontalAccuracy`, ~5 km radius, 4 updates/hour) and reserve `requestTemporaryFullAccuracyAuthorization` only for features that genuinely require exact position.
- Replace the deprecated `locationManager(_:didChangeAuthorization:)` delegate method and `CLLocationManager.authorizationStatus()` class method with the new `locationManagerDidChangeAuthorization` instance delegate and `CLLocationManager.authorizationStatus` instance property, which fire on both status and accuracy changes.
- Region monitoring (geofences) requires full accuracy permanently granted; a temporary accuracy upgrade is insufficient—prompt users to go to Settings or inform them that the geofence feature is unavailable at reduced accuracy.
- App Clips receive "While Using Until Tomorrow" location authorization (not always); design App Clip location features to not rely on always authorization, and handle the expiration gracefully when the clip is re-invoked after midnight.

---
_Source: WWDC20 Session 10660 page (transcript)._
