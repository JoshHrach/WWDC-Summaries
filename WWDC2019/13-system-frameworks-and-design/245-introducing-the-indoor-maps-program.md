# Introducing the Indoor Maps Program
**WWDC19 · Session 245** · [Watch](https://developer.apple.com/videos/play/wwdc2019/245/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15

## Overview
The Apple Indoor Maps Program allows organizations that own large public or private spaces — airports, shopping centers, hospitals, stadiums, office buildings — to deliver precise indoor location experiences. Participants create a standards-based Indoor Mapping Data Format (IMDF) map, validate it through Apple's tooling, enable Apple indoor positioning via Wi-Fi fingerprint surveys, and then surface the results in their own apps and websites using MapKit or MapKit JS.

The session walks through the full enablement workflow: enrolling an organization at register.apple.com/indoor, claiming building locations, uploading and validating IMDF archives (with the browser-based IMDF Sandbox), conducting Wi-Fi fingerprint surveys using the indoor survey app, and integrating the resulting indoor map and blue-dot location into an app. Buildings eligible for public display (airports, malls) may also request inclusion in Apple Maps itself.

Apple indoor positioning uses passive Wi-Fi fingerprinting entirely on-device. Once the fingerprint database is downloaded, no ongoing network connection is required. Core Location raises the user's `desiredAccuracy` to `kCLLocationAccuracyBest`, which triggers indoor positioning transparently.

## Key Topics
- **Indoor Mapping Data Format (IMDF)** — Apple's GeoJSON-based open specification for modeling indoor spaces (venue, building, footprint, level, unit, opening, section, kiosk, fixture, amenity, anchor, occupant layers)
- **IMDF Sandbox** — browser-based tool for visualizing, searching, validating, and lightly editing IMDF archives; available to anyone with an Apple ID
- **Organization enrollment and team management** — register.apple.com/indoor; role-based access control per venue; partner/system integrator invitation model
- **IMDF validation** — electronic validation on upload plus manual georeferencing review by Apple; errors block acceptance, warnings require business review
- **Apple indoor positioning via Wi-Fi fingerprinting** — passive scanning, on-device pattern matching, no network required post-download; survey app used to collect training data
- **Survey best practices** — same iPhone model and iOS version across surveyors, pins every 5–8 m, no case on device, smooth walking motion
- **MapKit / MapKit JS integration** — decode and render GeoJSON/IMDF overlays and annotations; style elements to match app brand

## APIs & Frameworks
- **MapKit** **[updated for indoor]**
  - GeoJSON decoding (`MKGeoJSONDecoder`) for rendering IMDF overlays **[NEW]**
  - Indoor map overlay support
  - `MKMapView` — displaying indoor levels and annotations
- **MapKit JS** **[updated for indoor]**
  - `mapkit.GeoJSON` — load and render IMDF GeoJSON files in web contexts
  - Indoor map overlay support
- **Core Location**
  - `CLLocationManager` — `desiredAccuracy = kCLLocationAccuracyBest` triggers indoor positioning
  - `CLFloor` — floor-level information attached to indoor `CLLocation` **[available where indoor positioning is active]**
  - Indoor blue-dot location updates
- **IMDF (Indoor Mapping Data Format)** — GeoJSON feature types: `venue`, `building`, `footprint`, `level`, `unit`, `opening`, `section`, `geofence`, `kiosk`, `fixture`, `amenity`, `anchor`, `occupant`, `address`, `manifest`
- **Apple Business Register** — organization enrollment and venue management portal (register.apple.com/indoor)
- **IMDF Sandbox** — browser-based validation and editing tool

## Code Highlights
No SDK code was demonstrated in this session. The integration pattern is:

```swift
// Enable indoor location in your app
let locationManager = CLLocationManager()
locationManager.desiredAccuracy = kCLLocationAccuracyBest
locationManager.requestWhenInUseAuthorization()
locationManager.startUpdatingLocation()

// CLLocation includes floor information indoors
func locationManager(_ manager: CLLocationManager,
                     didUpdateLocations locations: [CLLocation]) {
    if let floor = locations.last?.floor {
        print("Floor level: \(floor.level)")
    }
}
```

```javascript
// MapKit JS — load IMDF GeoJSON for a venue
map.showItems(mapkit.importGeoJSON("venue.geojson", {
    styleForFeature: function(feature) { /* custom styling */ }
}));
```

## Takeaways
- IMDF is Apple's open GeoJSON standard for indoor spaces and is now supported by major GIS/BIM platforms (Safe Software, Autodesk, Esri).
- Any developer can display IMDF in their app via MapKit/MapKit JS; joining the Indoor Maps Program is required only for enabling Apple indoor positioning or Apple Maps inclusion.
- Apple indoor positioning is entirely on-device after fingerprint download — passive Wi-Fi, no beacons, no network required while indoors.
- Survey quality (consistent hardware, correct walking pattern, 5–8 m pin spacing) directly determines indoor positioning accuracy (target: 3–5 m).

---
_Source: WWDC19 Session 245 page (abstract, chapter summaries, code samples, and resource links)._
