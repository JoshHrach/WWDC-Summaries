# Design for Location Privacy
**WWDC20 · Session 10162** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10162/)

_Platforms:_ iOS 14, iPadOS 14

## Overview
This session, presented by a designer from the Apple Maps team, explains how to adapt app UIs to the new approximate location capability introduced in iOS 14. In iOS 14, users can choose to share only an approximate location (a ~10 km radius area) rather than precise GPS coordinates, and this choice applies per app. The session uses the Maps app redesign as the primary case study, walking through the specific design decisions made to support approximate location without degrading the core experience.

The session is grounded in three guiding design principles — prioritize user control, build trust through transparency, and offer proportional value — and provides concrete, actionable guidance for applying each principle in any app that uses location.

## Key Topics

**The Scale of the Change**
Location data is a foundational input for many app categories — weather, payments, photos, navigation, recommendations. In Maps specifically, the precise blue dot is a primary organizational element used for search relevance, arrival time calculations, distance displays, and navigation. Supporting approximate location required rethinking each of these touchpoints.

**Principle 1: Prioritize User Control**
Design the app to function meaningfully at approximate precision. Remove or hide nonessential UI elements that require precise data; don't surface broken or empty states. Apple Maps removed calculated distances and ETAs from Favorites and search results when running with approximate location, since those cannot be computed without a precise point. The goal is to let users do as much as possible, not to create an app that degrades dramatically without precise data.

**Approximate Location Annotation Design**
The classic pulsing blue dot represents precise location. For approximate location, Maps introduced a shaded circular area overlaid on the map. The circle shares the visual language of the blue dot (blue color family) and adapts to interaction: as the user zooms in, the overlay fades to maintain cartographic legibility. This prevents the annotation from blocking map content at high zoom levels.

**Principle 2: Build Trust Through Transparency**
Use clear, accurate language at decision points (e.g., first app launch) to explain what data is used and what value it provides. After a user has made a location choice, remind them of it persistently but non-intrusively. Maps added a status bar and control at the top of the map showing the current location mode. On first launch it appears large and prominent; after interaction it reduces in size but remains accessible. Users should always be able to see what data they are sharing and change it easily.

**Principle 3: Offer Proportional Value**
Request precise location only when a specific feature requires it, only in response to user-initiated action, and as close to the use of that data as possible. Example from Maps: turn-by-turn navigation requires precise location. Rather than requesting precision upfront or blocking the directions flow, Maps allows users sharing approximate location to see all directions UI normally. When they tap "My Location" as their starting point — the specific moment they indicate intent to navigate from their current position — Maps requests one-time access to precise location. The request is tightly coupled to the action and the value is immediately obvious.

**Practical Implementation Checklist**
- Do not require precise data as a prerequisite to core app functionality
- Remove nonessential UI that requires precise data when user is in approximate mode
- Write clear, accurate explanations at permission prompts — avoid dark patterns
- Display a persistent but non-intrusive indicator of what location mode is active
- Make it easy for users to change their location preference at any time
- Scope precise location requests to specific features; request them at the moment they are needed, not at app launch

## APIs & Frameworks

### Core Location (new in iOS 14)
- `CLLocationManager.requestWhenInUseAuthorization()` — requests standard location authorization
- `CLLocationManager.requestTemporaryFullAccuracyAuthorization(withPurposeKey:)` **[NEW]** — requests one-time precise location access for a specific feature; key maps to an entry in `NSLocationTemporaryUsageDescriptionDictionary` in Info.plist
- `CLAccuracyAuthorization` **[NEW]** — enum: `.fullAccuracy` or `.reducedAccuracy`; read from `CLLocationManager.accuracyAuthorization`
- `CLLocationManagerDelegate.locationManagerDidChangeAuthorization(_:)` **[NEW]** — replaces `locationManager(_:didChangeAuthorization:)`; fires when either authorization status or accuracy authorization changes
- `NSLocationTemporaryUsageDescriptionDictionary` **[NEW]** — Info.plist key containing a dictionary of purpose-key → user-facing string; used by `requestTemporaryFullAccuracyAuthorization`
- `CLLocation.horizontalAccuracy` — approximately 1–5 km for reduced accuracy; use to detect which mode is active

### MapKit
- `MKUserTrackingButton` / `MKUserLocation` — annotates user location on a map; automatically uses the new approximate-location visual when `accuracyAuthorization == .reducedAccuracy`
- `MKCircle` — use to visually represent an approximate-location region in custom map UIs

### Privacy Settings
- Settings → Privacy → Location Services → [App] — per-app location precision toggle (new in iOS 14)
- App-level `NSLocationAlwaysAndWhenInUseUsageDescription`, `NSLocationWhenInUseUsageDescription` — required plist keys for location usage

## Code Highlights

Checking the accuracy authorization level:
```swift
let manager = CLLocationManager()

func locationManagerDidChangeAuthorization(_ manager: CLLocationManager) {
    switch manager.accuracyAuthorization {
    case .fullAccuracy:
        // Precise location available — show full UI
        enablePreciseFeatures()
    case .reducedAccuracy:
        // Approximate only — hide precise-dependent UI
        disablePreciseFeatures()
    @unknown default:
        break
    }
}
```

Requesting temporary precise access for a specific feature (requires Info.plist entry):
```swift
// Info.plist: NSLocationTemporaryUsageDescriptionDictionary
//   -> "NavigationFeature" -> "Turn-by-turn navigation requires your precise location."
manager.requestTemporaryFullAccuracyAuthorization(withPurposeKey: "NavigationFeature") { error in
    if manager.accuracyAuthorization == .fullAccuracy {
        startNavigation()
    }
}
```

## Takeaways
- Design your app to work meaningfully with approximate location as the default; users who choose reduced accuracy are making a deliberate privacy choice and the app should respect it gracefully, not punish them with broken states.
- Use `requestTemporaryFullAccuracyAuthorization(withPurposeKey:)` for features that genuinely require precise coordinates; invoke it at the moment the user initiates that specific feature, not at app launch — this makes the value proposition immediately clear.
- Always display a persistent, unobtrusive indicator of which location mode is active, and make it easy to change — transparency about data usage builds the trust that keeps users from revoking location access entirely.
- Remove or hide UI elements that display calculated values (ETAs, distances) when running in approximate mode; showing empty or broken values erodes trust more than simply removing the element.

---
_Source: WWDC20 Session 10162 page (abstract and full transcript)._
