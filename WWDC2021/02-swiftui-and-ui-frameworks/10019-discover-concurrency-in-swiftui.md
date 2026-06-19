# Discover concurrency in SwiftUI
**WWDC21 · Session 10019** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10019/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, tvOS 15, watchOS 8

## Overview
Swift 5.5's `async`/`await` concurrency model integrates directly with SwiftUI's data flow. This session explains how `ObservableObject` interacts with the SwiftUI main-actor run loop, why dispatching to a background queue for model updates is unsafe, and how `await` is the correct replacement. It also introduces three new SwiftUI APIs built on top of Swift concurrency: `AsyncImage` for declarative remote image loading, the `.task` view modifier for lifetime-scoped async work, and `.refreshable` for pull-to-refresh with automatic spinner dismissal.

## Key Topics

### SwiftUI Run Loop and the Main Actor
- The SwiftUI update lifecycle ("run loop") runs on the **main actor** in Swift 5.5.
- `ObservableObject` state changes must happen in a specific order: `objectWillChange` fires → state is written → run loop ticks on the next cycle.
- Dispatching to a background queue breaks this ordering: the state change and run loop tick can interleave, causing SwiftUI to miss the update and not re-render views.
- The fix: use `await` from the main actor. `await` *yields* the main actor back to SwiftUI during long I/O, so the run loop keeps ticking; when the async work completes, Swift re-enters the method on the main actor, guaranteeing correct ordering.

### `@MainActor` on ObservableObject **[NEW]**
- Annotating a class with `@MainActor` instructs the Swift compiler to enforce that all properties and methods are accessed only from the main actor.
- Eliminates the entire category of off-main-thread `@Published` mutation bugs without manual queue management.
- Pattern: mark the `ObservableObject` class `@MainActor`; make data-fetching methods `async`; use `await` on async calls within those methods.

### `.task` View Modifier **[NEW in SwiftUI]**
- Attaches an asynchronous task to a view's lifetime.
- The task starts when the view appears and is **automatically cancelled** when the view disappears.
- Async by default: call `await` on any async method directly in the closure.
- Replaces common `onAppear { Task { … } }` patterns with built-in lifetime management.
- Also works with `AsyncSequence`—the task loops over values and is cancelled when the view leaves the hierarchy.

### `AsyncImage` **[NEW in SwiftUI]**
- Loads and displays a remote image from a URL asynchronously with no manual URL session code.
- Simple form: `AsyncImage(url: photo.url)` — shows a placeholder until loaded.
- Customizable form: `AsyncImage(url:) { image in … } placeholder: { … }` — supply an `Image` modifier closure and a `View` placeholder.
- Phase-based form (`.phase`): full control over loading, success, and failure states.
- All network and memory work happens off the main thread automatically; no extra configuration required.

### `.refreshable` Modifier **[NEW in SwiftUI]**
- Adds pull-to-refresh to any `List` or `ScrollView`.
- Takes an `async` closure; the system spinner dismisses automatically when the closure's `await` calls complete.
- Replaces `UIRefreshControl` delegation patterns with a single modifier.

### Button Actions with `Task` Wrapper
- Button `action` closures are synchronous; to call an `async` method, wrap it in `Task { await … }`.
- Pattern for async button actions: set `@State` loading flag to `true`, `await` the async call, set flag to `false`—all on the main actor, so SwiftUI sees each state change.

## APIs & Frameworks

**SwiftUI**
- `@MainActor` on `ObservableObject` — compiler-enforced main-actor confinement **[NEW via Swift 5.5]**
- `.task { }` view modifier — async task tied to view lifetime, auto-cancelled on disappear **[NEW]**
- `AsyncImage(url:)` — declarative remote image loading **[NEW]**
- `AsyncImage(url:content:placeholder:)` — customizable image + placeholder variant **[NEW]**
- `AsyncImage(url:transaction:content:)` (phase-based) — full loading state control **[NEW]**
- `.refreshable { await … }` — pull-to-refresh with async closure **[NEW]**
- `.listRowSeparator(.hidden)` — hide row separators in List **[NEW]**
- `.listStyle(.plain)` — static-member enum-like style syntax **[NEW spelling, Xcode 13]**

**Swift Concurrency**
- `async` / `await` — non-blocking suspension points that yield the current actor **[NEW via Swift 5.5]**
- `@MainActor` — attribute that confines code to the main actor **[NEW via Swift 5.5]**
- `Task { }` — unstructured task for calling async code from sync contexts **[NEW via Swift 5.5]**
- `URLSession.shared.data(from:)` async overload — network fetch without callbacks **[NEW]**

## Code Highlights

Observable object confined to the main actor with async data fetching:
```swift
@MainActor
class Photos: ObservableObject {
    @Published private(set) var items: [SpacePhoto] = []

    func updateItems() async {
        let fetched = await fetchPhotos()
        items = fetched          // safe: back on main actor after await
    }

    func fetchPhotos() async -> [SpacePhoto] {
        var downloaded: [SpacePhoto] = []
        for date in randomPhotoDates() {
            let url = SpacePhoto.requestFor(date: date)
            if let photo = await fetchPhoto(from: url) {
                downloaded.append(photo)
            }
        }
        return downloaded
    }

    func fetchPhoto(from url: URL) async -> SpacePhoto? {
        do {
            let (data, _) = try await URLSession.shared.data(from: url)
            return try SpacePhoto(data: data)
        } catch {
            return nil
        }
    }
}
```

Using `.task` and `.refreshable` in a List view:
```swift
struct CatalogView: View {
    @StateObject private var photos = Photos()

    var body: some View {
        NavigationView {
            List {
                ForEach(photos.items) { item in
                    PhotoView(photo: item)
                        .listRowSeparator(.hidden)
                }
            }
            .listStyle(.plain)
            .refreshable {
                await photos.updateItems()      // spinner auto-dismisses when done
            }
        }
        .task {
            await photos.updateItems()          // runs on appear, cancelled on disappear
        }
    }
}
```

`AsyncImage` with custom content and placeholder:
```swift
AsyncImage(url: photo.url) { image in
    image
        .resizable()
        .aspectRatio(contentMode: .fill)
} placeholder: {
    ProgressView()
}
.frame(minWidth: 0, minHeight: 400)
```

Async save button with loading state:
```swift
struct SavePhotoButton: View {
    var photo: SpacePhoto
    @State private var isSaving = false

    var body: some View {
        Button {
            Task {
                isSaving = true
                await photo.save()
                isSaving = false
            }
        } label: {
            Text("Save")
                .opacity(isSaving ? 0 : 1)
                .overlay { if isSaving { ProgressView() } }
        }
        .disabled(isSaving)
    }
}
```

## Takeaways
- Mark `ObservableObject` classes `@MainActor` so the compiler guarantees updates never race with the SwiftUI run loop; use `await` inside those methods for all async I/O.
- `Task { await … }` in synchronous Button actions is the correct pattern for triggering async work from user events.
- `AsyncImage` eliminates all the boilerplate for remote image loading; use the phase-based overload for custom error states.
- `.task { await … }` is the preferred replacement for `onAppear + Task`: the task is automatically cancelled when the view disappears, preventing stale work and memory leaks.
- `.refreshable { await … }` adds pull-to-refresh with zero UIKit boilerplate; the spinner dismisses automatically when the async closure returns.

---
_Source: WWDC21 Session 10019 page (abstract, full transcript, code samples, and resource links)._
