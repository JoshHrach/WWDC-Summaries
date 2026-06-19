# Data Essentials in SwiftUI
**WWDC20 · Session 10040** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10040/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, watchOS 7, tvOS 14

## Overview
This session provides a comprehensive guide to data flow and state management in SwiftUI, covering the full progression from simple view-local state to app-wide observable models and persistent storage. Three SwiftUI engineers walk through the "three questions" every view designer should ask — what data the view needs, how it manipulates that data, and where the data comes from — and map each answer to the right tool: `@State`, `@Binding`, `@ObservedObject`, `@StateObject` (new), `@EnvironmentObject`, `@SceneStorage` (new), and `@AppStorage` (new).

The session introduces `@StateObject` as the solution to a subtle but important correctness and performance bug: instantiating an `ObservableObject` inline inside a `View` body with `@ObservedObject` causes repeated heap allocations and data loss. `@StateObject` fixes this by having SwiftUI own and persist the object's lifetime alongside the view. Two other new storage types, `@SceneStorage` and `@AppStorage`, allow lightweight UI state to survive process termination by persisting per-scene and globally via `UserDefaults` respectively.

## Key Topics

**The Three Questions**
When adding any view in SwiftUI: (1) What data does this view need? (2) How will it manipulate that data? (3) Where does the data come from? The answer to question three determines the correct property wrapper.

**`@State` and Value Types**
Use `@State` for view-local, transient data owned by the view itself. SwiftUI manages storage across re-renders. Encapsulating multiple related state properties into a value type `struct` with `mutating` methods improves encapsulation and testability.

**`@Binding`**
Use `@Binding` to share write access to a source of truth owned elsewhere. Get a binding from a `@State` property using the `$` prefix. Bindings compose: you can derive a `Binding` to a nested property from an existing `Binding` (`$editorConfig.note`). Bindings are agnostic to their source of truth, making them ideal for reusable components.

**`ObservableObject` and `@Published`**
For data models that outlive views, classes conform to `ObservableObject`. Mark mutable properties with `@Published` to automatically trigger view invalidation before any mutation. The `objectWillChange` publisher (a `PassthroughSubject` by default) can be replaced with a custom publisher (timer, KVO, etc.).

**`@ObservedObject`**
Declares a dependency on an external `ObservableObject`. Does not own the object's lifetime — the caller is responsible for keeping the object alive. SwiftUI subscribes to `objectWillChange` and re-renders all dependent views.

**`@StateObject` (New in iOS 14)**
SwiftUI owns and persists the `ObservableObject` for the entire view lifetime. Instantiated once, before the first `body` run. Prevents repeated heap allocations and data loss from the anti-pattern of `@ObservedObject var store = MyStore()` inline in a view. Use `@StateObject` as the preferred view-owned source of truth for reference-type models.

**`@EnvironmentObject`**
Injects an `ObservableObject` into the environment via `.environmentObject(_:)` on a parent view; reads it via `@EnvironmentObject` in any descendant, without passing it explicitly through the view hierarchy. Only creates a dependency where the value is actually read.

**`@SceneStorage` (New in iOS 14)**
Per-scene persistent storage, automatically saved and restored by SwiftUI. Use for lightweight UI state like current selection. Key must be unique per type of data stored within the scene. Acts as a source of truth (bindings work with it).

**`@AppStorage` (New in iOS 14)**
Global app-scoped persistent storage backed by `UserDefaults`. Readable from anywhere (views, app struct, etc.). Use for small settings and preferences. Bindings work, so it integrates naturally with `Toggle` and other controls.

**Performance: Avoiding Slow Updates**
View `body` must be a pure function: no dispatching, no side effects, just view composition. Expensive work should be dispatched to background queues from event handlers (`onChange`, `onReceive`, etc.), not from `body`. Never instantiate an `ObservableObject` inline in `body` with `@ObservedObject` — use `@StateObject` instead.

**Lifetime Scoping**
- View-lifetime: `@State`, `@StateObject`
- Scene-lifetime: `@SceneStorage`, or `@StateObject` hung off the scene root
- App-lifetime: `@StateObject` in the `App` struct, `@AppStorage`

## APIs & Frameworks

### SwiftUI — Property Wrappers
- `@State` — view-local value-type storage managed by SwiftUI
- `@Binding` — read-write reference to any source of truth
- `@ObservedObject` — dependency on external `ObservableObject`; does not own lifetime
- `@StateObject` **[NEW]** — view-owned `ObservableObject`; SwiftUI manages instantiation and lifetime
- `@EnvironmentObject` — reads injected `ObservableObject` from environment
- `@SceneStorage` **[NEW]** — per-scene persistent storage (lightweight UI state)
- `@AppStorage` **[NEW]** — app-global persistent storage backed by `UserDefaults`
- `@Published` — marks `ObservableObject` properties for automatic invalidation

### SwiftUI — Protocols & Types
- `ObservableObject` — class protocol enabling SwiftUI dependency tracking
- `ObservableObject.objectWillChange` — `Publisher` that fires before mutations
- `View.environmentObject(_:)` — injects `ObservableObject` into environment
- `App` protocol — new in SwiftUI; supports `@StateObject` for app-wide sources of truth

### SwiftUI — View Modifiers (Event Sources, New in iOS 14)
- `View.onChange(of:perform:)` **[NEW]** — reacts to value changes
- `View.onReceive(_:perform:)` — reacts to publisher emissions
- `View.onOpenURL(_:)` **[NEW]** — handles incoming URLs
- `View.onContinueUserActivity(_:perform:)` **[NEW]** — handles `NSUserActivity`

### UIKit / Foundation
- `UserDefaults` — backing store for `@AppStorage`
- `TextEditor` **[NEW]** — multi-line text editing control accepting a `Binding<String>`

## Code Highlights

View-local state with a struct encapsulating related properties:
```swift
struct EditorConfig {
    var isEditorPresented = false
    var note = ""
    var progress: Double = 0
    mutating func present(initialProgress: Double) {
        progress = initialProgress
        note = ""
        isEditorPresented = true
    }
}
struct BookView: View {
    @State private var editorConfig = EditorConfig()
    var body: some View {
        Button(action: { editorConfig.present(initialProgress: currentProgress) }) { … }
        ProgressEditor(editorConfig: $editorConfig)
    }
}
```

`@StateObject` as the correct replacement for inline `@ObservedObject` initialization:
```swift
// WRONG — causes repeated heap allocations and data loss:
struct ReadingList: View {
    @ObservedObject var store = ReadingListStore()
    …
}

// CORRECT — SwiftUI owns and persists the object:
struct ReadingList: View {
    @StateObject var store = ReadingListStore()
    …
}
```

Deriving a `Binding` to a nested property on an `ObservableObject`:
```swift
Toggle(isOn: $currentlyReading.isFinished) { Label("I'm Done", …) }
```

`@SceneStorage` for per-scene persistent UI state:
```swift
struct ReadingListViewer: View {
    @SceneStorage("selection") var selection: String?
    var body: some View {
        NavigationView {
            ReadingList(selection: $selection)
            BookDetailPlaceholder()
        }
    }
}
```

`@AppStorage` for global settings backed by `UserDefaults`:
```swift
struct BookClubSettings: View {
    @AppStorage("updateArtwork") private var updateArtwork = true
    @AppStorage("syncProgress") private var syncProgress = true
    var body: some View {
        Form {
            Toggle(isOn: $updateArtwork) { … }
            Toggle(isOn: $syncProgress) { … }
        }
    }
}
```

## Takeaways
- `@StateObject` (new) is the correct way to create a view-owned `ObservableObject`; never use `@ObservedObject var store = MyStore()` inline, as it causes repeated heap allocations and data loss.
- `@SceneStorage` and `@AppStorage` (both new) provide lightweight, automatic UI state persistence without manual `UserDefaults` code, surviving process termination and device restarts.
- A typical app uses multiple property wrappers together: `@State`/`@StateObject` for owned data, `@Binding` for shared write access, `@EnvironmentObject` for deeply nested sharing, and `@SceneStorage`/`@AppStorage` for persistence.
- View `body` must remain a pure function; all expensive work belongs in background queues triggered by event handlers, not in `body` itself.

---
_Source: WWDC20 Session 10040 page (abstract, chapter summaries, code samples, and resource links)._
