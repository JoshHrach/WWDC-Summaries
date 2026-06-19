# Build an App with SwiftData
**WWDC23 · Session 10154** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10154/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, tvOS 17, watchOS 10, visionOS 1

## Overview
This code-along session walks through integrating SwiftData into a multi-platform SwiftUI flashcards app, covering everything from annotating an existing model class to querying data in views, managing the model context, and building a SwiftData-backed document-based application. It is a practical companion to "Meet SwiftData," showing the concrete SwiftUI patterns that make the framework feel seamless in a real project.

The session demonstrates that adopting SwiftData in an existing SwiftUI app requires minimal changes: adding `@Model` to a model class, replacing `@State` with `@Query` at query call sites, and setting up a `modelContainer` on the scene or window group. The bonus section shows how the new `DocumentGroup(editing:contentType:)` initializer enables document-based apps with almost no additional work.

## Key Topics

### SwiftData Models with `@Model`
- Add `@Model` macro to an existing class to make it persistable — that's the entire model change.
- `@Model` automatically synthesizes `Observable` conformance (from the Observation framework), eliminating the need for `ObservableObject` and `@Published` property wrappers.
- Properties of `@Model` types become observable; views that read them update automatically when values change.
- Replace `@ObservedObject` with `@Bindable` in views that need two-way binding to a model's properties.

### Querying Models with `@Query`
- Replace `@State` (or hardcoded data) with `@Query` to pull models from SwiftData storage and keep the view updated on changes.
- `@Query` triggers view updates on every model change, similar to `@State`.
- Supports lightweight configuration for sorting, ordering, filtering, and animating changes — all at the property declaration site.
- `@Query` reads from the view's model context, which is populated automatically from the closest `modelContainer` in the view hierarchy.

### Model Container Setup
- `.modelContainer(for:)` view/scene modifier — sets up the full SwiftData storage stack (including the model context that `@Query` draws from).
- Set on `WindowGroup` to share one container across all windows in that group.
- Multiple containers can be set on separate windows or subviews for isolated storage stacks.
- For previews: provide a custom in-memory container with sample data using `.modelContainer(previewContainer)`.

### Model Context and Saving
- `@Environment(\.modelContext)` — access the view's model context for inserting, deleting, and saving.
- `modelContext.insert(object:)` — registers a new model instance with SwiftData.
- **Autosave**: SwiftData automatically saves the context on UI-related events and user input — explicit `modelContext.save()` is rarely needed (only before sharing the storage file or sending it over a network).

### Document-Based Apps with SwiftData **[NEW]**
- `DocumentGroup(editing:contentType:)` — new initializer that creates a document-based app backed by SwiftData, with full platform document infrastructure (open panel, save dialog, Command+N/O/S keyboard shortcuts).
- No manual `modelContainer` setup needed — the document infrastructure creates and manages one per document.
- Each SwiftData document is a **package** (a directory), because properties marked `@Attribute(.externalStorage)` store their data as separate files inside the package.
- A custom `UTType` must be declared in code and in `Info.plist` with a unique identifier, file extension, and conformance to `com.apple.package`.
- Supported on iOS, iPadOS, and macOS; use `#if os(iOS) || os(macOS)` to select between `DocumentGroup` and standard `WindowGroup`.

## APIs & Frameworks

### SwiftData **[NEW]**
- `@Model` macro — converts a Swift class into a SwiftData persistable model; synthesizes `Observable` conformance **[NEW]**
- `@Attribute(.externalStorage)` — stores large property data outside the model file, as part of the document package **[NEW]**
- `ModelContainer` — storage stack (context + persistent store) **[NEW]**
- `.modelContainer(for:)` view/scene modifier — create and inject a model container **[NEW]**
- `ModelContext` — unit-of-work for inserting, deleting, fetching, saving models **[NEW]**
- `ModelContext.insert(object:)` — register a new model instance **[NEW]**
- `ModelContext.save()` — explicitly persist pending changes (usually not needed due to autosave) **[NEW]**
- `ModelContext.delete(_:)` — delete a model instance **[NEW]**
- `@Environment(\.modelContext)` — access current view's model context **[NEW]**

### SwiftUI + SwiftData Integration **[NEW]**
- `@Query` property wrapper — fetch and observe models from SwiftData; auto-updates view on change **[NEW]**
- `@Bindable` property wrapper — two-way binding to `Observable`/`@Model` types (replaces `@ObservedObject`) **[NEW]**
- `DocumentGroup(editing:contentType:)` — document-based app scene backed by SwiftData **[NEW]**
- `.modelContainer(previewContainer)` — inject a custom in-memory container for Xcode previews **[NEW]**

### Observation Framework **[NEW]**
- `@Observable` macro — grants Observable conformance to Swift classes; synthesized by `@Model` **[NEW]**
- `Observable` protocol — Swift's new observation protocol (no `@Published` required) **[NEW]**

### UniformTypeIdentifiers
- `UTType` — used to declare a custom document content type for SwiftData document packages
- `com.apple.package` conformance — required for SwiftData document types (packages, not flat files)

## Code Highlights

Minimal SwiftData model:
```swift
@Model
final class Card {
    var front: String
    var back: String
    var creationDate: Date

    init(front: String, back: String, creationDate: Date = .now) {
        self.front = front
        self.back = back
        self.creationDate = creationDate
    }
}
```

Querying and displaying models:
```swift
@Query private var cards: [Card]
// View updates automatically whenever any Card changes
```

Binding to a model from a view:
```swift
@Bindable var card: Card
// Use card.front and card.back directly in TextField
```

Scene-level container setup:
```swift
WindowGroup { ContentView() }
    .modelContainer(for: Card.self)
```

Inserting a new model (no explicit save needed):
```swift
@Environment(\.modelContext) private var modelContext

let newCard = Card(front: "Sample Front", back: "Sample Back")
modelContext.insert(object: newCard)
// SwiftData autosaves on next UI event
```

Document-based app scene:
```swift
DocumentGroup(editing: Card.self, contentType: .flashCards) {
    ContentView()
}
// No modelContainer setup needed — document infrastructure handles it
```

## Takeaways
- Adding `@Model` to an existing class is the entire model migration — no Core Data entity descriptions, no `NSManagedObject` subclass boilerplate.
- `@Query` replaces `@State` at query sites and provides automatic view updates; pair it with `.modelContainer(for:)` on the scene.
- SwiftData autosaves on UI events — only call `modelContext.save()` before sharing the underlying file.
- `DocumentGroup(editing:contentType:)` gives a full document-based app with open/save dialogs and keyboard shortcuts for free; declare a custom `UTType` conforming to `com.apple.package` for the document format.

---
_Source: WWDC23 Session 10154 page (abstract, chapter summaries, code samples, and resource links)._
