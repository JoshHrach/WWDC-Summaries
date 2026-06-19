# What's New in MapKit and MapKit JS
**WWDC19 · Session 236** · [Watch](https://developer.apple.com/videos/play/wwdc2019/236/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15, tvOS 13; MapKit JS (web)

## Overview
This session presents a substantial wave of new MapKit and MapKit JS features built on top of Apple's completely rebuilt base map (covering 4 million+ miles of road with far greater detail for parks, roads, buildings, and addresses). Six major feature areas are covered: a new server-side Maps Web Snapshots service, Dark Mode map support on iOS/tvOS/macOS/web, point-of-interest filtering for map views and search, GeoJSON native decoding, multi-geometry overlay classes, and new camera boundary and zoom-range controls.

The session uses a WWDC companion app with three features (accommodation finder, After Hours restaurant search, and a concert event map) to demonstrate each API in a realistic context.

## Key Topics

### Rebuilt Apple Maps Base Map
- Rebuilt from scratch with custom cars and planes; more detail for roads, parks, water, buildings, baseball fields, running tracks, walking paths, swimming pools.
- Improved address detail means better search and directions accuracy.
- Improvements automatically available through MapKit and MapKit JS as regions are rolled out (full US by end of 2019, more countries in 2020).

### Maps Web Snapshots **[NEW]**
- New server-side service to generate static map images via URL — usable in emails, web pages, URL previews, anywhere a URL to an image is accepted.
- URL parameters control center coordinate, size, map type, color scheme (light/dark), and more.
- Requires a MapKit JS API key signature for authentication.
- 25,000 snapshots/day during beta.
- URL generation tool available on the MapKit JS developer page.

### Dark Mode Map Support **[NEW]**
- `MKMapView` automatically adapts to the `userInterfaceStyle` of its view hierarchy on iOS 13, tvOS (this was already supported on tvOS), and macOS.
- `MKMapSnapshotter.Options`: set `traitCollection` using the target view's trait collection so snapshots match the current appearance; observe trait collection changes to regenerate.
- MapKit JS: add `colorScheme: "dark"` to the snapshot URL or map options.

### Point of Interest Filtering **[NEW]**
- `MKPointOfInterestFilter` — filter categories shown on the map view and returned by search.
- Create with inclusion or exclusion sets of `MKPointOfInterestCategory` values.
- `MKPointOfInterestFilter.excludingAll` — hide all built-in POI icons.
- Apply to: `MKMapView.pointOfInterestFilter`, `MKLocalSearchCompleter.pointOfInterestFilter`, `MKLocalSearch.Request.pointOfInterestFilter`.
- `MKPointOfInterestCategory` values include: `.hotel`, `.restaurant`, `.cafe`, `.nightlife`, `.parking`, `.museum`, `.hospital`, `.store`, and many more **[NEW]**.
- `MKMapItem.pointOfInterestCategory` **[NEW]** — category of a search result item.
- MapKit JS: `pointOfInterestFilter` property on map and search objects (coming fall 2019).

### Search and Autocompletion Result Types **[NEW]**
- `MKLocalSearchCompleter.resultTypes: MKLocalSearchCompleter.ResultType` **[NEW]** — option set: `.address`, `.pointOfInterest`, `.query`; replaces the old `filterType` property.
- `MKLocalSearch.Request.resultTypes: MKLocalSearch.ResultType` **[NEW]** — option set: `.address`, `.pointOfInterest`.
- MapKit JS: `includeAddresses`, `includePointOfInterest`, `includeQueries` properties on the search object (coming fall 2019).

### Multi-Geometry Overlays **[NEW]**
- `MKMultiPolygon` **[NEW]** — groups multiple `MKPolygon` instances into one overlay object.
- `MKMultiPolyline` **[NEW]** — groups multiple `MKPolyline` instances into one overlay object.
- `MKMultiPolygonRenderer` **[NEW]** — single renderer for all polygons in an `MKMultiPolygon`.
- `MKMultiPolylineRenderer` **[NEW]** — single renderer for all polylines in an `MKMultiPolyline`.
- Reduces renderer object count and enables batched rendering, improving performance for large overlay sets.
- All built-in MapKit renderer objects now render as vector graphics by default (scale cleanly on zoom); opt out with `shouldRasterize = true` on the renderer.

### GeoJSON Decoding **[NEW]**
- `MKGeoJSONDecoder` **[NEW]** — decodes GeoJSON `Data` into MapKit objects.
- `MKGeoJSONDecoder.decode(_:)` returns `[MKGeoJSONObject]` — either `MKGeoJSONFeature` instances or MapKit geometry directly.
- `MKGeoJSONFeature` **[NEW]** — holds `identifier`, `geometry: [MKShape & MKGeoJSONObject]`, and `properties: Data?`.
- GeoJSON geometry types map to: `MKPointAnnotation`, `MKPolyline`, `MKPolygon`, `MKMultiPolyline`, `MKMultiPolygon` — complete coverage of the GeoJSON spec.
- `properties` exposed as raw `Data`; use `JSONDecoder` or `JSONSerialization` to parse.
- MapKit JS: already supported via `mapkit.importGeoJSON(urlOrObject)` returning an `ItemCollection`.
- Related: Indoor Mapping Data Format (IMDF), built on GeoJSON, covered in Session 241.

### Camera Boundary and Zoom Range **[NEW]**
- `MKMapCameraBoundary` **[NEW]** — defines a region within which the map view's center coordinate must remain.
  - Init with `MKCoordinateRegion` or `MKMapRect`.
  - Apply to `MKMapView.cameraBoundary`.
  - Map view strictly enforces the boundary; APIs like `setRegion` that would violate it move as close as allowed.
- `MKMapCameraZoomRange` **[NEW]** — constrains minimum and maximum center-coordinate distance (replaces altitude-based reasoning).
  - Init with `minCenterCoordinateDistance`, `maxCenterCoordinateDistance`, or both.
  - Apply to `MKMapView.cameraZoomRange`.
- `MKMapCamera.centerCoordinateDistance` **[NEW]** — preferred replacement for `altitude`; represents distance from center coordinate up to camera; changing pitch now preserves distance (not altitude).
- MapKit JS: `CameraZoomRange` object; `cameraBoundary` and `cameraZoomRange` properties on the map object.

## APIs & Frameworks

### MapKit (iOS/macOS)
- `MKMapView.pointOfInterestFilter: MKPointOfInterestFilter?` **[NEW]**
- `MKMapView.cameraBoundary: MKMapCameraBoundary?` **[NEW]**
- `MKMapView.cameraZoomRange: MKMapCameraZoomRange?` **[NEW]**
- `MKPointOfInterestFilter` **[NEW]** — `init(including:)`, `init(excluding:)`, `.excludingAll`
- `MKPointOfInterestCategory` **[NEW]** — static category constants
- `MKMapItem.pointOfInterestCategory: MKPointOfInterestCategory?` **[NEW]**
- `MKLocalSearchCompleter.pointOfInterestFilter` **[NEW]**
- `MKLocalSearchCompleter.resultTypes: MKLocalSearchCompleter.ResultType` **[NEW]**
- `MKLocalSearch.Request.pointOfInterestFilter` **[NEW]**
- `MKLocalSearch.Request.resultTypes: MKLocalSearch.ResultType` **[NEW]**
- `MKMultiPolygon` **[NEW]**, `MKMultiPolyline` **[NEW]**
- `MKMultiPolygonRenderer` **[NEW]**, `MKMultiPolylineRenderer` **[NEW]**
- `MKGeoJSONDecoder` **[NEW]**, `MKGeoJSONFeature` **[NEW]**, `MKGeoJSONObject` protocol **[NEW]**
- `MKMapCamera.centerCoordinateDistance` **[NEW]**
- `MKMapCameraBoundary` **[NEW]** — `init(coordinateRegion:)`, `init(mapRect:)`
- `MKMapCameraZoomRange` **[NEW]** — `init(minCenterCoordinateDistance:)`, `init(maxCenterCoordinateDistance:)`, `init(minCenterCoordinateDistance:maxCenterCoordinateDistance:)`
- `MKMapSnapshotter.Options.traitCollection` **[NEW]**
- `MKOverlayRenderer.shouldRasterize` — opt out of vector rendering

### MapKit JS (web)
- `pointOfInterestFilter` on map and search (fall 2019)
- `includeAddresses`, `includePointOfInterest`, `includeQueries` on search (fall 2019)
- `mapkit.CameraZoomRange` **[NEW]**, `cameraBoundary`, `cameraZoomRange` on map
- `mapkit.importGeoJSON(urlOrObject)` — existing GeoJSON support
- Maps Web Snapshots service **[NEW]** — URL-based static map images

## Code Highlights

Point-of-interest filtering for search and autocompletion:
```swift
let filter = MKPointOfInterestFilter(including: [.restaurant, .nightlife])

let completer = MKLocalSearchCompleter()
completer.pointOfInterestFilter = filter
completer.resultTypes = .pointOfInterest

// When performing a search:
let request = MKLocalSearch.Request(completion: selectedCompletion)
request.pointOfInterestFilter = filter
request.resultTypes = .pointOfInterest
let search = MKLocalSearch(request: request)
```

GeoJSON decoding with multi-polygon overlays:
```swift
let decoder = MKGeoJSONDecoder()
let objects = try decoder.decode(jsonData)
for object in objects {
    guard let feature = object as? MKGeoJSONFeature else { continue }
    for geometry in feature.geometry {
        if let multiPolygon = geometry as? MKMultiPolygon {
            mapView.addOverlay(multiPolygon)
        }
    }
}

// In delegate:
func mapView(_ mapView: MKMapView, rendererFor overlay: MKOverlay) -> MKOverlayRenderer {
    if let multiPolygon = overlay as? MKMultiPolygon {
        let renderer = MKMultiPolygonRenderer(multiPolygon: multiPolygon)
        renderer.fillColor = .systemBlue.withAlphaComponent(0.3)
        renderer.strokeColor = .systemBlue
        return renderer
    }
    return MKOverlayRenderer(overlay: overlay)
}
```

Camera boundary and zoom range:
```swift
let region = MKCoordinateRegion(center: eventCenter,
                                latitudinalMeters: 100,
                                longitudinalMeters: 80)
mapView.cameraBoundary = MKMapCameraBoundary(coordinateRegion: region)
mapView.cameraZoomRange = MKMapCameraZoomRange(minCenterCoordinateDistance: 50,
                                               maxCenterCoordinateDistance: 500)
```

## Takeaways
- Point-of-interest filtering and result-type filtering together bring search and autocompletion results from generic to highly relevant with about five lines of code.
- `MKGeoJSONDecoder` provides zero-boilerplate decoding of GeoJSON into ready-to-use MapKit annotations and overlays, including the new `MKMultiPolygon`/`MKMultiPolyline` types.
- Camera boundary and zoom range let you create tightly scoped map experiences (e.g., event maps, indoor venues) where users cannot pan or zoom away from relevant content.
- Maps Web Snapshots extends Apple Maps to any environment that can consume image URLs, unlocking use cases like email maps, web pages, and URL previews without MapKit.

---
_Source: WWDC19 Session 236 page (abstract, chapter summaries, code samples, and resource links)._
