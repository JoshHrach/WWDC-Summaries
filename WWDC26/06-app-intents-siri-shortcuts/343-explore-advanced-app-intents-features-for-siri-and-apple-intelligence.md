# Explore Advanced App Intents Features for Siri and Apple Intelligence
**WWDC26 · Session 343** · [Watch](https://developer.apple.com/videos/play/wwdc2026/343/)

_Platforms:_ iOS, iPadOS, macOS, watchOS, visionOS

## Overview
This session covers advanced App Intents techniques for polishing how your app works with Siri and Apple Intelligence, demonstrated across three sample apps: CosmoTunes (audio), UnicornChat (messaging), and CometCal (calendar). The session is organized around three themes: shaping the Siri conversation, improving content discovery, and leveraging existing system integrations.

From custom dialog strings and clarifying questions to structured search queries and onscreen awareness, the session shows how each incremental technique makes the Siri experience feel more native and personal. Particular emphasis is placed on the new `OwnershipProvidingEntity` protocol for smart confirmation behavior and the multi-mode onscreen awareness system (NSUserActivity, view annotations, collection annotations).

The session concludes with a recommended adoption order: start with `DisplayRepresentation`, then index entities, add `IntentValueQuery` and in-app search, annotate views and existing integrations, and finally donate UI interactions.

## Key Topics

### Customize How Siri Responds
Return an empty result to let Siri generate its own response, or adopt `ProvidesDialog` and provide an `IntentDialog` with `full:` (voice) and `supporting:` (visual) strings. Intents can also ask clarifying questions mid-execution by calling `$parameter.requestValue(_:)` on an `IntentParameter`.

### Visual Responses
`DisplayRepresentation` (title, subtitle, image) is used across Siri responses, disambiguation, Spotlight, and Shortcuts. For richer custom UI, adopt `ShowsSnippetView` and return a SwiftUI view. Customize only where it adds value and account for voice-only devices.

### Interaction Donations
System interactions are tracked automatically; UI interactions (tapping a send button) need explicit donation via `IntentDonationManager.shared.donate(intent:result:)`. Apple Intelligence uses donations to learn preferences and track ongoing activities. Accuracy matters — excessive or inaccurate donations are filtered out.

### Confirmations and Entity Ownership
Siri auto-confirms intents with meaningful side effects, especially on shared/public content. Implement the new `OwnershipProvidingEntity` protocol to return `.shared`, `.public`, or `.unknown` so Siri knows when to confirm. Your `DisplayRepresentation` is used as the confirmation visual.

### Semantic Index with IndexedEntity
Adopt `IndexedEntity` and call `CSSearchableIndex.indexAppEntities(_:)` to enable meaning-based search. Keep the index fresh with add, update, and delete operations. Support re-indexing with the new `IndexedEntityQuery`.

### Structured Search with IntentValueQuery
For server-side or large/fast-changing catalogs that can't be pre-indexed, implement `IntentValueQuery`: the system passes a structured `AudioSearch` (or similar input) and you return one or more entity types. CosmoTunes maps `AudioSearch` criteria (`.searchQuery`, `.unspecified`, `.url`) to a union of songs and playlists.

### In-App Search
Adopt the `system.searchInApp` schema to let Siri hand off to your custom in-app search UI. Works regardless of which domains you adopt or whether you index entities.

### Onscreen Awareness
Three levels of annotation connect on-screen views to entities:
1. **NSUserActivity** — single primary item (`activity.appEntityIdentifier`)
2. **View entity annotation** — one of many items (`.appEntityIdentifier(_:)` modifier)
3. **Collection annotation** — lists/grids (`forSelectionType:` variant)

Enable `displayRepresentations(for:requestedComponents:)` on your query for fast on-screen entity resolution. All techniques work in UIKit and AppKit as well.

### Leverage Existing Integrations
Attach `EntityIdentifier`s to existing system integrations without additional work:
- `UNMutableNotificationContent.appEntityIdentifiers` — announces notifications Siri can act on
- `MediaSessionRepresentable.content.appEntityIdentifiers` — Now Playing entities (most to least specific)
- `AlarmKit.AlarmConfiguration.appEntityIdentifier` — alarm entities for "snooze it" actions

Only use persistent entities (not `TransientAppEntity`) for these integrations.

## APIs & Frameworks

### AppIntents
- `ProvidesDialog` result protocol — return `IntentDialog(full:supporting:)`
- `IntentDialog` — `full:` (voice) and `supporting:` (short visual) string variants **[NEW per-field]**
- `IntentParameter.requestValue(_:)` — ask a clarifying question mid-intent **[NEW]**
- `ShowsSnippetView` result protocol — custom SwiftUI snippet view
- `DisplayRepresentation` — `title:`, `subtitle:`, `image:` init
- `DisplayRepresentation.Components` — `requestedComponents` parameter for queries **[NEW]**
- `IntentDonationManager` — `shared.donate(intent:result:)` for UI interaction donations
- `OwnershipProvidingEntity` protocol — `ownership` property returning `EntityOwnership` **[NEW]**
- `EntityOwnership` — `.shared`, `.public`, `.unknown` cases **[NEW]**
- `IndexedEntity` protocol — Spotlight semantic indexing
- `IndexedEntityQuery` — supports re-indexing **[NEW]**
- `IntentValueQuery` protocol — structured search; `values(for:)` method **[NEW]**
- `AudioSearch` — system search input type for audio queries
- `AudioSearch.Criteria` — `.searchQuery(String)`, `.unspecified`, `.url` cases
- `StringSearchCriteria` — `term` property for in-app search
- `EntityIdentifier` — `init(for:identifier:)` for annotating system integrations
- `@AppIntent(schema: .system.searchInApp)` — in-app search schema **[NEW]**
- `@AppIntent(schema: .audio.addToPlaylist)` — audio schema example
- `@AppIntent(schema: .clock.createTimer)` — clock schema example
- `@AppEntity(schema: .audio.song)` — audio song entity schema
- `@AppEntity(schema: .calendar.event)` — calendar event entity schema
- `EntityQuery` — `displayRepresentations(for:requestedComponents:)` method **[NEW]**

### CoreSpotlight
- `CSSearchableIndex` — `indexAppEntities(_:)` method

### SwiftUI / UIKit / AppKit
- `.appEntityIdentifier(_:)` view modifier — annotate a single entity on screen **[NEW]**
- `.appEntityIdentifier(forSelectionType:_:)` — collection / list annotation **[NEW]**
- `.userActivity(_:isActive:_:)` — NSUserActivity with `appEntityIdentifier` property

### UserNotifications
- `UNMutableNotificationContent.appEntityIdentifiers` — array of `EntityIdentifier` **[NEW]**

### NowPlaying
- `MediaSessionRepresentable` — `content.appEntityIdentifiers` property **[NEW]**
- `MusicContent` — `appEntityIdentifiers` array

### AlarmKit
- `AlarmManager.AlarmConfiguration` — `appEntityIdentifier:` parameter **[NEW]**

## Code Highlights

**Custom dialog with full and supporting strings:**
```swift
return .result(
    dialog: IntentDialog(
        full: "Added \(song.title) to the \(playlist.title) mix tape.",
        supporting: "Added"
    )
)
```

**Clarifying question mid-intent:**
```swift
label = try await $label.requestValue(
    "You already have a timer running. What should we call this one?"
)
```

**OwnershipProvidingEntity for smart confirmation:**
```swift
@AppEntity(schema: .calendar.event)
struct EventEntity: OwnershipProvidingEntity {
    var ownership: EntityOwnership {
        attendees.isEmpty ? .unknown : .shared
    }
}
```

**IntentValueQuery for structured search:**
```swift
struct AudioIntentValueQuery: IntentValueQuery {
    func values(for input: AudioSearch) async throws -> [AudioEntity] {
        switch input.criteria {
        case .searchQuery(let query): return try await searchResults(for: query)
        case .unspecified:            return try await likedSongResults()
        }
    }
}
```

**Entity annotations on UNMutableNotificationContent:**
```swift
content.appEntityIdentifiers = [
    EntityIdentifier(for: MessageEntity.self, identifier: message.id)
]
```

## Takeaways
- Custom `IntentDialog` with `full:` / `supporting:` and clarifying questions via `requestValue(_:)` give Siri your app's voice rather than a generic response.
- `OwnershipProvidingEntity` is the right hook for controlling confirmation frequency — keep the state accurate so Siri doesn't over- or under-confirm.
- `IndexedEntity` + `IntentValueQuery` together cover both local indexed content and large/remote catalogs; `system.searchInApp` bridges either approach into your full in-app search UI.
- Attaching `EntityIdentifier`s to UserNotifications, Now Playing, and AlarmKit is low-cost and high-value: it extends onscreen awareness to content that's already surfaced by the system.

---
_Source: WWDC26 Session 343 page (abstract, chapter summaries, code samples, and resource links)._
