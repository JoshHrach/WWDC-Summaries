# What's New in App Intents
**WWDC24 · Session 10134** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10134/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia 15, visionOS 2, watchOS 11

## Overview
App Intents in 2024 expands in four major directions: deeper Spotlight integration via a new `IndexedEntity` protocol that feeds app entities into Core Spotlight's semantic search; richer entity representations through `Transferable`, `FileEntity`, and updated `IntentFile` APIs that let app concepts be understood and used by other apps; universal link support through `URLRepresentableEntity`, `URLRepresentableIntent`, and `URLRepresentableEnum`; and a set of developer experience improvements including the `@UnionValue` macro, optional parameter titles, and cross-module entity references.

App Intents is central to Apple Intelligence and the new Controls feature in iOS 18. The session's running example is a Trails app—indexing trail entities in Spotlight, making activity summaries transferable as rich text and PNG, deep-linking to specific trails via universal links, and buying day passes with a union-typed parameter.

## Key Topics

### Spotlight Integration with IndexedEntity
The new `IndexedEntity` protocol provides a simple path to index app entities in `CSSearchableIndex`. Conforming types automatically generate a `CSSearchableItem` attribute set from their `DisplayRepresentation`. Developers can override `attributeSet` to supply richer metadata such as location keywords, city/state information, or activity tags. Existing `CSSearchableItem` workflows can adopt entities via the new `associateAppEntity(_:)` method. An optional `priority` value (higher = more important) lets favorites surface above ordinary items. Indexed entities appear in Spotlight search results and can trigger deep-link actions (`OpenIntent`) directly from the results list.

### Entities and Files: Transferable, IntentFile, FileEntity
`AppEntity` types can now conform to `Transferable`, declaring `DataRepresentation` and `FileRepresentation` transfer items in priority order (highest fidelity first). Siri and Shortcuts automatically convert entities to requested formats and can pass them to Mail, Photos, and other apps. Representations must be compilable at compile time; `ProxyRepresentation` may only reference `@Property`-attributed properties.

`IntentFile` gains new APIs for checking available content types and accessing a file URL for conversion. An `AppIntent` receiving an `IntentFile` parameter declares which content types it supports, and App Intents uses the `Transferable` representation to convert automatically.

`FileEntity` is designed for document-based apps where the entity _is_ a file (not a database object). It requires a `FileEntityIdentifier` (wrapping a URL's bookmark data, surviving renames/moves) and a list of supported `UTType` values. Shortcuts and Siri can grant other apps secure access to the underlying file, enabling intents in third-party apps (e.g., an image-rotation action) to operate directly on the file.

### Universal Links with URLRepresentable
Conforming an `AppEntity` to `URLRepresentableEntity` exposes a static `URLRepresentation` template using entity ID or `@Property`-attributed properties as interpolations. Conforming an `AppIntent` (typically an `OpenIntent`) to `URLRepresentableIntent` means no `perform` body is required—App Intents invokes the existing URL handler. Return `OpenURLIntent` from any `perform` to deep-link to newly created or discovered content. The same protocol is available for `AppEnum` as `URLRepresentableEnum`.

### Developer Improvements
- **`@UnionValue` macro**: Attach to an enum where each case has exactly one associated value of a distinct type. Use as an `@Parameter` type; switch on it like a normal enum. Enables a single parameter accepting `ParkEntity` _or_ `TrailEntity` without creating parallel intents.
- **Optional parameter titles**: When building with Xcode 16, `@Parameter` and `@Property` titles are auto-generated from the Swift property name; only specify a title string when the auto-generated string is wrong.
- **Cross-module entities**: In Xcode 16, app entities defined in framework targets can be referenced from app and extension targets. (Libraries outside frameworks are not yet supported.)

## APIs & Frameworks

- `App Intents` framework
- `IndexedEntity` **[NEW]** — protocol to index `AppEntity` types in Core Spotlight
  - `CSSearchableIndex.indexAppEntities(_:)` **[NEW]** — indexes an array of `IndexedEntity` conformances
  - `IndexedEntity.attributeSet` **[NEW]** — optional override to customize `CSSearchableItemAttributeSet`
  - `CSSearchableItem.associateAppEntity(_:priority:)` **[NEW]** — associates an `AppEntity` with an existing searchable item
  - `priority` **[NEW]** — optional integer on `IndexedEntity`; higher value = higher Spotlight ranking
- `Transferable` — existing protocol; `AppEntity` conformance is **[NEW]**
  - `DataRepresentation(exportedContentType:)` — exports entity as `Data` with a UTType
  - `FileRepresentation(exportedContentType:)` — exports entity as a file with a UTType
- `IntentFile` — existing type; new APIs **[NEW]**:
  - `IntentFile.supportedContentTypes` **[NEW]** — checks available content types on the file
  - `IntentFile.fileURL` **[NEW]** — URL for performing custom conversion
- `FileEntity` **[NEW]** — protocol for `AppEntity` types that are backed by files
  - `FileEntityIdentifier` **[NEW]** — identifier wrapping URL bookmark data; survives renames and moves
  - `FileEntityIdentifier(url:)` **[NEW]** — creates identifier from a file URL
  - `FileEntityIdentifier(draftIdentifier:)` **[NEW]** — creates a draft identifier for not-yet-created files
  - `FileEntity.supportedContentTypes` **[NEW]** — required array of `UTType` values for the entity
- `URLRepresentableEntity` **[NEW]** — protocol adding `URLRepresentation` to an `AppEntity`
  - `URLRepresentation` **[NEW]** — static string template with entity ID or `@Property` interpolations
- `URLRepresentableIntent` **[NEW]** — protocol; `OpenIntent` conformers get automatic URL-based perform
- `URLRepresentableEnum` **[NEW]** — protocol for `AppEnum` types with URL representations
- `OpenURLIntent` **[NEW]** — intent returned from `perform` to open a URL or entity-based URL
- `@UnionValue` **[NEW]** — macro for enum types where each case has exactly one distinct associated value; usable as `@Parameter` type
- `@Parameter` — existing; title is now auto-generated from property name in Xcode 16 **[NEW behavior]**
- `@Property` — existing; title auto-generation in Xcode 16 **[NEW behavior]**
- Cross-module entity references **[NEW in Xcode 16]** — entities in framework targets referenceable from app/extension targets
- `CSSearchableIndex` — Core Spotlight index (existing)
- `CSSearchableItem` — Core Spotlight item (existing)
- `CSSearchableItemAttributeSet` — attribute set for rich Spotlight metadata (existing)
- `OpenIntent` — existing protocol for intents that open content
- `AppEntity`, `AppEnum`, `AppIntent` — core App Intents protocols (existing)
- `DisplayRepresentation` — existing; used as default for `IndexedEntity` attribute set

## Code Highlights

Conform to `IndexedEntity` and index all entities on launch:
```swift
extension TrailEntity: IndexedEntity {
    var attributeSet: CSSearchableItemAttributeSet {
        let set = CSSearchableItemAttributeSet()
        set.city = location.city
        set.stateOrProvince = location.state
        set.keywords = supportedActivities.map(\.rawValue)
        return set
    }
}

// In app init:
try await CSSearchableIndex.default().indexAppEntities(dataManager.trails)
```

Conform an `AppEntity` to `Transferable`:
```swift
extension ActivityStatisticSummaryEntity: Transferable {
    static var transferRepresentation: some TransferRepresentation {
        DataRepresentation(exportedContentType: .rtf) { entity in
            entity.asRichTextData()
        }
        FileRepresentation(exportedContentType: .png) { entity in
            SentTransferredFile(entity.pngFileURL)
        }
    }
}
```

Add `URLRepresentableEntity` and `URLRepresentableIntent` for deep linking:
```swift
extension TrailEntity: URLRepresentableEntity {
    static var urlRepresentation: URLRepresentation {
        "https://trailsapp.example.com/trail/\(\.id)"
    }
}

struct OpenTrailIntent: OpenIntent, URLRepresentableIntent {
    static let title: LocalizedStringResource = "Open Trail"
    @Parameter(title: "Trail") var target: TrailEntity
    // No perform needed — App Intents calls URL handler automatically
}
```

Use `@UnionValue` for a parameter accepting multiple entity types:
```swift
@UnionValue
enum DayPassType {
    case trail(TrailEntity)
    case park(ParkEntity)
}

struct BuyDayPassIntent: AppIntent {
    @Parameter var passType: DayPassType
    func perform() async throws -> some IntentResult {
        switch passType {
        case .trail(let trail): // purchase for trail
        case .park(let park):   // purchase for park
        }
    }
}
```

## Takeaways

- `IndexedEntity` makes it straightforward to surface app entities in Spotlight's new semantic search and give Siri structured access to app content—just conform, override `attributeSet` for richer metadata, and call `indexAppEntities` on launch.
- `Transferable` conformance on `AppEntity` enables Shortcuts and Siri to automatically convert your app's concepts into files, images, and rich text that any app can consume, without you writing glue code for each destination.
- `FileEntity` with `FileEntityIdentifier` (bookmark-based) lets document-centric apps share secure file access with third-party intents, surviving moves and renames.
- `URLRepresentableIntent` eliminates boilerplate for open-app intents—just provide the URL template and App Intents invokes your existing URL handler.
- The `@UnionValue` macro, optional parameter titles, and cross-module entity references in Xcode 16 collectively reduce the verbosity and module-boundary friction of building complex intent graphs.

---
_Source: WWDC24 Session 10134 page (abstract, chapter summaries, transcript, and resource links)._
