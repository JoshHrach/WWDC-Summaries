# Code-along: Make your App Available to Siri
**WWDC26 · Session 344** · [Watch](https://developer.apple.com/videos/play/wwdc2026/344/)

_Platforms:_ iOS, iPadOS, macOS, watchOS

## Overview
This hands-on code-along walks through adopting App Schemas in the CometCal sample calendar app to make it fully available to Siri and Apple Intelligence. Starting from scratch, the session builds up three schematized entities — `CalendarEntity`, `AttendeeEntity`, and `EventEntity` — covering the calendar domain and demonstrating how Siri understands content and performs natural language actions without any training phrases.

The session covers all four CRUD intents for events (create, read via open, update, delete), Spotlight semantic indexing, onscreen awareness via view modifiers, and custom SwiftUI snippet views for Siri responses. By the end, Siri can search events, update times with conversational confirmation, and text attendees, all driven purely by App Intents schema conformances.

The CometCal sample project is available for download and accompanied by a full test suite using the new AppIntentsTesting framework. The session pairs closely with "Explore advanced App Intents features for Siri and Apple Intelligence" (Session 343) for polishing the integration.

## Key Topics

### App Schemas and the Plan
App Schemas describe content and actions in terms Siri already understands — no custom NLP or training phrases required. Schemas are grouped into domains (here, the `calendar` domain). The session's goals are two-fold: make content discoverable and make actions executable.

### Building CalendarEntity
Conform to `@AppEntity` with `schema: .calendar.calendar`, set `id` to `UUID`, adopt `IndexedEntity`, wire a `@Dependency` and a `@MainActor` `EnumerableEntityQuery` (`allEntities()`), set a `DisplayRepresentation`, and call `indexAppEntities` / `deleteAppEntities` on `CSSearchableIndex` to keep Spotlight fresh.

### Building AttendeeEntity
Attendees (`calendar_attendee`) are accessed only through their parent event, so they conform to `TransientAppEntity` instead of `IndexedEntity` — no identifier, query, or index needed. Uses `IntentPerson` and two schematized `@AppEnum` types: `calendar_attendeeStatus` and `calendar_attendeeType`.

### Building EventEntity
The central `IndexedEntity` (from `calendar_event`), where the semantic index enables title and note-content Q&A. Composes `CalendarEntity` and `[AttendeeEntity]`, handles recurrence via `Calendar.RecurrenceRule`, union values for location and alarms, and event status / span enums.

### Open Events with OpenIntent
A small `OpenEventIntent` conforming to `system.open` takes an `EventEntity` and calls a `NavigationManager` to navigate to it, so tapping an event in Spotlight or Siri opens the detail view directly.

### Onscreen Awareness
Two view modifiers connect on-screen views to entities: `.appEntityIdentifier` on the event list and `.userActivity` (with an `EntityIdentifier`) on the detail view. This lets Siri resolve references like "this event" or "the third one" without needing the exact title.

### Create, Update, and Delete Events with Siri
- `CreateEventIntent` (from `calendar_createEvent`) resolves schema parameters — location union value, recurrence — into the data layer and returns an `EventEntity`.
- `UpdateEventIntent` (`calendar_updateEvent`) mirrors create but parameters are optional. The `IntentParameter.valueState` distinguishes `.set(value)` (change it), `.set(nil)` (explicitly clear it), and `.unset` (not in the request).
- `DeleteEventIntent` is the simplest — event plus an optional span for recurring events — and Siri automatically handles confirmation and disambiguation.

### Custom Snippet Views
Add `ShowsSnippetView` to the intent's return type and return a custom SwiftUI view (e.g., `EventSnippetView`) to replace Siri's default result card with the app's own visual identity.

## APIs & Frameworks

### AppIntents
- `@AppEntity(schema:)` — **[NEW]** schema-based entity declaration
- `AppEntity` protocol
- `TransientAppEntity` protocol — for non-indexed, relationship-only entities **[NEW]**
- `IndexedEntity` protocol — for Spotlight-indexed entities
- `EnumerableEntityQuery` protocol — `allEntities()` method
- `EntityStringQuery` protocol — `entities(matching:)` method
- `@AppIntent(schema:)` — **[NEW]** schema-based intent declaration
- `AppIntent` protocol — `perform()` method
- `OpenIntent` protocol — `target` parameter, navigates into app
- `IntentDialog` — `full:` and `supporting:` string variants
- `ProvidesDialog` result protocol
- `ShowsSnippetView` result protocol — **[NEW]** custom SwiftUI snippet in Siri
- `IntentParameter` / `@Parameter` property wrapper
- `IntentParameter.valueState` — `.set(value)`, `.set(nil)`, `.unset` **[NEW]**
- `IntentPerson` — system type for representing a person in intents
- `@AppEnum` — enum conformance for schema-typed parameters
- `DisplayRepresentation` — `title:`, `subtitle:`, `image:` init
- `TypeDisplayRepresentation`
- `EntityIdentifier` — identifies an entity for system integrations
- `@Dependency` property wrapper — dependency injection in intents/queries
- `IntentDonationManager` — donating interactions to the system
- `RelevantEntities` — suggesting contextually relevant entities
- `AppEntityContext` — context type for relevant entity registration
- **App Schema domains (calendar):**
  - `.calendar.calendar`
  - `.calendar.attendee`
  - `.calendar.event`
  - `.calendar.createEvent`
  - `.calendar.updateEvent`
  - `.system.open`
- `Calendar.RecurrenceRule` — used for recurring event parameters

### CoreSpotlight
- `CSSearchableIndex` — `indexAppEntities(_:)`, `deleteAppEntities(_:)` methods
- `IndexedEntityQuery` — supports re-indexing **[NEW]**

### SwiftUI
- `.appEntityIdentifier(_:)` view modifier — annotates a view with an entity **[NEW]**
- `.userActivity(_:isActive:_:)` view modifier — publishes `NSUserActivity` with `EntityIdentifier`

### Foundation / EventKit
- `NSUserActivity` — `appEntityIdentifier` property for onscreen awareness

## Code Highlights

**Schematized entity with Spotlight indexing:**
```swift
@AppEntity(schema: .calendar.calendar)
struct CalendarEntity: IndexedEntity {
    var id: UUID
    @Dependency var calendarManager: CalendarManager
    var displayRepresentation: DisplayRepresentation { ... }
    static var defaultQuery = CalendarEntityQuery()
}
```

**Update intent using valueState to distinguish explicit nil:**
```swift
@AppIntent(schema: .calendar.updateEvent)
struct UpdateEventIntent {
    @Parameter var event: EventEntity
    @Parameter var startDate: Date?

    func perform() async throws -> some ReturnsValue<EventEntity> {
        switch $startDate.valueState {
        case .set(let date): event.startDate = date
        case .set(nil):      event.startDate = nil
        case .unset:         break  // user didn't mention it
        }
        // ...
    }
}
```

**Onscreen awareness via view modifier:**
```swift
EventDetailView(event: event)
    .userActivity("app.eventDetail", isActive: true) { activity in
        activity.appEntityIdentifier = EntityIdentifier(
            for: EventEntity.self, identifier: event.id
        )
    }
```

**Custom Siri snippet view:**
```swift
func perform() async throws -> some ReturnsValue<EventEntity> & ShowsSnippetView {
    return .result(value: entity, view: EventSnippetView(event: entity))
}
```

## Takeaways
- App Schemas eliminate custom NLP — conforming entities and intents to a schema domain is all Siri needs to understand your app's content and actions.
- `IndexedEntity` + `CSSearchableIndex` enables semantic search and content Q&A over your app's data in Spotlight and Siri.
- `IntentParameter.valueState` is critical for update intents: it distinguishes an explicit clear (`.set(nil)`) from a parameter the user never mentioned (`.unset`).
- View modifiers (`.appEntityIdentifier`, `.userActivity`) link on-screen UI to entities, enabling context-aware Siri references like "that event" or "the third one."

---
_Source: WWDC26 Session 344 page (abstract, chapter summaries, code samples, and resource links)._
