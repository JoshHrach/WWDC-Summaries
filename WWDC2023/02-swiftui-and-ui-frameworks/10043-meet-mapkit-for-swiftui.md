# Meet MapKit for SwiftUI
**WWDC23 · Session 10043** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10043/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, tvOS 17, watchOS 10, visionOS 1

## Overview
MapKit for SwiftUI receives a greatly expanded API in 2023, making it easier than ever to integrate rich, interactive maps across all Apple platforms with just a few lines of SwiftUI code. The session walks through building a complete trip-planning app from scratch, demonstrating annotations, overlays, camera control, map styles, Look Around, and map controls.

The new `Map` initializer accepts a `MapContentBuilder` closure analogous to SwiftUI's `ViewBuilder`, allowing `Marker`, `Annotation`, and overlay types to be composed declaratively alongside other SwiftUI views. Camera position is managed through a binding to `MapCameraPosition`, giving the app fine-grained control over what region or item is in view without manually configuring a `MapCamera`.

Look Around integration and driving routes via `MKDirections` can be combined with search results from `MKLocalSearch` to build feature-complete location-aware experiences entirely within SwiftUI.

## Key Topics

### Markers and Annotations
- `Marker` displays a balloon-shaped pin at a coordinate; supports custom icon (SF Symbol, image asset), monogram text, and tint color
- `Marker` initialized with `MKMapItem` automatically picks up the item's name, icon, and tint color
- `Annotation` displays an arbitrary SwiftUI `View` at a coordinate; `anchor` parameter controls alignment relative to the coordinate point

### Map Styles
- `.standard` – default flat style; optional `elevation: .realistic` parameter renders 3-D terrain **[NEW]**
- `.imagery` – satellite / flyover imagery
- `.hybrid` – imagery plus roads and labels

### Camera Position (`MapCameraPosition`)
- `.automatic` – frames all content added via the content builder
- `.region(MKCoordinateRegion)` – shows a geographic region
- `.rect(MKMapRect)` – shows an area via map rect
- `.item(MKMapItem)` – centers on a place, auto-zooming to fit
- `.camera(MapCamera)` – exact camera with pitch, heading, distance
- `.userLocation(fallback:)` – follows user; falls back when location unavailable
- `.positionedByUser` – set by MapKit when the user pans/zooms away from a programmatic position

### Camera Change Callbacks
- `onMapCameraChange(frequency:)` modifier – closure receives `MapCameraUpdateContext` with `region`, `rect`, and `camera` properties
- `frequency: .continuous` for live updates during gesture; default fires on gesture end

### Selection
- `Map(selection:)` initializer with a binding to `MKMapItem?` makes map-item markers selectable
- Tag-based selection with `.tag(_:)` modifier for heterogeneous content types

### Overlays
- `MapPolyline` – draws a polyline; can be initialized from `MKRoute.polyline` or from coordinate arrays
- `MapPolygon` – highlights a closed area
- `MapCircle` – highlights a circular area
- `.stroke(_:style:)` / `.fill(_:)` modifiers; `MapOverlayLevel` (`.aboveRoads`, `.aboveLabels`)

### User Location
- `UserAnnotation` – displays the standard blue dot at the user's current location

### Map Controls
- `MapUserLocationButton` **[NEW]** – centers map on user location
- `MapCompass` – shows compass when map is rotated
- `MapScaleView` – shows scale indicator while zooming
- `MapZoomStepper` – macOS zoom controls
- `MapPitchSlider` – macOS pitch control
- `.mapControls { }` modifier – positions controls in default system locations
- `mapScope` modifier – links standalone control views to a specific `Map`

### Look Around
- `MKLookAroundSceneRequest` – fetches a Look Around scene for an `MKMapItem`
- `LookAroundPreview` SwiftUI view **[NEW]** – displays the fetched scene inline

### Routing
- `MKDirections` – asynchronous driving/walking directions
- `MKRoute` – result containing `polyline` and `expectedTravelTime`
- `DateComponentsFormatter` – formats `expectedTravelTime` for display

## APIs & Frameworks

- **MapKit** / **MapKit for SwiftUI** **[NEW expanded API]**
- `Map` – SwiftUI map view with `MapContentBuilder` closure, `selection:`, `scope:` **[NEW]**
- `MapContentBuilder` **[NEW]** – result builder for map content
- `Marker` **[NEW SwiftUI type]** – balloon annotation for coordinates and `MKMapItem`
- `Annotation` **[NEW SwiftUI type]** – custom SwiftUI view annotation with `anchor` parameter
- `MapCameraPosition` **[NEW]** – enum controlling camera: `.automatic`, `.region`, `.rect`, `.item`, `.camera`, `.userLocation`, `.positionedByUser`
- `MapCamera` – value type with coordinate, distance, heading, pitch
- `onMapCameraChange(frequency:)` modifier **[NEW]**
- `MapCameraUpdateContext` **[NEW]** – context passed to camera change closure
- `MapCameraUpdateFrequency` (`.onEnd`, `.continuous`) **[NEW]**
- `MapPolyline` **[NEW SwiftUI type]** – polyline overlay
- `MapPolygon` **[NEW SwiftUI type]** – polygon overlay
- `MapCircle` **[NEW SwiftUI type]** – circle overlay
- `MapOverlayLevel` **[NEW]** (`.aboveRoads`, `.aboveLabels`)
- `.stroke(_:style:)` / `.fill(_:)` overlay modifiers **[NEW]**
- `UserAnnotation` **[NEW SwiftUI type]** – user location blue dot
- `MapUserLocationButton` **[NEW SwiftUI control]**
- `MapCompass` **[NEW SwiftUI control]**
- `MapScaleView` **[NEW SwiftUI control]**
- `MapZoomStepper` **[NEW SwiftUI control, macOS]**
- `MapPitchSlider` **[NEW SwiftUI control, macOS]**
- `.mapControls { }` modifier **[NEW]**
- `mapScope` modifier **[NEW]**
- `.mapStyle(_:)` modifier **[NEW]** with `.standard(elevation:)`, `.imagery`, `.hybrid`
- `MapStyle.standard(elevation: .realistic)` **[NEW]**
- `LookAroundPreview` **[NEW SwiftUI view]**
- `MKLookAroundSceneRequest` – async scene fetch
- `MKLocalSearch` / `MKLocalSearch.Request` – place search
- `MKMapItem` – represents a place
- `MKDirections` / `MKDirections.Request` – routing
- `MKRoute` – route result with `polyline` and `expectedTravelTime`
- `.safeAreaInset(edge:)` modifier – prevents UI from obscuring map content
- `.tag(_:)` modifier – tag-based selection for heterogeneous map content

## Code Highlights

Minimal interactive map:
```swift
import MapKit
import SwiftUI

struct ContentView: View {
    var body: some View {
        Map()
    }
}
```

Marker and Annotation with content builder:
```swift
Map {
    Marker("Parking", coordinate: parkingCoordinate)
    Annotation("Start", coordinate: startCoordinate, anchor: .bottom) {
        ZStack {
            Circle().fill(.blue)
            Image(systemName: "figure.walk")
        }
    }
}
```

Camera position bound to state, updating on search results:
```swift
@State private var position: MapCameraPosition = .automatic
@State private var searchResults: [MKMapItem] = []

Map(position: $position) {
    ForEach(searchResults, id: \.self) { item in
        Marker(item: item)
    }
}
.onChange(of: searchResults) {
    position = .automatic
}
```

Look Around Preview with scene request:
```swift
@State private var scene: MKLookAroundScene?

LookAroundPreview(scene: $scene)
    .task(id: selectedResult) {
        scene = try? await MKLookAroundSceneRequest(mapItem: selectedResult).scene
    }
```

Route polyline overlay:
```swift
if let route {
    MapPolyline(route.polyline)
        .stroke(.blue, lineWidth: 5)
}
```

## Takeaways
- A fully functional, interactive map with annotations, overlays, Look Around, and controls can be built entirely in SwiftUI with the new MapKit for SwiftUI APIs.
- `MapCameraPosition` decouples "what to show" from "how the camera is configured"—use `.automatic` to frame content and `.region`/`.item` to navigate programmatically.
- `onMapCameraChange` provides the visible region in real time, enabling search-in-visible-area patterns without any UIKit bridge code.
- Use `safeAreaInset` to layer UI above the map without obscuring Apple Maps attribution or system controls.

---
_Source: WWDC23 Session 10043 page (abstract, chapter summaries, code samples, and resource links)._
