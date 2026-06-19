# Unlock the Power of Places with MapKit
**WWDC24 · Session 10097** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10097/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia 15, visionOS 2, watchOS 11

## Overview
MapKit and MapKit JS gain several significant new capabilities in 2024 centered on "places": unique Place IDs that stably reference real-world locations, Place Card APIs that surface rich place details (hours, phone, reviews) within apps and websites, and improved search APIs with address-component filtering and region-priority constraints.

Place IDs are persistent, unique identifiers for points of interest and physical features (parks, rivers, cities, etc.). They can be used as keys in app data structures and shared across native apps, web pages, and server backends. The Place ID Lookup tool at developer.apple.com/maps/resources makes finding IDs without code easy.

MapKit JS also gets a simplified token provisioning model: long-lived, domain-specific tokens generated from a new web tool replace the previous JWT-based dynamic generation, and a new Maps Embed API enables single-place map embeds without writing any JavaScript.

## Key Topics

### Referencing Places with Place ID
`MKMapItem.Identifier` wraps a Place ID string. Pass it to `MKMapItemRequest(mapItemIdentifier:)` and call `.mapItem` (async) to fetch a live `MKMapItem`. In MapKit JS, `mapkit.PlaceLookup().getPlace(id, callback)` performs the same lookup. Place IDs work for POIs, physical features, and address components; Apple's editorial team keeps them fresh, so apps automatically inherit data corrections.

### Displaying Place Details (Place Cards)
Three presentation options in SwiftUI/UIKit/AppKit and MapKit JS:
- **On-map callout**: `.mapItemDetailSelectionAccessory()` or `.mapItemDetailSelectionAccessory(.callout)` modifier on map content shows a Place Card when that annotation is selected.
- **Sheet/detail view**: `.mapItemDetailSheet(item:)` (SwiftUI) presents a full Place Card sheet when a `MKMapItem` binding is non-nil.
- **Standalone detail view** (no map required): `MapItemDetail` (SwiftUI), `MKMapItemDetailViewController` (UIKit/AppKit).
- **Web**: `mapkit.PlaceSelectionAccessory()` attached to a `PlaceAnnotation`, or `mapkit.PlaceDetail(element, place, { colorScheme: ... })` to embed place details directly in a page element.
- **Map features (not just your annotations)**: `.mapFeatureSelectionAccessory(.callout)` enables Place Cards for all tappable map features on the map.

### Improved Search APIs
`MKAddressFilter(including:)` restricts search results to specific address components (`MKAddressCategory`): `.locality` (cities), `.postalCode`, `.country`, etc. `MKLocalSearch.Request.regionPriority = .required` ensures results lie within the specified region (previously results could spill outside). In MapKit JS: `mapkit.AddressFilter.including([mapkit.AddressCategory.Locality])` and `regionPriority: mapkit.Search.RegionPriority.Required`. MapKit Server API now supports pagination for large result sets.

### Web Embed API and Simplified Token Provisioning
A new "Create a Map" web tool at developer.apple.com/maps/resources generates `<iframe>` HTML snippets for embedding a single place's map without writing JavaScript. A new token provisioning tool generates long-lived, domain-scoped tokens, replacing dynamic JWT generation. The tokens are placed directly in the `data-token` attribute of the MapKit JS `<script>` tag.

## APIs & Frameworks

- `MapKit` framework (native)
- `MKMapItem.Identifier` **[NEW]** — wraps a Place ID string to identify a specific place
- `MKMapItemRequest(mapItemIdentifier:)` **[NEW]** — async request to fetch an `MKMapItem` by Place ID
- `MKMapItemRequest.mapItem` **[NEW]** — async property returning the fetched `MKMapItem`
- `.mapItemDetailSelectionAccessory()` **[NEW]** — SwiftUI modifier; shows Place Card when annotation selected
- `.mapItemDetailSelectionAccessory(.callout)` **[NEW]** — callout style Place Card on annotation selection
- `.mapFeatureSelectionAccessory(.callout)` **[NEW]** — enables Place Cards for map features (POIs on the basemap)
- `.mapItemDetailSheet(item:)` **[NEW]** — SwiftUI modifier; presents a Place Card sheet for a bound `MKMapItem`
- `MapItemDetail` **[NEW]** — SwiftUI view for displaying Place Card content (no map required)
- `MKMapItemDetailViewController` **[NEW]** — UIKit/AppKit view controller for displaying Place Card
- `MapSelection<MKMapItem>` **[NEW]** — selection type supporting both custom annotations and map features
- `MKAddressFilter` **[NEW]** — filters search results to specific address components
- `MKAddressFilter(including:)` **[NEW]** — initializer accepting an array of `MKAddressCategory`
- `MKAddressCategory` **[NEW]** — enum of address component types (`.locality`, `.postalCode`, `.country`, etc.)
- `MKLocalSearch.Request.regionPriority` **[NEW]** — `.required` constrains results to the specified region
- `MKLocalSearch.Request.addressFilter` **[NEW]** — attaches an `MKAddressFilter` to a search request
- `MKLocalSearch`, `MKLocalSearch.Request` — existing search classes (unchanged)
- `Marker(item:)` — SwiftUI map marker from an `MKMapItem`
- `Map(selection:)` — SwiftUI map with selection binding
- `MapKit JS` (web)
- `mapkit.PlaceLookup` **[NEW]** — JS class for looking up places by Place ID
- `mapkit.PlaceLookup.getPlace(id, callback)` **[NEW]** — async Place ID lookup
- `mapkit.PlaceAnnotation` **[NEW]** — JS annotation type built from a Place object
- `mapkit.PlaceSelectionAccessory` **[NEW]** — JS class that attaches a Place Card to a `PlaceAnnotation`
- `mapkit.PlaceDetail` **[NEW]** — JS class for embedding place details in a page element (no map required)
- `mapkit.PlaceDetail.ColorSchemes.Adaptive` **[NEW]** — matches system light/dark color scheme
- `mapkit.AddressFilter` **[NEW]** — JS search filter for address components
- `mapkit.AddressFilter.including([categories])` **[NEW]**
- `mapkit.AddressCategory.Locality` **[NEW]**
- `mapkit.Search.RegionPriority.Required` **[NEW]** — constrains JS search results to region
- Place ID Lookup tool — developer.apple.com/maps/resources web tool for finding Place IDs
- Maps Embed API (Web Embed) **[NEW]** — generates `<iframe>` HTML for embedding single-place maps
- Token Provisioning tool **[NEW]** — generates long-lived, domain-scoped MapKit JS tokens

## Code Highlights

Fetch and display a place by Place ID (SwiftUI):
```swift
let request = MKMapItemRequest(mapItemIdentifier: MKMapItem.Identifier(rawValue: placeID)!)
item = try? await request.mapItem
// Then: Marker(item: item)
```

Show Place Card on annotation tap (SwiftUI):
```swift
Map(selection: $selection) {
    ForEach(visitedStores, id: \.self) { store in
        Marker(item: store)
    }
    .mapItemDetailSelectionAccessory(.callout)
}
.mapFeatureSelectionAccessory(.callout)
```

Show Place Card sheet from a list (SwiftUI):
```swift
List(stores, id: \.self, selection: $selectedStore) { Text($0.name ?? "") }
    .mapItemDetailSheet(item: $selectedStore)
```

Search for cities only (Swift):
```swift
let request = MKLocalSearch.Request()
request.naturalLanguageQuery = "cupertino"
request.addressFilter = MKAddressFilter(including: .locality)
```

Search restricted to a region (Swift):
```swift
request.region = downtown
request.regionPriority = .required
```

## Takeaways

- Place ID (`MKMapItem.Identifier`) provides a stable, persistent, shareable reference to real-world places—use it instead of hardcoded coordinates so your app inherits Apple Maps' editorial updates automatically.
- The new `SelectionAccessory` APIs (`.mapItemDetailSelectionAccessory()`, `.mapFeatureSelectionAccessory()`, `.mapItemDetailSheet(item:)`) make it trivial to display rich Place Card information for both your custom annotations and all map features.
- `MKAddressFilter` and `regionPriority = .required` give precise control over what search results come back, enabling efficient address-component searches (city-only, postcode-only, etc.) and region-bounded POI searches.
- MapKit JS now supports long-lived domain-scoped tokens (no more JWT generation) and a Maps Embed API for zero-JavaScript place embeds on websites.

---
_Source: WWDC24 Session 10097 page (abstract, chapter summaries, code samples, and resource links)._
