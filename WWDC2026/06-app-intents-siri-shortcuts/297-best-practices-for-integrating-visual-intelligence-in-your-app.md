# Best Practices for Integrating Visual Intelligence in Your App
**WWDC26 · Session 297** · [Watch](https://developer.apple.com/videos/play/wwdc2026/297/)

_Platforms:_ iOS, iPadOS, macOS

## Overview
This session covers how to integrate your app with Visual Intelligence — the system feature that lets users point their camera (or take a screenshot on Mac/iPad) at content and get relevant results from installed apps. The session uses a music-discovery sample app throughout and covers the full integration: defining entities, implementing an image-similarity query, opening results, returning multiple result types with `@UnionValue`, providing an in-app search fallback, and reading back data written to system stores.

iOS 26 expands Visual Intelligence to iPadOS and macOS with the same entity/query/intent model, requiring only minor platform-specific adaptations. The session also covers three system store integrations — EventKit, Contacts, and HealthKit — where Visual Intelligence can write data that your app then reads back automatically via store-change notifications.

## Key Topics

### Defining Your Content
Model your app's content as an `AppEntity` with a `DisplayRepresentation` (title, subtitle, thumbnail image). Best practices: use concise identifying text and thumbnail-sized images — Visual Intelligence results are compact. Implement `EntityQuery` with `entities(for:)` to allow the system to retrieve full entity details.

### Implementing a Query
Implement `IntentValueQuery` to receive a `SemanticContentDescriptor` containing a pixel buffer from the camera or screenshot. Use Vision's `GenerateImageFeaturePrintRequest` to compute feature prints for your catalog, then compare query print distance against pre-computed catalog prints. Keep a pre-built `CatalogEntry` array with `(entity, featurePrint)` pairs; the `distance(to:)` method on `FeaturePrintObservation` drives similarity ranking. Use a `maxDistance` threshold to filter low-quality matches.

### Opening Results
Implement `OpenIntent` so tapping a Visual Intelligence result opens your app directly to that content. Keep `perform()` lightweight since it runs as the app foregrounds. Reuse an existing `OpenIntent` if you already have one — no need to create a Visual Intelligence-specific version.

### Mac and iPad Adoption
The same entities, query, and intent carry over to iPadOS and macOS with minimal changes. Key differences:
- **Mac**: input comes from screenshots, not camera; pixel buffers are much larger and may need resizing before computing feature prints
- **iPad**: same camera-based input as iPhone

### Returning Multiple Result Types
Use `@UnionValue` to return more than one entity type from a single `IntentValueQuery`. Encourage deriving related content — the sample returns albums plus nearby concerts inferred from matching artist names — not just direct pixel matches.

### Continuing Search in Your App
Adopt the `.visualIntelligence.semanticContentSearch` schema to provide a "Search in [App]" entry point that pre-populates your full in-app search with the captured context. Set `openAppWhenRun: true`.

### System Store Integrations
Visual Intelligence can write to three system stores that your app reads back:
- **EventKit** (`EKEventStore`) — calendar events; observe `EKEventStoreChanged` notifications
- **CNContactStore** — contacts; observe `CNContactStoreDidChange` notifications
- **HealthKit** (`HKHealthStore`) — medical device readings; use `HKObserverQuery`

Request access, fetch on load, and register for store-change notifications so captured data appears automatically in your app.

## APIs & Frameworks

### VisualIntelligence (new framework)
- `SemanticContentDescriptor` — **[NEW]** input to `IntentValueQuery`; `.pixelBuffer` property (`CVReadOnlyPixelBuffer`)
- `CVReadOnlyPixelBuffer` — pixel buffer type from `SemanticContentDescriptor`
- `@AppIntent(schema: .visualIntelligence.semanticContentSearch)` — **[NEW]** in-app search continuation schema
- `VisualIntelligence` documentation: `https://developer.apple.com/documentation/VisualIntelligence`

### AppIntents
- `AppEntity` protocol — `id`, `displayRepresentation`, `defaultQuery`
- `@Property` property wrapper — entity property declaration
- `DisplayRepresentation` — `title:`, `subtitle:`, `image:` (thumbnail)
- `TypeDisplayRepresentation`
- `EntityQuery` protocol — `entities(for:)` method
- `IntentValueQuery` protocol — `values(for:)` method; receives `SemanticContentDescriptor` **[NEW for visual intelligence]**
- `OpenIntent` protocol — `target` parameter, `perform()` for deep-link navigation
- `@UnionValue` macro — **[NEW]** multi-type return from a single query; `typeDisplayRepresentation`, `caseDisplayRepresentations`
- `@AppIntent(schema:)` — schema-based intent declaration
- `@Dependency` property wrapper — inject `AlbumCatalog`, `ConcertFinder`, `AppState`
- `openAppWhenRun: Bool` — static property on `AppIntent`; set `true` for in-app search intents

### Vision
- `GenerateImageFeaturePrintRequest` — **[NEW / updated]** on-device image feature print computation; `perform(on:)` async method
- `FeaturePrintObservation` — `.distance(to:)` method for similarity scoring

### EventKit
- `EKEventStore` — `requestFullAccessToEvents()` async method
- `EKEventStore` — `predicateForEvents(withStart:end:calendars:)`, `.events(matching:)`
- `EKAuthorizationStatus` — `.notDetermined`, `.denied`, `.fullAccess`
- `NSNotification.Name.EKEventStoreChanged` — observe for store updates

### Contacts
- `CNContactStore` — contact access and store-change observation
- `NSNotification.Name.CNContactStoreDidChange` — observe for new contacts

### HealthKit
- `HKHealthStore` — health data access
- `HKObserverQuery` — observe new health readings

### Foundation / AVFoundation
- `CVReadOnlyPixelBuffer` — `withUnsafeBuffer(_:)` for `CGImage` conversion
- `VTCreateCGImageFromCVPixelBuffer` — convert pixel buffer to `CGImage` for Vision

## Code Highlights

**IntentValueQuery receiving a pixel buffer:**
```swift
struct SearchHandler: IntentValueQuery {
    @Dependency var catalog: AlbumCatalog

    func values(for input: SemanticContentDescriptor) async throws -> [VisualSearchResult] {
        guard let pixelBuffer = input.pixelBuffer else { return [] }
        let albums = try await catalog.search(matching: pixelBuffer)
        return albums.map { VisualSearchResult.album($0) }
    }
}
```

**On-device similarity search with feature prints:**
```swift
func search(matching pixelBuffer: CVReadOnlyPixelBuffer, maxDistance: Double = 1.0) async throws -> [AlbumEntity] {
    // Convert pixel buffer to CGImage, compute feature print, compare distances
    let queryPrint = try await generateFeaturePrint(for: cgImage)
    return try entries.compactMap { entry in
        let distance = try queryPrint.distance(to: entry.featurePrint)
        return distance <= maxDistance ? (entry.album, distance) : nil
    }.sorted { $0.distance < $1.distance }.map(\.album)
}
```

**@UnionValue for multiple result types:**
```swift
@UnionValue
enum VisualSearchResult {
    case album(AlbumEntity)
    case concert(ConcertEntity)
}
```

**In-app search continuation:**
```swift
@AppIntent(schema: .visualIntelligence.semanticContentSearch)
struct SemanticContentSearchIntent: AppIntent {
    static let openAppWhenRun: Bool = true
    var semanticContent: SemanticContentDescriptor

    func perform() async throws -> some IntentResult {
        guard let pixelBuffer = semanticContent.pixelBuffer else { return .result() }
        let albums = try await catalog.search(matching: pixelBuffer)
        await appState.openSearch(albums: albums, concerts: concerts)
        return .result()
    }
}
```

**EventKit store-change observation:**
```swift
for await _ in NotificationCenter.default.notifications(named: .EKEventStoreChanged) {
    await fetchUpcomingConcerts()
}
```

## Takeaways
- `IntentValueQuery` with `SemanticContentDescriptor` is the single integration point for Visual Intelligence image search — implement it alongside `AppEntity` and `OpenIntent` for a complete integration.
- Vision's `GenerateImageFeaturePrintRequest` + pre-computed `FeaturePrintObservation` catalog is the recommended on-device approach: fast, private, and distance-based.
- `@UnionValue` enables richer results by returning related entity types (concerts derived from album artist names) — go beyond direct pixel matches to surface contextually relevant content.
- The `.visualIntelligence.semanticContentSearch` schema gives users a "Search in App" escape hatch to your full search UI; pair it with `openAppWhenRun: true`.

---
_Source: WWDC26 Session 297 page (abstract, chapter summaries, code samples, and resource links)._
