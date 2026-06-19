# What's New in MapKit
**WWDC15 · Session 206** · [Watch](https://developer.apple.com/videos/play/wwdc2015/206/)

_Platforms:_ iOS 9, OS X El Capitan 10.11, watchOS 2

## Overview
This session by Sumit Lonkar and Elisabeth Lindkvist covers the MapKit additions in iOS 9 and OS X El Capitan, organized into three themes: annotation and callout customization (pin tint color, fully custom detail callout views), Transit integration (Transit ETAs, launching Maps in Transit mode), and Flyover — two new map types bringing photo-realistic 3D city models to third-party apps for the first time.

The session also reviews smaller quality-of-life improvements: traffic overlay control, compass and scale visibility toggles, time zone support on `MKMapItem`, Swift compatibility improvements, and WatchKit support for launching Maps from Apple Watch apps. A live demo walks through building a San Francisco city tour app that combines all the new features.

## Key Topics

### Annotation Customization

- **`pinTintColor`** (new property on `MKPinAnnotationView`) **[NEW]**: Accepts any `UIColor` (iOS) or `NSColor` (OS X) — replaces the deprecated `pinColor` property which only offered three fixed colors (red, green, purple). Apps still using `pinColor` should migrate to `pinTintColor`.
- **`detailCalloutAccessoryView`** (new property on `MKAnnotationView`) **[NEW]**: Accepts any `UIView` / `NSView` — replaces the clipped `subtitle` for rich content inside the callout bubble. Supports Auto Layout and right-to-left languages. Takes precedence over `subtitle` when set. Use cases include `UIImageView`, `UITextView`, progress bars, or fully custom layouts, all in a single line of code.

### Map Display Control

- **`showsTraffic`** (new Bool property on `MKMapView`) **[NEW]**: Toggle real-time high-traffic overlay visibility.
- **`showsScale`** (new Bool property on `MKMapView`) **[NEW]**: Show or hide the distance scale in the top-left corner.
- **`showsCompass`** (new Bool property on `MKMapView`) **[NEW]**: Show or hide the compass indicator.
- Complements existing `showsPointsOfInterest` and `showsBuildings` for full display control.

### Transit Integration **[NEW]**

- **`MKDirectionsTransportType.transit`** **[NEW]**: New transport type for `MKDirectionsRequest` — enables querying transit ETAs (expected arrival time, expected departure time). Note: only ETAs are returned, not step-by-step transit directions.
- **`MKLaunchOptionsDirectionsModeKey` = `MKLaunchOptionsDirectionsModeTransit`** **[NEW]**: Launch the Maps app directly into Transit mode with a destination `MKMapItem`. Complements the existing `Driving` and `Walking` launch modes.
- ETA response includes `expectedArrivalDate` and `expectedDepartureDate` — critical because transit frequency varies by time of day and location.

### Flyover Map Types **[NEW]**

- **`MKMapType.satelliteFlyover`** **[NEW]**: Photo-realistic 3D models of cities and landmarks, rendered on a globe. Available in Maps since iOS 6; now exposed to third-party apps.
- **`MKMapType.hybridFlyover`** **[NEW]**: Flyover imagery plus roads, labels, POIs, and borders layered on top.
- Flyover renders on a globe (not a 2D projection) — rectangular region representations are approximations; large regions may have areas hidden behind the globe curvature.
- **Recommended camera setup**: Use `MKMapCamera` (introduced iOS 7) rather than `MKCoordinateRegion` / `MKMapRect` for precise framing in Flyover.
- **New `MKMapCamera` initializer** **[NEW]**: `MKMapCamera(lookingAtCenter:fromDistance:pitch:heading:)` — specifies distance from the center coordinate in meters rather than altitude above sea level. More predictable in Flyover where terrain and buildings vary.
- Annotation behavior in Flyover: `MKAnnotationView` pins are placed on top of Flyover buildings and terrain (same as standard 3D).
- Overlay behavior: `MKOverlay` objects are occluded by Flyover buildings but follow the terrain contours.

### Other Improvements

- **`MKMapItem.timeZone`** (new property) **[NEW]**: Returns the `TimeZone` associated with the map item. Works with `CLGeocoder` (look up time zone for a coordinate) and `MKLocalSearch` (look up time zone for a search result).
- Swift API improvements for better Swift compatibility throughout MapKit.
- WatchKit support: Launch the Maps application on Apple Watch from a Watch extension.
- **`MKMapSnapshotter`** with Flyover: Set `mapType` to `.satelliteFlyover` on `MKMapSnapshotOptions` to generate static Flyover images — useful for callout thumbnail previews or custom imagery without an interactive map view.

## APIs & Frameworks

- `MapKit` framework
- `MKAnnotationView`
  - `pinTintColor: UIColor` (iOS) / `NSColor` (OS X) **[NEW]** — replaces deprecated `pinColor`
  - `detailCalloutAccessoryView: UIView?` **[NEW]** — fully custom callout detail area
- `MKPinAnnotationView`
  - `pinTintColor` **[NEW]**
- `MKMapView`
  - `showsTraffic: Bool` **[NEW]**
  - `showsScale: Bool` **[NEW]**
  - `showsCompass: Bool` **[NEW]**
  - `showsPointsOfInterest: Bool` (existing)
  - `showsBuildings: Bool` (existing)
- `MKMapType`
  - `.satelliteFlyover` **[NEW]**
  - `.hybridFlyover` **[NEW]**
- `MKDirectionsRequest`
  - `transportType: MKDirectionsTransportType`
- `MKDirectionsTransportType`
  - `.transit` **[NEW]**
  - `.automobile`, `.walking` (existing)
- `MKDirections`
  - `calculateETA(completionHandler:)` — returns `MKETAResponse` with `expectedArrivalDate` and `expectedDepartureDate`
- `MKETAResponse` — `expectedArrivalDate`, `expectedDepartureDate`, `distance`, `transportType` **[NEW]**
- `MKMapCamera`
  - `init(lookingAtCenter:fromDistance:pitch:heading:)` **[NEW]** — distance-based Flyover initializer
  - `init(lookingAtCenter:fromEyeCoordinate:eyeAltitude:)` (existing iOS 7)
  - `centerCoordinate`, `heading`, `pitch`, `altitude` properties
- `MKMapItem`
  - `timeZone: TimeZone?` **[NEW]**
  - `openInMaps(launchOptions:)` — with `MKLaunchOptionsDirectionsModeKey: MKLaunchOptionsDirectionsModeTransit` **[NEW]**
- `MKLaunchOptionsDirectionsModeTransit` **[NEW]** — launch Maps in Transit mode
- `MKMapSnapshotter` / `MKMapSnapshotOptions`
  - `mapType: MKMapType` — set to `.satelliteFlyover` for Flyover snapshots **[NEW]**
  - `camera: MKMapCamera` — use new distance-based initializer for Flyover
- `CLGeocoder` — returns `MKPlacemark` with time zone info via `MKMapItem`
- `MKLocalSearch` — search results include `MKMapItem` with `timeZone` **[NEW]**

## Code Highlights

Custom pin color:
```swift
let pinView = MKPinAnnotationView(annotation: annotation, reuseIdentifier: "pin")
pinView.pinTintColor = UIColor(red: 0.2, green: 0.6, blue: 1.0, alpha: 1.0)
```

Custom callout detail view with image:
```swift
func mapView(_ mapView: MKMapView, didSelect view: MKAnnotationView) {
    let imageView = UIImageView(image: UIImage(named: "goldengate"))
    view.detailCalloutAccessoryView = imageView
}
```

Request a Transit ETA:
```swift
let request = MKDirectionsRequest()
request.source = MKMapItem.forCurrentLocation()
request.destination = destinationItem
request.transportType = .transit
let directions = MKDirections(request: request)
directions.calculateETA { response, error in
    guard let response = response else { return }
    print("Arrives: \(response.expectedArrivalDate)")
}
```

Launch Maps in Transit mode:
```swift
var options = [String: AnyObject]()
options[MKLaunchOptionsDirectionsModeKey] = MKLaunchOptionsDirectionsModeTransit
destinationItem.openInMaps(launchOptions: options)
```

Set up Flyover map with new distance-based camera:
```swift
mapView.mapType = .satelliteFlyover
let camera = MKMapCamera(lookingAtCenter: coordinate,
                         fromDistance: 500,
                         pitch: 65,
                         heading: 0)
mapView.setCamera(camera, animated: true)
```

Generate a Flyover snapshot for a callout:
```swift
let options = MKMapSnapshotOptions()
options.mapType = .satelliteFlyover
options.camera = MKMapCamera(lookingAtCenter: coordinate, fromDistance: 300, pitch: 45, heading: 0)
let snapshotter = MKMapSnapshotter(options: options)
snapshotter.start { snapshot, error in
    guard let snapshot = snapshot else { return }
    annotationView.detailCalloutAccessoryView = UIImageView(image: snapshot.image)
}
```

## Takeaways
- `detailCalloutAccessoryView` replaces the old subtitle-only callout with any `UIView`, enabling rich content (images, text views, custom layouts) with a single property assignment.
- Transit ETA queries (`MKDirectionsTransportType.transit`) and the `MKLaunchOptionsDirectionsModeTransit` launch option provide a complete in-app transit workflow without building a transit engine.
- The two new Flyover map types (`.satelliteFlyover`, `.hybridFlyover`) bring photo-realistic 3D city models to third-party apps — use `MKMapCamera` with the new `fromDistance:` initializer for reliable framing since altitude is ambiguous over terrain.
- Deprecate `pinColor` immediately — `pinTintColor` accepts the full range of `UIColor`/`NSColor` values.

---
_Source: WWDC15 Session 206 page (abstract, chapter summaries, code samples, and resource links)._
