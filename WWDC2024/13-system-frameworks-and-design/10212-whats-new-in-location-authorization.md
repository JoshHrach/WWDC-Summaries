# What's New in Location Authorization
**WWDC24 · Session 10212** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10212/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia 15, watchOS 11, visionOS 2

## Overview
Core Location authorization is effectively version 2.0 in iOS 18. The centerpiece is `CLServiceSession` — a new declarative struct that lets apps express authorization *goals* (what they need) rather than issuing imperative request calls (what they are doing). Paired with this is a new system of **Diagnostic Properties** on sessions and API result objects that explain in actionable detail why an authorization goal cannot be met, eliminating the need for timeouts and guesswork.

Iterating `CLLocationUpdate.liveUpdates()` or `CLMonitor.events` now implicitly creates a `.whenInUse` service session, so many apps can eliminate most authorization boilerplate entirely — sometimes just by deleting code.

## Key Topics

### CLServiceSession: Declarative Authorization Goals
`CLServiceSession` is a new immutable, `Sendable` struct. Instead of calling `requestWhenInUseAuthorization()` or `requestAlwaysAuthorization()` imperatively, apps hold a `CLServiceSession` for as long as a location-dependent feature is active. Core Location reads the goal and presents appropriate authorization requests when the user can respond — even handling recovery if the app was backgrounded when it would have asked.

Three authorization goals: `.whenInUse`, `.always`, and `.none`. To layer additional requirements (e.g., full accuracy for navigation), hold an additional `CLServiceSession(authorization: .whenInUse, fullAccuracyPurposeKey: "Nav")` alongside the general one — don't replace, layer.

**Key properties of the approach:**
- Sessions are held proactively — even if your app already has full accuracy, hold a full-accuracy session whenever a feature that warrants it is active
- Sessions should be tied to feature lifetime, not app lifecycle events
- Multiple independent components can each hold their own session; Core Location reconciles them

### Implicit Service Sessions
Iterating `CLLocationUpdate.liveUpdates()` or `CLMonitor.events` automatically acts as a `.whenInUse` session. No explicit `CLServiceSession` is needed for these common patterns. This behavior can be disabled by setting `NSLocationRequireExplicitServiceSession` = `true` in `Info.plist`, after which explicit sessions are required.

### Session Lifecycle: Suspension and Termination
Sessions should span the full logical duration of a feature — including while the app is backgrounded. When an app is suspended or terminated, Core Location tracks outstanding sessions and implicit sessions for a short window after the next launch. Apps must resume iterating updates or re-create session objects quickly after re-launch to maintain continuity.

For `.always` authorization: starting in iOS 18, Always authorization is only effective while holding an explicit `.always` CLServiceSession, and sessions can only be started while the app is in the foreground (or while another session is already in effect).

### Diagnostic Properties
Each `CLServiceSession` exposes a `diagnostics` async sequence yielding `CLServiceSession.Diagnostic` values with boolean properties:
- `authorizationDenied` — user has denied location access to the app
- `authorizationDeniedGlobally` — location services disabled system-wide
- `authorizationRequestInProgress` — Core Location is currently asking the user
- `insufficientlyInUse` — app is not sufficiently in-use for Core Location to ask
- `alwaysAuthorizationDenied` — `.always` goal was not granted

Diagnostic Properties also appear on result objects:
- `CLLocationUpdate`: `authorizationDenied`, `accuracyLimited` (approximate location updated every 15–20 min), `locationUnavailable`
- `CLMonitor.Event`: `authorizationDenied`, `accuracyLimited`, condition-count exceeded, and others

## APIs & Frameworks

**Core Location**
- `CLServiceSession` **[NEW]** — declarative authorization goal holder
  - `init(authorization: CLServiceSession.AuthorizationRequirement)` **[NEW]**
  - `init(authorization:fullAccuracyPurposeKey:)` — requires full accuracy **[NEW]**
  - `CLServiceSession.AuthorizationRequirement`: `.whenInUse`, `.always`, `.none` **[NEW]**
  - `diagnostics: AsyncSequence<CLServiceSession.Diagnostic>` **[NEW]**
- `CLServiceSession.Diagnostic` **[NEW]** — diagnostic snapshot struct
  - `.authorizationDenied: Bool` **[NEW]**
  - `.authorizationDeniedGlobally: Bool` **[NEW]**
  - `.authorizationRequestInProgress: Bool` **[NEW]**
  - `.insufficientlyInUse: Bool` **[NEW]**
  - `.alwaysAuthorizationDenied: Bool` **[NEW]**
- `CLLocationUpdate` (introduced iOS 17) — gains new diagnostic properties **[NEW properties]**
  - `.authorizationDenied: Bool` **[NEW]**
  - `.accuracyLimited: Bool` **[NEW]**
  - `.locationUnavailable: Bool` **[NEW]**
  - `.stationary: Bool` (existing)
- `CLMonitor.Event` (introduced iOS 17) — gains new diagnostic properties **[NEW properties]**
  - `.authorizationDenied: Bool` **[NEW]**
  - `.accuracyLimited: Bool` **[NEW]**
  - Condition count exceeded property **[NEW]**
- `CLLocationUpdate.liveUpdates()` — implicitly creates `.whenInUse` service session when iterated (existing, new implicit session behavior)
- `CLMonitor.events` — implicitly creates `.whenInUse` service session when iterated (existing, new implicit session behavior)
- `NSLocationRequireExplicitServiceSession` — `Info.plist` key to disable implicit sessions **[NEW]**
- `CLLocationManager.requestWhenInUseAuthorization()` — still available but superseded by CLServiceSession
- `requestTemporaryFullAccuracyAuthorization(withPurposeKey:)` — still available; CLServiceSession's `fullAccuracyPurposeKey` is the preferred alternative

## Code Highlights

Minimal location code using implicit session (no explicit CLServiceSession needed):
```swift
Task {
    for try await update in CLLocationUpdate.liveUpdates() {
        // Process update.location or update.authorizationDenied
    }
}
```

Explicit CLServiceSession with diagnostics (for more control):
```swift
Task {
    let mySession = CLServiceSession(authorization: .whenInUse)
    for try await diagnostic in mySession.diagnostics {
        if !diagnostic.authorizationRequestInProgress {
            reactToChanges(authorized: !diagnostic.authorizationDenied)
        }
    }
}
```

Layering a full-accuracy session for a navigation feature:
```swift
// Keep base session for general when-in-use use
let baseSession = CLServiceSession(authorization: .whenInUse)

// Add when user starts navigation
let navSession = CLServiceSession(authorization: .whenInUse, fullAccuracyPurposeKey: "Nav")
// Hold both for duration of navigation feature
```

Checking `authorizationDeniedGlobally` vs `authorizationDenied` for tailored UI:
```swift
for try await diagnostic in mySession.diagnostics {
    if diagnostic.authorizationDeniedGlobally {
        showSystemLocationDisabledMessage()
    } else if diagnostic.authorizationDenied {
        showAppLocationDeniedMessage()
    }
}
```

## Takeaways
- Replace imperative `requestWhenInUseAuthorization()` calls with `CLServiceSession` (or rely on the implicit session from iterating `liveUpdates()`); the declarative model is more robust across app lifecycle events.
- Tie session objects to feature lifetime — hold them across backgrounding, suspension, and re-launch to maintain continuity; Core Location tracks outstanding sessions and resumes delivery when the app re-launches.
- Use `CLServiceSession.Diagnostic` boolean properties instead of timeouts to understand why authorization is not yet granted — the system will always tell you via diagnostics rather than silence.
- Starting iOS 18, `.always` authorization requires an explicit `.always` CLServiceSession held while the app is in the foreground; update any background-location features to start their sessions before leaving the foreground.

---
_Source: WWDC24 Session 10212 page (abstract, chapter summaries, code samples, and resource links)._
