# Dive into App Intents
**WWDC22 · Session 10032** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10032/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, tvOS 16, watchOS 9

## Overview
This session introduces App Intents, Apple's new Swift-first framework for exposing app functionality to the system — enabling Siri voice commands, Spotlight integration, the Shortcuts app, Focus filters, and more. Presented by Michael Gorbach from Shortcuts Engineering, the talk contrasts App Intents with the older SiriKit Intents framework and walks through every layer of the API: defining basic intents, adding parameters and enums, creating entities and queries (string, property), dialog and snippets for user feedback, and the two deployment models (in-app vs. extension).

The central design philosophy is "your code is the source of truth" — no separate definition files, no codegen editors. Swift language features (result builders, property wrappers, protocols, generics) make intents concise, modern, and maintainable. A simple intent can be written in a few lines; the API scales to deep, filter-driven entity queries with sorting, limits, and Core Data predicates.

## Key Topics

### Intents
- An `AppIntent` is a struct conforming to `AppIntent` with a `perform()` method.
- Required: `static var title: LocalizedStringResource` — localized name for Shortcuts editor.
- `openAppWhenRun: Bool` — set `true` for intents that open UI; defaults to `false` for background intents.
- `@MainActor` on `perform()` when app code requires the main thread.
- `IntentResult` returned from `perform()` can carry a value, dialog, snippet view, and an openIntent.

### App Shortcuts
- `AppShortcutsProvider` with `appShortcuts: [AppShortcut]` wraps intents for automatic Siri, Spotlight, and Shortcuts app discovery — no user setup required.
- `AppShortcut(intent:phrases:systemImageName:)` — define Siri trigger phrases using `\(.applicationName)`.

### Parameters and Enums
- `@Parameter(title:)` property wrapper on intent properties declares input parameters.
- Parameters support: `String`, `Int`, `Double`, `Bool`, `Date`, `File`, `URL`, `Measurement`, `Duration`, `AppEnum`, `AppEntity`.
- `AppEnum` protocol — enum with `String` raw value; provide `typeDisplayRepresentation` and `caseDisplayRepresentations` dictionary literal (read at build time).
- `ParameterSummary` result builder (`Summary("Open \(\.$shelf)")`) creates inline Shortcuts editor UI; always recommended.
- `When`/`Otherwise`, `Switch`/`Case`/`Default` builders vary summary based on parameter values.

### Entities and Queries
- `AppEntity` protocol — struct with stable `id: Identifiable`, `displayRepresentation: DisplayRepresentation`, `typeDisplayRepresentation`, and `static var defaultQuery`.
- `EntityQuery` — struct with `entities(for identifiers:)` required method.
- `EntityStringQuery` — adds `suggestedEntities()` and `entities(matching string:)` for search pickers.
- `EntityPropertyQuery` — adds `static var properties: QueryProperties` (with comparators) and `static var sortingOptions: SortingOptions`, plus `entities(matching comparators:mode:sortedBy:limit:)`.
- `QueryProperties` result builder with `Property(\Entity.$keyPath) { EqualToComparator {...} ContainsComparator {...} }`.
- `SortingOptions` result builder with `SortableBy(\Entity.$keyPath)`.
- `@Property(title:)` on entity fields exposes them for magic variables and Find/Filter actions in Shortcuts.

### User Interaction
- `IntentResult & ProvidesDialog` — return `.result(value:dialog:)` for spoken/textual feedback in Siri.
- `IntentResult & ShowsSnippetView` — return `.result(value:) { MySwiftUIView() }` for visual snippet.
- `$parameter.requestValue("Prompt")` — ask user for a missing/ambiguous parameter value; intent reruns with updated value.
- `$parameter.requestDisambiguation(among:dialog:)` — user picks from a list of options.
- `$parameter.requestConfirmation(for:dialog:)` — confirm a guessed parameter value.
- `requestConfirmation(output:)` on the intent — confirm a transactional result before executing.

### Return Values and openIntent
- `IntentResult & ReturnsValue<BookEntity>` — chains result to next intents as a magic variable.
- `OpensIntent` — `.result(value:openIntent: OpenBook(book: book))` adds "Open When Run" toggle in Shortcuts.

### Architecture
- In-app intents: no extension needed, higher memory limit, supports audio playback; app runs in background scene-less mode (implement scene support alongside).
- Extension: `App Intents Extension` target (File > New Target); lighter weight, no app launch required for Focus intents.
- Build-time metadata extraction: Xcode generates a metadata bundle from Swift compiler output — keep `AppIntent` types in the target/extension (not a framework); strings files must be in the same bundle.
- Migration: existing SiriKit `.intentdefinition` files have a "Convert to App Intent" button.

## APIs & Frameworks

### App Intents **[NEW]**
- `AppIntent` protocol — conformance + `perform()` **[NEW]**
- `LocalizedStringResource` — type-safe localized string for intent/parameter titles **[NEW]**
- `@Parameter(title:)` — intent parameter property wrapper **[NEW]**
- `AppEnum` protocol — enum as intent parameter type **[NEW]**
- `TypeDisplayRepresentation` — localized type name for enums and entities **[NEW]**
- `DisplayRepresentation` — localized display value (string, subtitle, image) **[NEW]**
- `ParameterSummary` result builder, `Summary(...)` **[NEW]**
- `AppEntity` protocol **[NEW]**
- `EntityQuery` protocol — `entities(for identifiers:)` **[NEW]**
- `EntityStringQuery` protocol — adds `suggestedEntities()` + `entities(matching string:)` **[NEW]**
- `EntityPropertyQuery` protocol — adds property-based filtering **[NEW]**
- `QueryProperties` result builder + `Property(\.$keyPath) { ... }` **[NEW]**
- `EqualToComparator`, `ContainsComparator`, `LessThanComparator`, `GreaterThanComparator` **[NEW]**
- `SortingOptions` result builder + `SortableBy(\.$keyPath)` **[NEW]**
- `@Property(title:)` — entity property exposure **[NEW]**
- `IntentResult` — result protocol for `perform()` return type **[NEW]**
- `ReturnsValue<T>` — result protocol for returning a typed value **[NEW]**
- `ProvidesDialog` — result protocol for spoken/textual feedback **[NEW]**
- `ShowsSnippetView` — result protocol for SwiftUI snippet **[NEW]**
- `OpensIntent` — result protocol enabling "Open When Run" **[NEW]**
- `AppShortcutsProvider` + `AppShortcut` **[NEW]**
- `CustomLocalizedStringResourceConvertible` — localize thrown errors **[NEW]**
- `$parameter.requestValue(_:)` — prompt user for value **[NEW]**
- `$parameter.requestDisambiguation(among:dialog:)` — disambiguation picker **[NEW]**
- `$parameter.requestConfirmation(for:dialog:)` — confirm guessed value **[NEW]**
- `requestConfirmation(output:)` — confirm transactional intent result **[NEW]**
- `ComparatorMode` (`.and` / `.or`) — predicate combination mode **[NEW]**
- `Sort<Entity>` — sort descriptor for property query **[NEW]**

## Code Highlights

```swift
// Minimal intent
struct OpenCurrentlyReading: AppIntent {
    static var title: LocalizedStringResource = "Open Currently Reading"
    static var openAppWhenRun: Bool = true

    @MainActor
    func perform() async throws -> some IntentResult {
        Navigator.shared.openShelf(.currentlyReading)
        return .result()
    }
}

// App Shortcut for Siri/Spotlight discovery
struct LibraryAppShortcuts: AppShortcutsProvider {
    static var appShortcuts: [AppShortcut] {
        AppShortcut(
            intent: OpenCurrentlyReading(),
            phrases: ["Open Currently Reading in \(.applicationName)"],
            systemImageName: "books.vertical.fill"
        )
    }
}

// Entity + EntityStringQuery
struct BookEntity: AppEntity, Identifiable {
    var id: UUID
    var displayRepresentation: DisplayRepresentation { "\(title)" }
    static var typeDisplayRepresentation: TypeDisplayRepresentation = "Book"
    static var defaultQuery = BookQuery()

    @Property(title: "Title") var title: String
    @Property(title: "Publishing Date") var datePublished: Date
    @Property(title: "Read Date") var dateRead: Date?
}

struct BookQuery: EntityStringQuery {
    func entities(for identifiers: [UUID]) async throws -> [BookEntity] {
        identifiers.compactMap { Database.shared.book(for: $0) }
    }
    func suggestedEntities() async throws -> [BookEntity] { Database.shared.books }
    func entities(matching string: String) async throws -> [BookEntity] {
        Database.shared.books.filter { $0.title.lowercased().contains(string.lowercased()) }
    }
}

// Property query with NSPredicate comparators
struct BookQuery: EntityPropertyQuery {
    static var sortingOptions = SortingOptions {
        SortableBy(\BookEntity.$title)
        SortableBy(\BookEntity.$dateRead)
        SortableBy(\BookEntity.$datePublished)
    }
    static var properties = QueryProperties {
        Property(\BookEntity.$title) {
            EqualToComparator { NSPredicate(format: "title = %@", $0) }
            ContainsComparator { NSPredicate(format: "title CONTAINS %@", $0) }
        }
        Property(\BookEntity.$datePublished) {
            LessThanComparator { NSPredicate(format: "datePublished < %@", $0 as NSDate) }
            GreaterThanComparator { NSPredicate(format: "datePublished > %@", $0 as NSDate) }
        }
    }
    func entities(matching comparators: [NSPredicate], mode: ComparatorMode,
                  sortedBy: [Sort<BookEntity>], limit: Int?) async throws -> [BookEntity] {
        Database.shared.findBooks(matching: comparators, matchAll: mode == .and,
            sorts: sortedBy.map { (keyPath: $0.by, ascending: $0.order == .ascending) })
    }
}

// Intent returning value + openIntent + dialog
struct AddBook: AppIntent {
    static var title: LocalizedStringResource = "Add Book"
    @Parameter(title: "Title") var title: String
    @Parameter(title: "Author Name") var authorName: String?

    func perform() async throws -> some IntentResult & ReturnsValue<BookEntity> & OpensIntent & ProvidesDialog {
        guard var book = await BooksAPI.shared.findBooks(named: title, author: authorName).first else {
            throw Error.notFound
        }
        Database.shared.add(book: book)
        return .result(value: book, dialog: "Added \(book) to Library!", openIntent: OpenBook(book: book))
    }
}
```

## Takeaways
- App Intents replaces custom SiriKit Intents for Siri/Shortcuts; use the "Convert to App Intent" button in existing `.intentdefinition` files to migrate.
- Keep `AppIntent` types directly in the app target or extension — not a shared framework — so Xcode's build-time metadata extraction works correctly.
- `EntityPropertyQuery` with `@Property` fields unlocks automatic Find/Filter actions in Shortcuts with a predicate editor UI; users can do things like "find the three most recently published books by a given author" without any additional code.
- `OpensIntent` and `openAppWhenRun` together give the "Open When Run" toggle in Shortcuts, allowing background-only execution when desired.
- For intents that might fail or need clarification, use `requestValue`, `requestDisambiguation`, and `requestConfirmation` — they pause and resume the intent with user input, enabling robust voice workflows.

---
_Source: WWDC22 Session 10032 page (transcript, code samples, and resource links)._
