# What's new in Wallet
**WWDC25 · Session 202** · [Watch](https://developer.apple.com/videos/play/wwdc2025/202/)

_Platforms:_ iOS 26, watchOS 26

## Overview
This session covers three improvements to the Wallet platform: multi-event tickets (Upcoming Events), a redesigned boarding pass with automatic flight tracking and a new Live Activity, and a new `PKPassLibrary` API for adding passes in the background after a one-time user authorization prompt.

The boarding pass redesign is the most substantial change. Rather than requiring airlines to push updates for every gate change, the upgraded boarding pass integrates with Apple's flight service to update times, boarding windows, and status automatically. The session is aimed at pass issuers (airlines, event organizers) and app developers who integrate with PassKit.

## Key Topics

### Upcoming Events for Poster Event Tickets
Poster Event Tickets (introduced in iOS 18) now support an `upcomingPassInformation` array in `pass.json`. Each entry has a `type: "event"`, a unique `identifier`, a `displayName`, and a date. Each upcoming event mirrors the structure of the parent Poster Event Ticket: it can have its own `semantics`, `additionalInfoFields`, `backFields`, `URLs`, and `images` (including a `headerImage` and `venueMap`). The event guide is specific to each upcoming event; properties set on the pass-level are not automatically inherited. Use `isActive` to mark events as current; remove canceled events from the array entirely.

### Upgraded boarding passes
The new boarding pass design:
- Pulls live flight status from Apple's flight service — gate changes, delays, and cancellations update automatically without a push update
- Drives a system **Live Activity** (sharable via Messages) with real-time flight info
- Integrates with Maps (directions to airport) and FindMy (luggage tracking)
- Shows an **airline services and upgrades** section built from URLs and semantics in `pass.json`

Key semantics for flight identification: `airlineCode`, `flightNumber`, `originalDepartureDate` (or marketing airline code for codeshares). Gate times come from `originalDepartureDate`/`currentDepartureDate` semantics; boarding time from `originalBoardingDate`/`currentBoardingDate`. When the flight is delayed, the system recalculates boarding time to maintain the boarding duration.

**Badges** are generated automatically from semantics: `passengerServiceSSRs` for IATA Special Service Request codes, plus custom label semantics for airline-specific data. Airlines provide URL fields (`purchaseLoungeAccessURL`, etc.) and semantics (e.g., `airlineLoungePlaceID`) for the services screen.

The new schema is additive — existing boarding passes continue to render on older OS versions. The migration path: add semantics and URLs to existing boarding passes.

### Background add passes API
New `PKPassLibrary` capability: request one-time user authorization to add passes silently in the future.

```swift
// One-time prompt
let status = await PKPassLibrary().requestAuthorization(for: .backgroundAddPasses)

// Check before adding
if PKPassLibrary().authorizationStatus(for: .backgroundAddPasses) == .authorized {
    try PKPassLibrary().addPasses([pass])  // Wallet notifies user silently
}
```

Authorization status persists and is user-toggleable in Settings. The API is targeted at frequent Wallet users (e.g., frequent flyers adding multiple boarding passes per week).

## APIs & Frameworks

### PassKit (PKPassLibrary)
- **`PKPassLibrary.requestAuthorization(for: .backgroundAddPasses)`** **[NEW]** — async, one-time prompt
- **`PKPassLibrary.authorizationStatus(for: .backgroundAddPasses)`** **[NEW]** — check current status without prompting
- `PKPassLibrary.addPasses(_:)` (existing) — now works silently when background authorization is granted
- `PKPassLibrary.Capability.backgroundAddPasses` **[NEW]**
- `PKAddPassesViewController` (existing)

### Wallet pass schema (pass.json)
- **`upcomingPassInformation`** array **[NEW]** — array of upcoming event objects for multi-event tickets
- Upcoming event fields: `type`, `identifier`, `displayName`, date, `semantics`, `additionalInfoFields`, `backFields`, `URLs`, `images.headerImage`, `images.venueMap`, `isActive`
- `images.venueMap.reuseExisting` **[NEW]** — boolean to reuse the pass-level venue map
- **Upgraded boarding pass schema** **[NEW]**: `airlineCode`, `flightNumber`, `originalDepartureDate`, `currentDepartureDate`, `originalBoardingDate`, `currentBoardingDate`, `passengerServiceSSRs`, `airlineLoungePlaceID`, `purchaseLoungeAccessURL` and other URL fields
- Badge-driving semantics: `passengerServiceSSRs` (IATA SSR codes), `ticketFareClass`, `airlineStatus` and others

### SwiftUI / PassKit integration
- `PKPassLibrary` used in a SwiftUI context via `Task` / async-await

## Code Highlights

```swift
// Request background add passes authorization
Button("Add to Wallet") {
    Task {
        _ = await PKPassLibrary().requestAuthorization(for: .backgroundAddPasses)
    }
}

// Add passes silently on subsequent launches
.onAppear {
    guard PKPassLibrary().authorizationStatus(for: .backgroundAddPasses) == .authorized else { return }
    try? PKPassLibrary().addPasses([boardingPass])
}
```

```json
// Upcoming event in pass.json
{
  "upcomingPassInformation": [
    {
      "type": "event",
      "identifier": "event-001",
      "displayName": "Opening Night",
      "date": "2025-09-15T19:30:00-04:00",
      "isActive": true,
      "semantics": { "venueName": "The Paramount" },
      "URLs": { "manageURL": "https://example.com/manage" }
    }
  ]
}
```

## Takeaways
- Add the `upcomingPassInformation` array to existing event tickets to give holders a single pass that covers an entire season or multi-day event — no need to reissue passes.
- Migrate boarding passes by adding semantics (`airlineCode`, `flightNumber`, `originalDepartureDate`) to unlock automatic flight tracking, the Live Activity, and badges at no additional development cost.
- Implement `requestAuthorization(for: .backgroundAddPasses)` and call it right after the user's first successful Wallet add — subsequent passes (like weekly boarding passes) will then appear silently.
- The new boarding pass schema is fully additive; ship it now and older OS users see unchanged behavior.

---
_Source: WWDC25 Session 202 page (abstract, chapter summaries, code samples, and resource links)._
