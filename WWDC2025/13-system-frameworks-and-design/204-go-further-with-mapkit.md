# Go further with MapKit

**Session ID:** 204  
**WWDC Year:** 2025  
**Folder:** `13-system-frameworks-and-design`  
**URL:** https://developer.apple.com/videos/play/wwdc2025/204/

---

## Overview

This session covers the major MapKit and GeoToolbox additions in iOS 26, iPadOS 26, and macOS 26. The headline feature is the new GeoToolbox framework (and its MapKit integration), which provides high-quality reverse and forward geocoding using the new `PlaceDescriptor` type, replacing the older `CLGeocoder` for most use cases. The session also covers cycling directions in `MKDirections`, LookAround API updates for JavaScript/web embedding, and enhancements to `MKMapView` annotation clustering and map style customization.

---

## Key Topics

- GeoToolbox framework: `PlaceDescriptor`, reverse geocoding, forward geocoding
- Forward geocoding: searching by structured address vs. free-text query
- Reverse geocoding: converting coordinates to rich `PlaceDescriptor` results
- Cycling directions in `MKDirections.Request`
- LookAround JavaScript API for web embedding
- `MKMapView` annotation and marker clustering improvements
- Map style tokens and custom map styling
- Migrating from `CLGeocoder` to GeoToolbox

---

## APIs & Frameworks

- **GeoToolbox** framework (`import GeoToolbox`) – **[NEW]** (iOS 26, iPadOS 26, macOS 26) Higher-level geocoding and place description framework; works standalone or alongside MapKit.
- **`PlaceDescriptor`** – **[NEW]** Rich place representation returned by GeoToolbox geocoding operations. Properties: `name: String?`, `coordinate: CLLocationCoordinate2D`, `postalAddress: CNPostalAddress?`, `categories: [PlaceDescriptor.Category]`, `timeZone: TimeZone?`, `region: CLCircularRegion?`.
- **`GeoToolbox.reverseGeocode(coordinate:)`** – **[NEW]** Async function returning `[PlaceDescriptor]` for a given coordinate. More detailed than `CLGeocoder.reverseGeocodeLocation`.
- **`GeoToolbox.geocode(address:)`** – **[NEW]** Async forward geocoding from a structured `CNPostalAddress` or free-text string, returning `[PlaceDescriptor]`.
- **`PlaceDescriptor.Category`** – **[NEW]** Enum of semantic place categories: `.restaurant`, `.park`, `.transit`, `.shopping`, `.landmark`, etc.
- **`MKDirections.Request.transportType`** – Existing property; now includes `.cycling` **[NEW]** as a valid transport type on supported regions. Returns turn-by-turn cycling routes with elevation data.
- **`MKRoute.steps` / `MKRouteStep`** – Cycling routes include step-level elevation gain/loss in `MKRouteStep.distance` context; new `MKRouteStep.elevationChange: Double` **[NEW]** property.
- **`MKLookAroundSceneRequest`** – Existing API; updated to support LookAround in additional countries and cities in iOS 26.
- **MapKit JS LookAround API** – **[NEW]** JavaScript API for embedding LookAround panoramas in web pages via MapKit JS; mirrors the native `MKLookAroundViewController`.
- **`MKAnnotationView` clustering** – **[NEW]** Improved clustering algorithm with `clusteringPriority` property on `MKAnnotationView` to control which annotations prefer to remain visible when a cluster forms.
- **Map Style Token** – **[NEW]** Server-side custom map style configuration; associate a style token with an `MKMapView` via `mapView.preferredConfiguration = MKStandardMapConfiguration(styleToken: token)` to apply custom colors, POI visibility, and road label settings without client-side SDK changes.
- **`CLGeocoder`** – Existing API; still functional but GeoToolbox provides richer results and async/await integration without a callback pattern.

---

## Code Highlights

Reverse geocoding with GeoToolbox:
```swift
import GeoToolbox
import CoreLocation

let coordinate = CLLocationCoordinate2D(latitude: 37.3318, longitude: -122.0312)
let places = try await GeoToolbox.reverseGeocode(coordinate: coordinate)

if let place = places.first {
    print(place.name ?? "Unknown")
    print(place.postalAddress?.city ?? "")
    print(place.categories)    // e.g., [.landmark, .park]
}
```

Forward geocoding from free text:
```swift
let results = try await GeoToolbox.geocode(address: "1 Infinite Loop, Cupertino, CA")
for place in results {
    print("\(place.name ?? "?") at \(place.coordinate.latitude), \(place.coordinate.longitude)")
}
```

Requesting cycling directions:
```swift
import MapKit

let request = MKDirections.Request()
request.source = MKMapItem.forCurrentLocation()
request.destination = MKMapItem(placemark: MKPlacemark(coordinate: destination))
request.transportType = .cycling

let directions = MKDirections(request: request)
let response = try await directions.calculate()
for step in response.routes.first?.steps ?? [] {
    print("\(step.instructions) — \(step.distance)m, elevation: \(step.elevationChange)m")
}
```

---

## Takeaways

- GeoToolbox's `PlaceDescriptor` provides richer, more structured geocoding results than `CLGeocoder`, including place categories, time zones, and regional boundaries — in a modern async/await API.
- Forward geocoding in GeoToolbox supports both free-text input and structured `CNPostalAddress`, making it suitable for both search boxes and form-driven address input.
- Cycling directions are now a first-class transport type in `MKDirections`; elevation data per step enables fitness and navigation apps to display grade information.
- LookAround in MapKit JS brings the full native panorama experience to web apps with minimal JavaScript.
- Map Style Tokens enable design teams to configure map appearances server-side without app updates — useful for branded map experiences.
- `PlaceDescriptor.Category` enables apps to filter geocoding results semantically (e.g., only show `.park` results) rather than parsing raw address strings.
