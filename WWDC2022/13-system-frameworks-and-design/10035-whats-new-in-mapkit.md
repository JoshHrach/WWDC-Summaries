# What's New in MapKit
**WWDC22 · Session 10035** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10035/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, tvOS 16

## Overview
MapKit receives a major upgrade in iOS 16, bringing the all-new Apple Maps experience — including the 3D City Experience with handcrafted landmarks, realistic terrain elevation, turn lanes, crosswalks, and bike lanes — directly into third-party apps. Simply recompiling against the iOS 16 SDK opts apps in automatically. A new Map Configuration API replaces the older `MKMapType` and scattered `MKMapView` properties with a clean, composable configuration model.

Beyond the new map, iOS 16 adds significant enhancements to overlays (transparent buildings when pitched, elevated route lines that follow terrain), blend modes for creative overlay styling, a brand-new Selectable Map Features API letting users tap on built-in map POIs and territories, and Look Around — Apple's immersive street-level imagery — brought directly into MapKit for in-app embedding.

Hardware requirements: the 3D City Experience with realistic terrain requires A12-based iOS/iPadOS devices or M1 Macs. Older devices automatically fall back to the flat all-new map.

## Key Topics

### Map Configuration API
Replaces `MKMapType` enum and various `MKMapView` properties (soft deprecated in iOS 16). Three concrete subclasses of `MKMapConfiguration`:
- `MKImageryMapConfiguration` — satellite imagery only
- `MKHybridMapConfiguration` — imagery + road labels/POIs; controls POI filtering and traffic
- `MKStandardMapConfiguration` — vector map; `emphasisStyle` (.default or .muted); POI filtering; traffic toggle

All configurations support `elevationStyle`: `.flat` (default) or `.realistic` (3D terrain).

### Overlay Improvements
- `aboveRoads` is the new default overlay level in iOS 16
- **Transparent buildings**: When a pitched map has overlays at the `aboveRoads` level, buildings and trees automatically render with increasing transparency as pitch increases
- **Elevated route lines**: Overlays from MapKit's Directions API automatically follow realistic terrain elevation; regular overlays force the map to flat representation
- Semi-transparent overlays combine alpha values with transparent building alpha

### Blend Modes
New `blendMode` property on `MKOverlayRenderer` accepts any `CGBlendMode` value. Enables creative effects: desaturate surrounding areas, darken/highlight regions, color burn, hard light, saturation, etc. Blend order follows overlay insertion order.

### Selectable Map Features
New API allowing users to tap built-in map features (POIs, territories, physical features). Configure via `MKMapView.selectableMapFeatures` option set. Handled through existing and new delegate callbacks:
- `mapView(_:didSelect:)` — new callback when a map feature is selected
- `mapView(_:viewFor:)` — return nil for default gradient style, or a custom `MKMarkerAnnotationView`
- `MKMapFeatureAnnotation` — new annotation class with `featureType`, `pointOfInterestCategory`, and `iconStyle` properties
- `MKMapItemRequest` — resolve a feature annotation to a full `MKMapItem` with address, phone, URL

### Look Around
New in-app street-level imagery API:
- `MKLookAroundSceneRequest` — check data availability with coordinate or map item; async `scene` property returns optional `MKLookAroundScene`
- `MKLookAroundViewController` — interactive embeddable look around view; expandable to full screen
- `MKLookAroundSnapshotter` — static image snapshot of a look around scene
- Interface Builder support for embedding Look Around view controller

## APIs & Frameworks

**MapKit**

_Map Configuration_
- `MKMapConfiguration` **[NEW]** — abstract base configuration class
- `MKImageryMapConfiguration` **[NEW]** — satellite imagery config
- `MKHybridMapConfiguration` **[NEW]** — hybrid map config with POI/traffic controls
- `MKStandardMapConfiguration` **[NEW]** — vector map config with emphasis style
- `MKMapConfiguration.elevationStyle` **[NEW]** — `.flat` or `.realistic`
- `MKStandardMapConfiguration.emphasisStyle` **[NEW]** — `.default` or `.muted`
- `MKMapView.preferredConfiguration` **[NEW]** — replaces `mapType` (soft deprecated)

_Overlays_
- `MKOverlayLevel.aboveRoads` — new default level in iOS 16
- Transparent buildings effect **[NEW]** — automatic with `aboveRoads` overlays on pitched 3D map
- Elevated route overlay (from Directions API) **[NEW behavior]** — follows realistic terrain

_Blend Modes_
- `MKOverlayRenderer.blendMode` **[NEW]** — accepts `CGBlendMode` values

_Selectable Map Features_
- `MKMapView.selectableMapFeatures` **[NEW]** — `MKMapFeatureOptions` option set
- `MKMapFeatureOptions` **[NEW]** — `.pointsOfInterest`, `.territories`, `.physicalFeatures`
- `MKMapFeatureAnnotation` **[NEW]** — annotation for selected map features
- `MKMapFeatureAnnotation.featureType` **[NEW]** — type of the selected feature
- `MKMapFeatureAnnotation.pointOfInterestCategory` **[NEW]** — POI category if applicable
- `MKMapFeatureAnnotation.iconStyle` **[NEW]** — `MKIconStyle` with icon image and background color
- `MKIconStyle` **[NEW]** — icon styling info for a map feature
- `MKMapItemRequest` **[NEW]** — async request to resolve feature annotation to `MKMapItem`
- `mapView(_:didSelectAnnotation:)` delegate callback **[NEW]** — called when map feature selected

_Look Around_
- `MKLookAroundSceneRequest` **[NEW]** — request look around scene by coordinate or map item
- `MKLookAroundScene` **[NEW]** — opaque token representing available look around imagery
- `MKLookAroundViewController` **[NEW]** — interactive embeddable look around view
- `MKLookAroundSnapshotter` **[NEW]** — static image snapshot of look around scene
- `MKLookAroundSnapshotter.snapshot` **[NEW]** — async snapshot property

_Existing enhanced_
- `MKMapCamera(lookingAtForOver:)` **[NEW]** — create camera framed at a map item with pitch support
- `MKMapView.pointOfInterestFilter` — POI filtering (existing, used with new configs)

## Code Highlights

```swift
// New map configuration with realistic terrain
let config = MKStandardMapConfiguration(elevationStyle: .realistic,
                                          emphasisStyle: .muted)
mapView.preferredConfiguration = config

// Selectable map features
mapView.selectableMapFeatures = [.pointsOfInterest]

// Handling a selected feature
func mapView(_ mapView: MKMapView, didSelect annotation: MKAnnotation) {
    guard let feature = annotation as? MKMapFeatureAnnotation else { return }
    let request = MKMapItemRequest(mapFeatureAnnotation: feature)
    Task {
        let mapItem = try? await request.mapItem
        // use mapItem.name, mapItem.phoneNumber, etc.
    }
}

// Look Around availability check and display
let request = MKLookAroundSceneRequest(mapItem: mapItem)
if let scene = try? await request.scene {
    lookAroundVC.scene = scene
    previewContainer.isHidden = false
}
```

## Takeaways

- Recompiling against iOS 16 SDK automatically enables the all-new Apple Map with 3D City Experience (A12+ / M1+); use `MKStandardMapConfiguration(elevationStyle: .realistic)` for explicit terrain control.
- The Selectable Map Features API (`selectableMapFeatures`) enables users to tap built-in POIs and territories, with full delegate customization and `MKMapItemRequest` for rich place details.
- Look Around is now embeddable in any app via `MKLookAroundViewController` — check availability with `MKLookAroundSceneRequest` first as not every location has imagery.
- Blend modes on `MKOverlayRenderer` unlock creative map styling possibilities like highlighting regions or desaturating the surrounding map area.

---
_Source: WWDC22 Session 10035 page (abstract, chapter summaries, code samples, and resource links)._
