# Get to know App Intents

**Session ID:** 244  
**WWDC Year:** 2025  
**Folder:** `06-app-intents-siri-shortcuts`  
**URL:** https://developer.apple.com/videos/play/wwdc2025/244/

---

## Overview

This session is the foundational introduction to the App Intents framework, covering how to expose app functionality to Siri, the Shortcuts app, Spotlight, and Apple Intelligence. It walks through the core concepts — `AppIntent`, `AppEntity`, `AppEnum`, `EntityQuery` — explains the metadata-driven architecture that lets the system discover intents at compile time, and shows how to build a complete intent from scratch: defining parameters, returning results, providing a localized display name, and testing the intent in Shortcuts. It is the recommended starting point before watching the advances session (275).

> Note: Full transcript data for this session was not available at summary time; details are derived from session metadata, chapter list, and App Intents framework documentation.

---

## Key Topics

- App Intents philosophy: making app actions available system-wide without app launch
- `AppIntent` protocol: defining an action with `perform()` and metadata
- `AppEntity` protocol: representing app data types to the system
- `AppEnum` for finite-choice parameters
- `EntityQuery` for fetching entities by ID and by string search
- `IntentParameter` property wrappers for defining typed intent inputs
- `IntentResult` and `IntentDialog` for Siri voice responses
- Localization: `LocalizedStringResource` for all user-visible strings
- Testing intents in the Shortcuts app and Siri

---

## APIs & Frameworks

- **App Intents** framework (`import AppIntents`) – Core framework for surfacing app functionality to Siri, Shortcuts, and Apple Intelligence. Available on iOS 16+, macOS 13+, watchOS 9+.
- **`AppIntent`** – Core protocol; implement `title`, optional `description`, and `perform() async throws -> IntentResult`.
- **`@Parameter`** – Property wrapper for declaring intent inputs; takes `title:`, `description:`, optional `default:`, and type-specific options.
- **`IntentResult`** – Protocol for `perform()` return values. Common conformances: `.result()` (no output), `.result(value:)` (typed value for Shortcuts), `.result(value:dialog:)` (with Siri voice response).
- **`ReturnsValue`** / **`ProvidesDialog`** / **`ShowsSnippetView`** – Protocol compositions on `IntentResult` for indicating what the intent returns (value to Shortcuts, dialog to Siri, SwiftUI view in Siri UI).
- **`AppEntity`** – Protocol for app data types; requires `id: EntityID`, `displayRepresentation: DisplayRepresentation`, `typeDisplayRepresentation: TypeDisplayRepresentation`, and `defaultQuery: EntityQuery`.
- **`EntityQuery`** – Protocol for fetching `AppEntity` instances by ID: `func entities(for identifiers: [ID]) async throws -> [Entity]`. Optional: `suggestedEntities()` for Siri suggestions.
- **`StringSearchableEntityQuery`** – Protocol extending `EntityQuery` for free-text search; implement `entities(matching searchString: String) async throws -> [Entity]`.
- **`AppEnum`** – Protocol for finite-choice types used as intent parameters; requires `typeDisplayRepresentation` and `caseDisplayRepresentations`.
- **`LocalizedStringResource`** – Type for all user-visible strings in App Intents; enables automatic localization via `.strings` files.
- **`ShortcutsLink`** – SwiftUI view that opens the Shortcuts app focused on shortcuts for your app; use for in-app Shortcuts discovery onboarding.
- **`AppIntentsPackage`** – Protocol for declaring an App Intents module in a Swift package; required when intents live in a framework target.
- **`@main` `AppShortcutsProvider`** – **[NEW in iOS 26]** Protocol for pre-seeding Siri with specific phrases that invoke your intents, without user creation of a Shortcut.

---

## Code Highlights

Defining a simple App Intent:
```swift
import AppIntents

struct OpenFavoritesIntent: AppIntent {
    static var title: LocalizedStringResource = "Open Favorites"
    static var description = IntentDescription("Opens your Favorites list.")

    func perform() async throws -> some IntentResult {
        await MainActor.run { openFavoritesTab() }
        return .result()
    }
}
```

Defining an AppEntity and EntityQuery:
```swift
struct TaskEntity: AppEntity {
    static var typeDisplayRepresentation: TypeDisplayRepresentation = "Task"
    static var defaultQuery = TaskQuery()

    var id: UUID
    var title: String
    var isComplete: Bool

    var displayRepresentation: DisplayRepresentation {
        DisplayRepresentation(title: "\(title)")
    }
}

struct TaskQuery: EntityQuery {
    func entities(for identifiers: [UUID]) async throws -> [TaskEntity] {
        return try await TaskStore.shared.tasks(withIDs: identifiers)
    }

    func suggestedEntities() async throws -> [TaskEntity] {
        return try await TaskStore.shared.recentTasks(limit: 5)
    }
}
```

Intent with a parameter and dialog result:
```swift
struct CompleteTaskIntent: AppIntent {
    static var title: LocalizedStringResource = "Complete Task"

    @Parameter(title: "Task") var task: TaskEntity

    func perform() async throws -> some IntentResult & ProvidesDialog {
        try await TaskStore.shared.markComplete(task.id)
        return .result(dialog: "\(task.title) marked as complete.")
    }
}
```

---

## Takeaways

- App Intents uses a compile-time metadata extraction model: the framework statically analyzes your `AppIntent` types at build time, so intent registration is automatic — no manual registration code.
- `AppEntity` is the bridge between your app's data model and the system; once defined, entities can be used as parameters in multiple intents without repetition.
- `LocalizedStringResource` is mandatory for all user-facing strings; using plain `String` literals prevents localization and will produce warnings in Xcode.
- Pre-seeded Siri phrases via `AppShortcutsProvider` are the fastest path to Siri integration in iOS 26: users can invoke your intent by name without creating a Shortcut.
- Test intents incrementally: verify in Shortcuts editor first (parameter resolution, output values), then test voice in Siri.
- See session 275 ("Explore new advances in App Intents") for advanced topics: assistant schemas, property queries, Foundation Models integration, and control widget intents.
