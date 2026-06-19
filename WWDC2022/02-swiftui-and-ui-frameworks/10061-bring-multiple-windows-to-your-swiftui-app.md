# Bring Multiple Windows to Your SwiftUI App
**WWDC22 · Session 10061** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10061/)

_Platforms:_ iPadOS 16, macOS Ventura 13

## Overview
SwiftUI gains powerful new scene and window management APIs in 2022. Two new scene types are introduced: `Window` (a single, unique window instance) and `MenuBarExtra` (a persistent macOS menu bar item with either a pulldown menu or a chromeless window). Together with new environment actions for programmatically opening windows and documents, and new scene modifiers for customizing window position, size, and keyboard shortcuts, this release dramatically expands what a SwiftUI multi-window app can do.

The session also clarifies the value types guidance for presented window values: prefer passing lightweight identifiers (not full model objects) to `openWindow`, as values must conform to `Hashable` and `Codable` for deduplication and state restoration.

## Key Topics

### New Scene Types
- `Window(title:id:)` — single unique window; appears once regardless of platform; ideal for global state that should not be duplicated
- `MenuBarExtra(label:systemImage:)` — macOS-only; persistent menu bar item; two rendering styles:
  - Default (`.menuBar`): pulldown menu anchored to menu bar
  - `.window`: chromeless window anchored to menu bar
- Can be used standalone or composed with other scenes in the same `App`

### Opening Windows Programmatically
- `openWindow` environment action — presents windows by scene `id` or by a typed value
  - `openWindow(id: "activity")` — opens a `Window` or `WindowGroup` by identifier
  - `openWindow(value: book.id)` — opens a `WindowGroup` matched by value type; if window exists for that value, brings it to front instead of creating a new one
- `newDocument` environment action — opens a new document window for `FileDocument` or `ReferenceFileDocument`
- `openDocument` environment action — opens an existing document by `URL` (`async throws`)
- Values passed to `openWindow(value:)` must conform to `Hashable + Codable`; prefer lightweight identifiers over full model objects for performance and state restoration

### WindowGroup with Presented Value
- `WindowGroup(title:for:) { $value in ... }` — new initializer accepting a type for the presented value **[NEW]**
- View builder receives a `Binding` to the presented value (optional)
- SwiftUI handles deduplication (same value = existing window brought to front) and state restoration (value persisted via `Codable`)

### Scene Customization Modifiers
- `.commandsRemoved()` — removes default File/Window menu commands for a scene **[NEW]**
- `.defaultPosition(_ position: UnitPoint)` — locale-aware initial window placement (e.g., `.topTrailing`) **[NEW]**
- `.defaultSize(width:height:)` — initial window size (layout system derives actual size) **[NEW]**
- `.keyboardShortcut(_:modifiers:)` on scenes — binds shortcut to the scene's window-opening command **[NEW/extended]**
- `.menuBarExtraStyle(_ style:)` — `.menu` (default) or `.window` **[NEW]**

## APIs & Frameworks

**SwiftUI — Scene Types** **[NEW]**
- `Window(_ title:id:content:)` **[NEW]** — single-instance window scene
- `MenuBarExtra(_ title:systemImage:content:)` **[NEW]** (macOS only)
- `MenuBarExtraStyle` — `.menu`, `.window` **[NEW]**

**SwiftUI — Environment Actions** **[NEW]**
- `OpenWindowAction` **[NEW]**
  - `openWindow(id:)` — by scene identifier
  - `openWindow(value:)` — by typed value (for `WindowGroup(for:)`)
  - Environment key: `\.openWindow`
- `NewDocumentAction` **[NEW]**
  - `newDocument(_ document:)` — opens a new document window
  - Environment key: `\.newDocument`
- `OpenDocumentAction` **[NEW]**
  - `openDocument(at url:) async throws` — opens existing document by URL
  - Environment key: `\.openDocument`

**SwiftUI — WindowGroup Additions**
- `WindowGroup(_ title:for: T.Type) { $value in ... }` **[NEW]** — value-driven WindowGroup; `T` must be `Hashable & Codable`

**SwiftUI — Scene Modifiers** **[NEW]**
- `.commandsRemoved()` — removes default menu commands from scene **[NEW]**
- `.defaultPosition(_ position: UnitPoint)` **[NEW]** — initial window position
- `.defaultSize(width:height:)` **[NEW]** — initial window size
- `.keyboardShortcut(_:modifiers:)` on `Scene` **[extended to scenes]**
- `.menuBarExtraStyle(_:)` **[NEW]**

**Existing Scene Types (for reference)**
- `WindowGroup` — multi-window data-driven apps
- `DocumentGroup` — document-based apps
- `Settings` — macOS settings window

## Code Highlights

New `Window` scene and `MenuBarExtra`:
```swift
@main struct BookClub: App {
    var body: some Scene {
        WindowGroup { ReadingListViewer(store: store) }
        Window("Activity", id: "activity") { ReadingActivity(store: store) }
        MenuBarExtra("Book Club", systemImage: "book") { AppMenu() }
    }
}
// Window-style MenuBarExtra:
MenuBarExtra("Time Tracker", systemImage: "rectangle.stack.fill") {
    TimeTrackerChart()
}.menuBarExtraStyle(.window)
```

Opening a window by value:
```swift
struct OpenWindowButton: View {
    var book: Book
    @Environment(\.openWindow) private var openWindow
    var body: some View {
        Button("Open In New Window") { openWindow(value: book.id) }
    }
}
// Matching WindowGroup in App:
WindowGroup("Book Details", for: Book.ID.self) { $bookId in
    BookDetail(id: $bookId, store: store)
}
```

Scene customization:
```swift
Window("Activity", id: "activity") { ReadingActivity(store: store) }
    .defaultPosition(.topTrailing)
    .defaultSize(width: 400, height: 800)
    .keyboardShortcut("0", modifiers: [.option, .command])

WindowGroup("Book Details", for: Book.ID.self) { $bookId in
    BookDetail(id: $bookId, store: store)
}.commandsRemoved()
```

## Takeaways
- `Window` provides a single-instance scene — ideal for global state like activity views, dashboards, or utility panels that should not be duplicated.
- `MenuBarExtra` makes it straightforward to build menu bar utility apps or augment existing apps with a menu bar presence, choosing between a pulldown menu or chromeless window style.
- `openWindow(value:)` deduplicates windows automatically — calling it with a value that already has an open window brings that window to front instead of creating a new one.
- Pass lightweight identifiers (not full model values) to `openWindow(value:)` — values must be `Codable` for state restoration and `Hashable` for deduplication; smaller values restore faster.

---
_Source: WWDC22 Session 10061 page (abstract, chapter summaries, code samples, and resource links)._
