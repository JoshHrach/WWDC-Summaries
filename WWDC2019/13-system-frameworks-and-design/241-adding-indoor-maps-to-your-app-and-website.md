# Adding Indoor Maps to your App and Website
**WWDC19 · Session 241** · [Watch](https://developer.apple.com/videos/play/wwdc2019/241/)

_Platforms:_ iOS 13, iPadOS 13

## Overview
The Indoor Maps Program allows organizations with large public or private spaces to deliver precise indoor location experiences inside their own apps and websites. This session covers the full workflow: adopting the Indoor Mapping Data Format (IMDF) to describe a venue, using Apple's developer tools (IMDF Sandbox, Indoor Survey App) to validate and enable positioning, and then rendering the resulting indoor maps using MapKit on iOS and MapKit JS on the web.

The session walks through building a complete "Dinoseum" museum app — including floor-plan overlays, level picker, styled annotations, custom icons, and live indoor blue-dot positioning — and then replicates the same experience in a browser using MapKit JS's `importGeoJSON` API. Both platforms share the IMDF GeoJSON data format, making data reuse straightforward.

The session also provides best-practice guidance on styling, zoom-level-based detail management, icon design, and surrounding context rendering.

## Key Topics

### Indoor Mapping Data Format (IMDF)
- An open 2D GeoJSON-based specification for modeling indoor spaces (registered at register.apple.com).
- An IMDF archive is a set of JSON files: one `manifest.json` plus separate GeoJSON files per feature type.
- Key IMDF feature types: `Venue`, `Building`, `Footprint`, `Level`, `Unit`, `Opening`, `Kiosk`, `Occupant`, `Anchor`, `Amenity`, `Section`.
- Each feature carries a UUID identifier, a `feature_type` property, and optional geometry (2D GeoJSON geometry).
- Level ordinals: ground = 0, positive = above ground, negative = subterranean.
- `Occupant` features carry business metadata (name, phone, hours) but no geometry; they link to an `Anchor` for their display point.

### Indoor Maps Program Workflow
1. Sign up at register.apple.com/indoor and have Apple review the application.
2. Work with a geospatial tool provider to produce an IMDF archive.
3. Validate with the **IMDF Sandbox** tool (visualize, detect and fix errors).
4. Submit for Apple's extended validation.
5. Survey the venue with the **Indoor Survey App** (collects Wi-Fi radiofrequency fingerprints; uploads to Apple servers for processing).
6. Test accuracy and, optionally, enable venue display inside Apple Maps.

### MapKit (iOS) — Displaying Indoor Maps
- Three-step process: (1) Model feature types as Swift classes, (2) decode GeoJSON with `MKGeoJSONDecoder`, (3) render with MapKit overlays and annotations.
- `MKGeoJSONDecoder` **[NEW]** decodes any GeoJSON data into `MKGeoJSONFeature` objects.
- `MKGeoJSONFeature` exposes `geometry: [MKShape & MKGeoJSONObject]`, `identifier: String?`, and `properties: Data?`.
- New renderers: `MKMultiPolygonRenderer` **[NEW]** and `MKMultiPolylineRenderer` **[NEW]** for efficient multi-geometry rendering.
- `CLLocation.floor` provides indoor floor information when the venue has been surveyed.
- `MKMapView.showsUserLocation` + `CLLocationManager` for indoor blue-dot positioning.
- Level picker driven by ordinals; `showFeaturesForOrdinal(_:)` pattern removes and re-adds overlays and annotations.

### MapKit JS — Displaying Indoor Maps on the Web
- `mapkit.importGeoJSON(data, delegate)` **[NEW]** converts GeoJSON features to MapKit JS items.
- Delegate callbacks: `geoJSONDidComplete(result)` (array of created items), `geoJSONDidError(error)`.
- `styleForOverlay(overlay)` delegate method: customize `Style` (fillColor, strokeColor, fillOpacity, lineWidth, etc.) per overlay.
- `itemForPoint(coordinate, geoJSON, defaultAnnotation)` delegate method: replace default `MarkerAnnotation` with custom `ImageAnnotation` or dot annotations.
- `mapkit.ImageAnnotation` — image-based annotation; set `url` property for custom icons.
- `mapkit.Annotation.DisplayPriority` — controls which annotations survive when display area is constrained.
- `mapkit.MapView.cameraZoomRange` **[NEW]** — restricts the user's ability to zoom beyond specified limits.
- `mapkit.MapView.showItems(items)` vs `addItems(items)` — showItems also re-centers the map.

### Best Practices
- Style indoor maps to match your app's visual identity, not Apple Maps.
- Use distinct colors per unit category (e.g., elevators, walkways, food court).
- Use clear, recognizable icons for amenities.
- Progressively reveal detail as zoom level increases; avoid clutter at small scales.
- Include surrounding streets and context to help users orient.
- Enable indoor user location via Indoor Survey App participation for the best UX.

## APIs & Frameworks

**MapKit (iOS/iPadOS)**
- `MKGeoJSONDecoder` **[NEW]** — `func decode(_ data: Data) throws -> [MKGeoJSONObject]`
- `MKGeoJSONFeature` **[NEW]** — `geometry`, `identifier`, `properties`
- `MKGeoJSONObject` protocol **[NEW]**
- `MKMultiPolygon` **[NEW]** — multi-polygon geometry shape
- `MKMultiPolyline` **[NEW]** — multi-polyline geometry shape
- `MKMultiPolygonRenderer` **[NEW]** — renderer for MKMultiPolygon
- `MKMultiPolylineRenderer` **[NEW]** — renderer for MKMultiPolyline
- `MKMapView.addOverlays(_:)` / `removeOverlays(_:)`
- `MKMapView.addAnnotations(_:)` / `removeAnnotations(_:)`
- `MKMapView.showsUserLocation: Bool`
- `MKMapViewDelegate.mapView(_:rendererFor:)` — return overlay renderer
- `MKMapViewDelegate.mapView(_:viewFor:)` — return annotation view
- `MKMapViewDelegate.mapView(_:didUpdate:)` — receive indoor location updates
- `CLLocation.floor: CLFloor?` — provides indoor floor number when surveyed
- `CLLocationManager.requestWhenInUseAuthorization()`

**MapKit JS**
- `mapkit.importGeoJSON(data, delegate)` **[NEW]**
- `mapkit.GeoJSONDelegate` — `geoJSONDidComplete`, `geoJSONDidError`, `styleForOverlay`, `itemForPoint`
- `mapkit.MapView.showItems(items)` / `addItems(items)` / `removeItems(items)`
- `mapkit.Style` — `fillColor`, `strokeColor`, `fillOpacity`, `lineWidth`, `lineJoin`, `lineCap`
- `mapkit.MarkerAnnotation` — default point annotation
- `mapkit.ImageAnnotation` **[NEW]** — image-based point annotation with `url` property
- `mapkit.Annotation.DisplayPriority` — `.low`, `.high`, `.required`
- `mapkit.MapView.cameraZoomRange` **[NEW]** — `minCameraDistance`, `maxCameraDistance`

## Code Highlights

```swift
// iOS: Decode IMDF GeoJSON files
let decoder = MKGeoJSONDecoder()
let geoJSONObjects = try decoder.decode(data)
let features = geoJSONObjects.compactMap { $0 as? MKGeoJSONFeature }

// iOS: Indoor user location floor update
func mapView(_ mapView: MKMapView, didUpdate userLocation: MKUserLocation) {
    guard let location = userLocation.location,
          let floor = location.floor else { return }
    showFeaturesForOrdinal(floor.level)
}
```

```javascript
// MapKit JS: importGeoJSON with custom styling
map.importGeoJSON(imdfData, {
    styleForOverlay: (overlay) => {
        const style = overlay.style
        if (overlay.data.feature_type === "unit") {
            style.fillColor = unitStyles[overlay.data.properties.category]?.fillColor ?? "#f0f0f0"
        }
        return style
    },
    itemForPoint: (coordinate, geoJSON, defaultAnnotation) => {
        const iconUrl = iconUrls[geoJSON.properties.category]
        return iconUrl
            ? new mapkit.ImageAnnotation(coordinate, { url: { 1: iconUrl } })
            : new DotAnnotation(coordinate)
    },
    geoJSONDidComplete: (result) => map.showItems(result)
})

// Restrict zoom range
map.cameraZoomRange = new mapkit.CameraZoomRange(200, 5000)
```

## Takeaways
- IMDF is a standard GeoJSON-based format; standard GeoJSON tooling applies, making it easy to load, filter, and render on both iOS (MKGeoJSONDecoder) and the web (importGeoJSON).
- The new `MKGeoJSONDecoder` eliminates custom JSON parsing — one call converts IMDF data into typed MapKit geometry objects.
- Participating in the Indoor Maps Program and surveying a venue is the only way to enable the indoor blue-dot on iOS; it requires Wi-Fi infrastructure and Apple approval.
- Feature-level styling based on category (unit type, amenity type) is the highest-leverage way to make indoor maps immediately useful and scannable.

---
_Source: WWDC19 Session 241 page (abstract, chapter summaries, code samples, and resource links)._
