# What's New in SwiftUI
**WWDC26 · Session 269** · [Watch](https://developer.apple.com/videos/play/wwdc2026/269/)

_Platforms:_ iOS, iPadOS, macOS, watchOS, visionOS, tvOS

## Overview
This session surveys the major additions to SwiftUI landing with the 2027 OS releases. It opens by describing how apps automatically receive the new Liquid Glass appearance when built against the latest SDK, then dives into entirely new APIs across five themes: refreshed look and feel, document-based apps, presentation and interaction, and data flow and performance.

The document story is the largest area of change. A new `WritableDocument` / `ReadableDocument` protocol pair replaces the older `FileDocument` system for high-performance apps, providing direct disk access via URLs, snapshot-based diffing, and progress-reporting writers. The session also covers new reordering APIs that make drag-to-reorder available inside any container — including `LazyVGrid` and watchOS — and expands swipe actions beyond `List` to arbitrary scroll views.

On the performance side, `@State` is now implemented as a macro that lazily initializes class instances, eliminating redundant allocations. `AsyncImage` gains standard HTTP-caching semantics and a new API for supplying a custom `URLSession` and `URLRequest`. A new `@ContentBuilder` function-builder rounds out the additions.

## Key Topics

### Refreshed Look and Feel (2:12)
- Apps adopt Liquid Glass automatically; interactive glass elements provide physical-feeling feedback.
- New `appearsActive` environment value lets views adjust opacity or style when a window becomes inactive (demonstrated on iPadOS multi-window).
- `Tab(role: .prominent)` pins a tab so it remains visible when the tab bar collapses during scroll.
- Toolbar gains `visibilityPriority`, `ToolbarOverflowMenu`, and `topBarPinnedTrailing` placement.
- `toolbarMinimizeBehavior(.onScrollDown, for: .navigationBar)` slides the navigation bar away while scrolling.

### Document-Based Apps (8:06)
- `DocumentCreationSource` and a new `NewDocumentButton(source:)` allow custom new-document flows (e.g., "New from Photo").
- `WritableDocument` protocol: implement `snapshot(contentType:)` returning a `sending` value type snapshot, and a `DocumentWriter` that writes that snapshot to a URL — enabling background writes and `Subprogress` reporting.
- `ReadableDocument` protocol for the read side; supports multiple export formats by adding types to `writableContentTypes`.

### Presentation and Interaction (15:18)
- `reorderable()` + `reorderContainer(for:in:)` modifiers enable drag-to-reorder in `List`, `LazyVGrid`, and on watchOS for the first time. Uses `ReorderDifference` with `OrderedDictionary` (Swift Collections).
- `swipeActions` now works on any view inside a `swipeActionsContainer()`, not just inside `List`.
- `confirmationDialog` and `alert` gain an `item:` binding overload that automatically dismisses when the binding becomes `nil`.

### Data Flow and Performance (19:58)
- `AsyncImage` now reads from `URLCache` by default; use `.asyncImageURLSession(_:)` to supply a custom session and `URLRequest(cachePolicy: .returnCacheDataElseLoad)`.
- `@State` becomes a Swift macro: class instances used as state are now lazily initialized and live only once for the view's lifetime. Back-ported to iOS 17 / macOS 14.
- `@ContentBuilder` result-builder for composing heterogeneous content in functions.

## APIs & Frameworks

**SwiftUI**
- **[NEW]** `WritableDocument` protocol (replaces `FileDocument` for high-perf use cases)
- **[NEW]** `ReadableDocument` protocol
- **[NEW]** `DocumentWriter` protocol with `write(snapshot:to:previous:progress:)`
- **[NEW]** `DocumentCreationSource` / `NewDocumentButton(source:)`
- **[NEW]** `.reorderable()` view modifier
- **[NEW]** `.reorderContainer(for:in:)` view modifier
- **[NEW]** `ReorderDifference<ItemID, CollectionID>` type
- **[NEW]** `.swipeActionsContainer()` modifier (extends swipe actions beyond `List`)
- **[NEW]** `.confirmationDialog(_:item:)` and `.alert(_:item:)` item-binding overloads
- **[NEW]** `ToolbarOverflowMenu`
- **[NEW]** `ToolbarItemGroup.visibilityPriority(_:)`
- **[NEW]** `.topBarPinnedTrailing` toolbar placement
- **[NEW]** `toolbarMinimizeBehavior(_:for:)` modifier
- **[NEW]** `Tab(role: .prominent)` tab role
- **[NEW]** `\.appearsActive` environment value
- **[NEW]** `.asyncImageURLSession(_:)` modifier
- **[NEW]** `@ContentBuilder` result builder
- **[CHANGED]** `@State` — converted to a macro; lazy initialization for Observable class types
- `DocumentGroup`, `DocumentGroupLaunchScene` (updated with `context` parameter)
- `AsyncImage(request:)` initializer (now with caching)
- `URLRequest(url:cachePolicy:)`
- `URLSessionConfiguration`, `URLCache`
- `Subprogress` (Swift Structured Concurrency progress)

**Swift Collections (github.com/apple/swift-collections)**
- `OrderedDictionary` — used with `ReorderDifference.apply(to:)`

**UniformTypeIdentifiers**
- `UTType` — custom document type declaration with `exportedAs:`

## Code Highlights

Minimize toolbar on scroll:
```swift
ScrollView { StickerListView() }
    .toolbarMinimizeBehavior(.onScrollDown, for: .navigationBar)
```

Reorderable grid:
```swift
LazyVGrid {
    ForEach(stickers) { sticker in StickerListItemView(sticker: sticker) }
    .reorderable()
}
.reorderContainer(for: Sticker.self) { difference in
    difference.apply(to: &stickers)
}
```

Swipe actions on any scroll view:
```swift
ScrollView {
    LazyVStack {
        ForEach(stickers) { sticker in
            StickerListItemView(sticker: sticker)
                .swipeActions { DeleteButton(sticker: sticker) }
        }
    }
}
.swipeActionsContainer()
```

Lazy `@State` (Observable class):
```swift
@State private var store = StickerStore() // lazily initialized once
```

## Takeaways
- Build against the iOS/macOS 27 SDK with Xcode 27 to automatically pick up the Liquid Glass appearance; no code changes required.
- Migrate document-based apps to `WritableDocument` / `ReadableDocument` for direct URL access and snapshot-based diffing, especially for large or complex documents.
- Replace List-only reorder and swipe-action patterns with the new container APIs to support grids and arbitrary scroll views.
- Adopt the lazy `@State` macro pattern for Observable model objects — it's back-ported to iOS 17/macOS 14 and eliminates double-allocation bugs.

---
_Source: WWDC26 Session 269 page (abstract, chapter summaries, code samples, and resource links)._
