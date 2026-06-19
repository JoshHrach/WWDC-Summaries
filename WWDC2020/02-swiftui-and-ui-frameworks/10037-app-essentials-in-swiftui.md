# App Essentials in SwiftUI
**WWDC20 · Session 10037** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10037/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, watchOS 7, tvOS 14

## Overview
SwiftUI 2 extends the framework beyond views to support declaring entire app life cycles. Three new protocol types — `App`, `Scene`, and the existing `View` — form a unified ownership hierarchy: an `App` contains one or more `Scene` instances, and each `Scene` contains a `View` tree. This means a complete, production-quality app can now be written entirely in SwiftUI with no UIApplicationDelegate, NSApplicationDelegate, or AppDelegate boilerplate.

The session introduces `WindowGroup` as the primary scene type for data-driven apps, which automatically manages multi-window support on iPadOS and macOS (including tabbed windows), populates navigation titles in the window title bar and App Switcher, and enables Command-N for new windows on Mac. New scene types for document-based and settings workflows are also introduced, along with a declarative commands API for adding custom menu items and keyboard shortcuts.

## Key Topics
- **`App` protocol** **[NEW]** — Struct conforming to `App` with a `body: some Scene` property; decorated with `@main` to serve as the program entry point (requires Swift 5.3). Replaces `main.swift` and all delegate boilerplate.
- **`Scene` protocol** **[NEW]** — Fundamental unit that the platform uses to display UI in windows, tabs, or full-screen regions; platform controls life cycle, not the app.
- **`WindowGroup`** **[NEW]** — Scene type for primary app UI; automatically creates multiple independent windows on iPadOS and macOS, merges windows into tabs (macOS), and shows a File > New Window menu item (Command-N). Each window has independent view state while sharing a common model.
- **`@StateObject`** **[NEW]** — Property wrapper declaring that the `App` (or any view) owns an `ObservableObject`; object is created once and persists for the lifetime of the declaring type.
- **`@SceneStorage`** **[NEW]** — Property wrapper for per-scene state restoration; takes a unique string key; SwiftUI automatically saves and restores the value at appropriate times.
- **`.navigationTitle(_:)`** **[NEW]** — View modifier; on iOS populates the navigation bar title and App Switcher label; on macOS sets the window title bar text and Windows menu entry.
- **`DocumentGroup`** **[NEW]** — Scene type for document-based apps; automatically handles open/edit/save/create document workflows; uses `FileDocument` or `ReferenceFileDocument` conformance.
- **`Settings`** **[NEW]** — Scene type (macOS only); automatically sets up the Preferences window, adds it to the app menu, and applies the correct window style.
- **`Commands` + `.commands(_:)` modifier** **[NEW]** — Declarative API for adding custom menu commands to scenes; uses `CommandMenu`, `CommandGroup`, `Button`, and `@FocusedBinding` for focus-based targeting (analogous to UIKit/AppKit responder chain).
- **`@FocusedBinding`** **[NEW]** — Property wrapper used inside `Commands` to read and write a value from the currently focused scene window.
- **`@SceneBuilder`** **[NEW]** — Result builder for composing multiple scene types in an `App` body.

## APIs & Frameworks

### SwiftUI
- **`App`** **[NEW]** — Protocol; `var body: some Scene { get }`; used with `@main`
- **`Scene`** **[NEW]** — Protocol; base type for all scene values
- **`WindowGroup<Content: View>`** **[NEW]** — `init(content: () -> Content)`; `init(_ title: StringProtocol, content: () -> Content)` (macOS)
- **`DocumentGroup`** **[NEW]** — `init(newDocument:editor:)` (new doc); `init(viewing:viewer:)` (read-only); content block receives `FileDocumentConfiguration` or `ReferenceFileDocumentConfiguration`
- **`FileDocument`** **[NEW]** — Protocol: `static var readableContentTypes: [UTType]`, `init(configuration:) throws`, `fileWrapper(configuration:) throws -> FileWrapper`
- **`Settings`** **[NEW]** — `init(content: () -> Content)`; macOS only
- **`@StateObject`** **[NEW]** — `@propertyWrapper struct StateObject<ObjectType: ObservableObject>`
- **`@SceneStorage`** **[NEW]** — `@propertyWrapper struct SceneStorage<Value>`; `init(_ key: String)`; stores `Bool`, `Int`, `Double`, `String`, `URL`, `Data`, or `RawRepresentable`
- **`.navigationTitle(_:)`** **[NEW]** — `func navigationTitle(_ title: LocalizedStringKey) -> some View`
- **`Commands`** **[NEW]** — Protocol; `var body: some Commands { get }`
- **`CommandMenu`** **[NEW]** — `init(_ name: LocalizedStringKey, content: () -> Content)`; adds a top-level menu
- **`CommandGroup`** **[NEW]** — `init(replacing:addition:)` or `init(after:addition:)` / `init(before:addition:)`; inserts into system menus
- **`.commands(_:)`** **[NEW]** — Scene modifier: `func commands<C: Commands>(content: () -> C) -> some Scene`
- **`@FocusedBinding`** **[NEW]** — `@propertyWrapper struct FocusedBinding<Value>`; reads focus-dependent binding from the active window
- **`@SceneBuilder`** **[NEW]** — Result builder for composing heterogeneous scene types

## Code Highlights

Minimal complete app entry point with shared model:
```swift
@main
struct BookClubApp: App {
    @StateObject private var store = ReadingListStore()

    var body: some Scene {
        WindowGroup {
            ReadingListViewer(store: store)
        }
    }
}
```

Per-scene state restoration with `@SceneStorage`:
```swift
struct ReadingListViewer: View {
    @SceneStorage("selectedItem") private var selectedItem: String?
    // selectedItem is saved/restored automatically per window
}
```

Document-based app with `DocumentGroup`:
```swift
@main
struct ShapeEditApp: App {
    var body: some Scene {
        DocumentGroup(newDocument: ShapeDocument()) { file in
            DocumentView(document: file.$document)
        }
    }
}
```

Settings window and custom commands on macOS:
```swift
@SceneBuilder var body: some Scene {
    WindowGroup { ContentView() }
    #if os(macOS)
    Settings { AppSettingsView() }
    #endif
}
```

Custom menu commands with focus-based targeting:
```swift
struct BookCommands: Commands {
    @FocusedBinding(\.selectedBook) private var selectedBook: Book?

    var body: some Commands {
        CommandMenu("Book") {
            Button("Update Progress…") { selectedBook?.updateProgress() }
                .keyboardShortcut("u")
                .disabled(selectedBook == nil)
        }
    }
}
```

## Takeaways
- The `App` protocol + `@main` replaces all delegate boilerplate; a complete app now fits in a handful of lines of declarative SwiftUI code.
- `WindowGroup` automatically handles multi-window UIs on iPadOS and macOS — including tab merging, the Windows menu, and Command-N — with zero additional code.
- `@StateObject` declares model ownership at the `App` level so the store is created once and shared across all scene windows via the environment or explicit passing.
- `@SceneStorage` provides per-window state restoration with a simple key — SwiftUI handles the save/restore timing automatically, keeping each window's navigation state independent.
- Use `Settings` for the macOS preferences window and `DocumentGroup` for document-based flows; both wire up system menus automatically.

---
_Source: WWDC20 Session 10037 page (abstract, chapter summaries, code samples, and resource links)._
