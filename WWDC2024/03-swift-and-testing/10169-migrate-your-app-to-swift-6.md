# Migrate Your App to Swift 6
**WWDC24 · Session 10169** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10169/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia 15, watchOS 11, tvOS 18, visionOS 2

## Overview
Swift 6 introduces complete data-race safety checking enforced at compile time. This session walks through migrating the Coffee Tracker sample app — module by module — to Swift 6, demonstrating the incremental migration strategy the Swift team recommends. It covers how to enable strict concurrency checking per module, understand and resolve each category of warning, and finally enable the Swift 6 language mode once a module is clean.

The key insight: Swift 6 migration is not a big-bang rewrite. Enable warnings first, fix them, then flip the language mode — one module at a time.

## Key Topics

**The Migration Strategy**
- Migrate module by module, not all at once — each module can independently move from Swift 5 → strict concurrency warnings → Swift 6 language mode
- Start with leaf modules (fewest dependencies); work inward toward the app target
- Enable `SWIFT_STRICT_CONCURRENCY = complete` in build settings to surface all data-race warnings without breaking the build
- Once all warnings are resolved, switch `SWIFT_VERSION = 6` for that module

**Category 1: Global Variables (Shared Mutable State)**
- `var` globals that are non-isolated trigger "not concurrency-safe" warnings
- Three options (in order of preference):
  1. Change `var` to `let` — safest; works when the value never changes after initialization
  2. Add `@MainActor` — isolates to the main actor; access must happen on the main thread
  3. `nonisolated(unsafe)` — opt out of checking; only use when you guarantee external synchronization
- Most `Logger`, `HKHealthStore`, and similar singletons should be `let`

**Category 2: Main-Actor-Isolated Classes Calling Main-Actor APIs**
- A non-isolated free function calling `@MainActor`-isolated APIs produces: "Call to main actor-isolated … in a synchronous nonisolated context"
- Fix: annotate the calling function with `@MainActor`
- Entire observable classes (e.g., `ObservableObject` subclasses) should be `@MainActor`

**Category 3: Protocol Conformances with Actor Isolation Mismatches**
- A `@MainActor`-isolated class cannot satisfy a protocol requirement that is not `@MainActor`-isolated
- Options:
  1. Mark the method `nonisolated` — then access to `@MainActor`-isolated properties inside requires a `Task { @MainActor in ... }` or `MainActor.assumeIsolated { ... }`
  2. Add `@MainActor` to the protocol itself — fixes the mismatch at the source (requires you to own the protocol)
  3. Use `@preconcurrency` on the conformance — silences the warning when the protocol is from code you don't control and is known to be called on the main actor; remove once the protocol adopts `@MainActor`
- `MainActor.assumeIsolated { }` — assert at runtime that you're on the main actor; crashes if called from another actor

**Category 4: Sending Values Between Actors**
- Passing a value from a main-actor-isolated context to another actor requires the type to be `Sendable`
- Simple value types (structs with `let` stored properties of `Sendable` types) conform to `Sendable` automatically or can be marked explicitly
- If a stored property is non-`Sendable`, either make it `Sendable` or use `nonisolated(unsafe)` as a last resort
- Enums with all `Sendable` cases are `Sendable`

**Category 5: Delegate Callbacks and Concurrency**
- Delegate methods from frameworks (e.g., `CLLocationManagerDelegate`) are often called from arbitrary threads, not the main actor
- Pattern: mark the delegate class `@MainActor`; mark the protocol method `nonisolated`; use `MainActor.assumeIsolated { }` inside the method body to access main-actor state synchronously
- This is safe when the framework guarantees delivery on the main thread (or when you set the delegate's dispatch queue)

**Enabling Swift 6 Language Mode**
- Set `SWIFT_VERSION = 6` in build settings; this makes all data-race issues errors (not warnings)
- Do this only after clearing all `SWIFT_STRICT_CONCURRENCY = complete` warnings in that module

## APIs & Frameworks

**Swift Concurrency**
- `@MainActor` — isolate a type, function, or property to the main actor
- `nonisolated` — opt a specific member out of actor isolation
- `nonisolated(unsafe)` **[NEW]** — disable concurrency-safety checking for a declaration; use when external synchronization is guaranteed
- `Sendable` — protocol; conformance declares a type is safe to share across concurrency domains
- `@preconcurrency` **[NEW]** — annotation on conformances/imports to suppress isolation mismatch warnings for legacy APIs
- `MainActor.assumeIsolated(_:)` **[NEW]** — assert at runtime that current context is the main actor; execute a closure with main-actor isolation
- `Task { @MainActor in ... }` — schedule work on the main actor asynchronously
- `actor` — define a reference type with its own isolated executor

**Xcode Build Settings**
- `SWIFT_STRICT_CONCURRENCY = complete` — enable all data-race warnings without enforcing Swift 6 mode **[NEW]**
- `SWIFT_VERSION = 6` — enable Swift 6 language mode; all data-race issues become errors **[NEW]**

**Swift Standard Library**
- `Sendable` — existing protocol; explicitly conforming value types resolves actor-boundary transfer warnings
- `CLLocationUpdate.liveUpdates()` — async sequence API for location updates (watchOS 10+)

## Code Highlights

Three options for a non-concurrency-safe global:
```swift
// Option 1 (best): make it a let constant
let logger = Logger(subsystem: "com.example", category: "Root View")

// Option 2: isolate to main actor
@MainActor var logger = Logger(subsystem: "com.example", category: "Root View")

// Option 3 (last resort): unsafe opt-out
nonisolated(unsafe) var logger = Logger(subsystem: "com.example", category: "Root View")
```

Isolate a whole class to the main actor:
```swift
@MainActor
class Recaffeinater: ObservableObject {
    @Published var recaffeinate: Bool = false
    var minimumCaffeine: Double = 0.0
}
```

Satisfy a non-isolated delegate protocol from a `@MainActor` class:
```swift
extension Recaffeinater: @preconcurrency CaffeineThresholdDelegate {
    public func caffeineLevel(at level: Double) {
        // Runs on main actor because of @preconcurrency
    }
}
```

`nonisolated` + `MainActor.assumeIsolated` for framework callbacks:
```swift
@MainActor
class CoffeeLocationDelegate: NSObject, CLLocationManagerDelegate {
    nonisolated func locationManager(_ manager: CLLocationManager,
                                     didUpdateLocations locations: [CLLocation]) {
        MainActor.assumeIsolated {
            self.location = locations.last
        }
    }
}
```

Mark a value type `Sendable` to allow actor-boundary transfer:
```swift
public struct Drink: Hashable, Codable, Sendable {
    public let mgCaffeine: Double
    public let date: Date
    public let uuid: UUID
    public let type: DrinkType  // DrinkType must also be Sendable
}
```

## Takeaways
- Enable `SWIFT_STRICT_CONCURRENCY = complete` first; fix all warnings before switching to `SWIFT_VERSION = 6` — don't try to do both at once.
- Most global-variable warnings are fixed by changing `var` to `let`; reserve `@MainActor` and `nonisolated(unsafe)` for cases where mutation is genuinely needed.
- Use `@preconcurrency` on conformances to protocols from frameworks you don't control; remove it once the framework adds `@MainActor` annotations.
- `MainActor.assumeIsolated { }` is the correct tool for framework delegate callbacks that are documented to arrive on the main thread — it's synchronous and avoids the task-scheduling overhead of `Task { @MainActor in ... }`.

---
_Source: WWDC24 Session 10169 page (abstract, chapter list, code samples, and resource links)._
