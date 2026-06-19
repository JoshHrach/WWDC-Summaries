# Build Intelligent Siri Experiences with App Schemas
**WWDC26 · Session 240** · [Watch](https://developer.apple.com/videos/play/wwdc2026/240/)

_Platforms:_ iOS, iPadOS, macOS, watchOS, visionOS

## Overview
This is the primary introductory session for App Schemas and their role in bringing app content and actions to Siri and Apple Intelligence. Using the UnicornChat messaging sample, the session explains how App Entities model data, how App Schemas categorize that data into semantic domains Siri already understands, and how App Schema domains group related intents into coherent Siri capabilities — all without custom NLP or training phrases.

The session covers the full lifecycle from entity modeling and Spotlight semantic indexing through to cross-app content transfer with `Transferable` and `IntentValueRepresentation`. It also introduces best practices for designing complete Siri conversations (Xcode surfacing missing related schemas with Fix-Its) and a four-stage testing progression: AppIntentsTesting → Shortcuts app → Spotlight → Siri.

Apple Intelligence gains three new capabilities in the presented releases: accessing app entities, taking action through intents, and understanding onscreen context — all built on the same App Intents foundation.

## Key Topics

### What's New in Siri
Three new Siri capabilities, all built on App Intents: accessing your app's entities (content understanding), taking actions through your intents (natural language execution), and understanding onscreen context (awareness of what's visible). These extend across messaging, calendar, audio, and more domains.

### Contributing Content with App Entities
Model your app's content as `AppEntity`: what a thing is, how it's identified, and which properties matter. Conform to an App Schema so Siri understands the semantic category. For messaging, `MessageEntity` conforms to `.messages.message` and uses `@Property(indexingKey: \.textContent)` to mark searchable text.

### Entity Resolution and IndexedEntity
Siri resolves spoken references to real entities two ways:
1. **IndexedEntity** + `CSSearchableIndex` — semantic search; supports content Q&A over indexed fields via `@Property(indexingKey:)`
2. **EntityStringQuery** — live lookup for data that can't be pre-indexed (server-side, real-time)

### Making Actions Available
`AppIntent` conformances expose actions across Shortcuts, Spotlight, and Widgets. Conforming to an App Schema (e.g., `.messages.sendMessage`) makes the action executable by Siri via natural language. Grouping intents under a schema domain (`.messages`) makes the full conversation available to Apple Intelligence.

### Adopting a Schema Domain in UnicornChat
End-to-end walkthrough of adopting the Messages domain: mapping `sendMessage` schema parameters onto UnicornChat's messaging flow and returning the sent message as a `MessageEntity`. This lets Siri send messages without launching the app.

### Moving Content Across Apps
Export entities with `Transferable` + `IntentValueRepresentation` so other apps can act on them. On import, use `IntentValueQuery` to match existing content (e.g., match an `IntentPerson` to a local `Contact`) or `IntentValueRepresentation(importing:)` to create new content from the incoming value.

### Onscreen Awareness
Connect visible content to App Entities so Siri can resolve "this" and "that":
- `NSUserActivity` with `appEntityIdentifier` for a single primary item
- `.appEntityIdentifier(_:)` view modifier for one of many visible items

### Best Practices
Adopt complete schema sets — Xcode detects missing related schemas (e.g., `draftMessage` alongside `sendMessage`) at build time and offers Fix-Its. Design for full conversational flows, not just single-shot commands.

### Testing Your Integration
Progressive validation: **AppIntentsTesting** (business logic, no UI) → **Shortcuts app** (intent shape and parameter flow) → **Spotlight** (indexing and search) → **Siri** (end-to-end conversation).

## APIs & Frameworks

### AppIntents
- `@AppEntity(schema:)` — schema-based entity declaration **[NEW]**
- `AppEntity` protocol — `id`, `displayRepresentation`, `defaultQuery`
- `IndexedEntity` protocol — enables Spotlight semantic indexing
- `@Property(indexingKey:)` — marks a property as searchable in Spotlight **[NEW]**
- `TransientAppEntity` protocol — non-indexed relationship entities
- `EntityStringQuery` protocol — `entities(matching:)` live lookup
- `EntityQuery` protocol — `entities(for:)` by identifier
- `@AppIntent(schema:)` — schema-based intent declaration **[NEW]**
- `AppIntent` protocol — `perform()` method
- `OpenIntent` protocol — `target` parameter
- `IntentValueQuery` protocol — structured search; `values(for:)` returning matched entities **[NEW]**
- `IntentPerson` — system type representing a person; `displayName` property
- `IntentValueRepresentation` — **[NEW]** `Transferable` representation for cross-app entity export/import; `exporting:`, `importing:` closures
- `DisplayRepresentation` — `title:`, `subtitle:`, `image:` init
- `TypeDisplayRepresentation`
- `EntityIdentifier` — `init(for:identifier:)` for onscreen awareness
- `@Dependency` property wrapper
- `@Parameter` property wrapper
- **App Schema domains (messages):**
  - `.messages.message`
  - `.messages.sendMessage`
  - `.messages.draftMessage`
- **App Schema domains (system):**
  - `.system.open`
  - `.system.searchInApp`
- `AppIntentsTesting` framework — `IntentDefinitions`, `makeIntent(...)`, `.run()` **[NEW]**

### CoreSpotlight
- `CSSearchableIndex` — `indexAppEntities(_:)` method
- Semantic index — meaning-based search over `IndexedEntity` fields

### SwiftUI
- `.appEntityIdentifier(_:)` view modifier — annotate a view with an entity identifier **[NEW]**

### Foundation
- `NSUserActivity` — `appEntityIdentifier` property, `title` property
- `AttributedString` — used for message body content in `MessageEntity`

### SwiftData / Core Data
- `FetchDescriptor` — used in `EntityStringQuery` examples with `#Predicate`
- `ModelContext` — `.fetch(_:)` for entity lookup

### Transferable
- `Transferable` protocol — enables cross-app content sharing
- `TransferRepresentation` — base protocol
- `IntentValueRepresentation` — new representation type for structured intent values

## Code Highlights

**IndexedEntity with searchable property:**
```swift
@AppEntity(schema: .messages.message)
struct MessageEntity: IndexedEntity {
    @Property(indexingKey: \.textContent)
    var body: AttributedString?
}
```

**EntityStringQuery for live name lookup:**
```swift
struct ContactQuery: EntityStringQuery {
    func entities(matching string: String) async throws -> [ContactEntity] {
        let predicate = #Predicate<Person> { $0.name.localizedStandardContains(string) }
        return try modelContext.fetch(FetchDescriptor(predicate: predicate)).map(\.entity)
    }
}
```

**Cross-app export with IntentValueRepresentation:**
```swift
extension ContactEntity: Transferable {
    static var transferRepresentation: some TransferRepresentation {
        IntentValueRepresentation(exporting: \.person)
    }
}
```

**Cross-app import — create new content from incoming value:**
```swift
IntentValueRepresentation(exporting: \.person, importing: { intentPerson in
    let contact = Contact(importing: intentPerson)
    ContactManager.shared.contacts.append(contact)
    return contact.entity
})
```

**IntentValueQuery for cross-app entity matching:**
```swift
struct ContactEntityQuery: IntentValueQuery {
    func values(for input: [IntentPerson]) async throws -> [ContactEntity] {
        // Match incoming IntentPerson names against local contacts
    }
}
```

## Takeaways
- App Schemas are the primary on-ramp for Siri integration: conforming entities and intents to a schema domain gives Siri semantic understanding of your content and actions without any NLP configuration.
- `IndexedEntity` + `@Property(indexingKey:)` enables Siri to answer content questions ("what did Alex say about the meeting?") directly from your app's indexed data.
- `IntentValueRepresentation` closes the cross-app loop: export entities so other apps can act on them, and import `IntentPerson` / other system types to create or match local content.
- The four-stage test progression (AppIntentsTesting → Shortcuts → Spotlight → Siri) catches bugs at the lowest cost; start with AppIntentsTesting to validate business logic before involving the full Siri stack.

---
_Source: WWDC26 Session 240 page (abstract, chapter summaries, code samples, and resource links)._
