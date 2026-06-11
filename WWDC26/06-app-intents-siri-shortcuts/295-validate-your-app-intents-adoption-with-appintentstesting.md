# Validate Your App Intents Adoption with AppIntentsTesting
**WWDC26 · Session 295** · [Watch](https://developer.apple.com/videos/play/wwdc2026/295/)

_Platforms:_ iOS, iPadOS, macOS

## Overview
This session introduces `AppIntentsTesting`, a new framework for testing App Intents integrations without UI automation. Tests run through the same infrastructure used by Siri, Shortcuts, and Spotlight — with no mocks and no direct imports of app code — making them stable, fast, and CI-friendly. The framework is demonstrated throughout using the CometCal calendar sample and its full test suite.

The session walks from the simplest possible test (execute one intent, assert on its result) through entity query testing, chained multi-intent tests, test-only seed intents, Spotlight indexing regression tests, and view annotation verification. A recommended workflow positions AppIntentsTesting as the unit/integration layer below manual Siri and Shortcuts testing.

The CometCal sample and full test suite are available for download alongside the `AppIntentsTesting` documentation.

## Key Topics

### How AppIntentsTesting Works
Tests run in a standard `XCUITest` bundle while the app runs in a separate process. The framework executes intents through the full App Intents stack — real entities, real queries, real Spotlight — with no mocking. Access intents by bundle identifier, not by importing app targets, keeping tests decoupled and deployment-friendly.

### Your First Test: Execute an Intent
1. Create an `IntentDefinitions` from the app's bundle identifier.
2. Look up an intent by name: `definitions.intents["CreateCalendarIntent"]`.
3. Call `makeIntent(name:color:)` with parameters.
4. `await intent.run()` executes through the full stack.
5. Access result properties via dynamic member lookup (`result.value.title`).

### Testing Entity Queries
Drive test-driven development of `EntityStringQuery`: call `entityDefinition.entities(matching:)`, watch it fail, implement the `entities(matching:)` conformance, rerun to green. The same API tests `EntityQuery`, `EnumerableEntityQuery`, and `IntentValueQuery`.

### Combining Multiple Intents
Chain intents in one test to replicate how people compose Shortcuts: run `CreateEventIntent`, then pass its returned `EventEntity` value directly into `UpdateEventIntent`. Runtime entity resolution (e.g., resolving a `CalendarEntity` from a plain string) happens automatically.

### Test-Only Intents
Intents that exist only for test setup (seed known data, navigate to a specific screen, wrap unadopted functionality). Make any intent test-only with `isDiscoverable: false` inside an `#if DEBUG` guard so they never appear in Siri, Shortcuts, or Spotlight.

### Testing Spotlight Indexing
Call `entityDefinition.spotlightQuery(_:)` before and after creating an entity. A failing test that asserts the post-create result is non-empty will catch real bugs (e.g., indexing code accidentally commented out).

### Testing View Annotations
Open an event via `OpenEventIntent`, confirm the UI with XCUI, then call `entityDefinition.viewAnnotations()` to assert which entity is annotated on screen. Caught a real bug where the wrong `EntityIdentifier` (calendar ID instead of event ID) was used in the modifier.

### The App Intents Testing Workflow
- **AppIntentsTesting**: unit/integration layer — intents, queries, entities, Spotlight, view annotations
- **Manual Siri + Shortcuts**: conversation quality, disambiguation, voice experience
The two layers complement; AppIntentsTesting does not replace Siri testing.

## APIs & Frameworks

### AppIntentsTesting (NEW framework)
- `IntentDefinitions` — **[NEW]** `init(bundleIdentifier:)`; `.intents[name]` subscript
- `IntentDefinition` — **[NEW]** `.makeIntent(param:...)` builder; `.run()` async execution
- `IntentResult` (test) — dynamic member lookup on result properties; `.value` accessor
- `EntityDefinition` — **[NEW]** per-entity-type test handle
  - `.entities(matching:)` — test `EntityStringQuery`
  - `.entities(for:)` — test `EntityQuery` by identifier
  - `.suggestedEntities()` — test `EnumerableEntityQuery`
  - `.spotlightQuery(_:)` — **[NEW]** query Spotlight index for this entity type
  - `.viewAnnotations()` — **[NEW]** return on-screen `ViewAnnotation` array
- `ViewAnnotation` — **[NEW]** `.entity` property (dynamic member lookup); wraps an on-screen entity annotation
- `AppIntent.isDiscoverable` — set to `false` to hide from Siri/Shortcuts/Spotlight **[NEW]**

### AppIntents
- `AppIntent` protocol — `perform()`, `isDiscoverable`
- `EntityStringQuery` — `entities(matching:)` — testable via `EntityDefinition`
- `EntityQuery` — `entities(for:)` — testable via `EntityDefinition`
- `EnumerableEntityQuery` — `allEntities()` — testable via `EntityDefinition`
- `IndexedEntity` — Spotlight indexing tested via `spotlightQuery(_:)`
- `OpenIntent` — used in view annotation tests to navigate to a specific screen
- `.appEntityIdentifier(_:)` view modifier — tested via `viewAnnotations()`
- `EntityIdentifier` — the value asserted in view annotation tests

### XCTest / XCUITest
- `XCUIApplication` — used alongside `AppIntentsTesting` for UI verification
- `XCTAssertEqual`, `XCTAssertTrue` — standard assertions
- `waitForExistence(timeout:)` — UI element wait

## Code Highlights

**Basic intent test:**
```swift
import AppIntentsTesting

func testCreateCalendar() async throws {
    let definitions = IntentDefinitions(bundleIdentifier: "com.example.CometCal")
    let createCalendar = definitions.intents["CreateCalendarIntent"]
    let result = try await createCalendar.makeIntent(name: "Occupy Saturn", color: "red").run()
    XCTAssertEqual(try result.value.title, "Occupy Saturn")
}
```

**Entity string query test:**
```swift
func testEventStringQuery() async throws {
    let results = try await eventEntityDefinition.entities(matching: "Cosmic Ray")
    XCTAssertEqual(results.count, 1)
    XCTAssertEqual(try results[0].title, "Cosmic Ray Calibration")
}
```

**Chained multi-intent test:**
```swift
func testCreateAndUpdateEvent() async throws {
    let createResult = try await createEventDefinition
        .makeIntent(title: "Asteroid Dodgeball Practice", startDate: Date(), ...).run()
    let updateResult = try await updateEventDefinition
        .makeIntent(title: "Asteroid Dodgeball Rules Overview", event: createResult.value).run()
    XCTAssertEqual(try updateResult.value.title, "Asteroid Dodgeball Rules Overview")
}
```

**Spotlight indexing regression test:**
```swift
func testNewEventIndexedInSpotlight() async throws {
    let before = try await eventEntityDefinition.spotlightQuery("Supernova Viewing Party")
    XCTAssertTrue(before.isEmpty)
    // ... create the event ...
    let after = try await eventEntityDefinition.spotlightQuery("Supernova Viewing Party")
    XCTAssertEqual(after.count, 1)
}
```

**Test-only seed intent:**
```swift
#if DEBUG
struct SeedSampleEventsIntent: AppIntent {
    static let isDiscoverable = false
    func perform() async throws -> some IntentResult {
        // Create known event fixtures
        return .result()
    }
}
#endif
```

**View annotation test:**
```swift
func testEventViewAnnotation() async throws {
    try await openEventDefinition.makeIntent(target: "Morning Launch Briefing").run()
    let annotations = try await eventEntityDefinition.viewAnnotations()
    XCTAssertEqual(annotations.count, 1)
    XCTAssertEqual(try annotations[0].entity.title, "Morning Launch Briefing")
}
```

## Takeaways
- `AppIntentsTesting` runs through the real App Intents stack in an `XCUITest` bundle — no mocks, no app-target imports — making tests stable enough for CI without sacrificing fidelity.
- `spotlightQuery(_:)` and `viewAnnotations()` let you write regression tests for two of the most fragile integration points (Spotlight indexing and view annotations) that are otherwise only testable manually.
- Test-only intents (`isDiscoverable: false` + `#if DEBUG`) are the clean way to seed known state or navigate to specific screens without polluting the public intent surface.
- The recommended workflow positions AppIntentsTesting as the primary automated layer, with Siri and Shortcuts reserved for conversation quality and voice-flow validation.

---
_Source: WWDC26 Session 295 page (abstract, chapter summaries, code samples, and resource links)._
