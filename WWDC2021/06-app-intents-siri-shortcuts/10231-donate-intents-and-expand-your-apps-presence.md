# Donate Intents and Expand Your App's Presence
**WWDC21 · Session 10231** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10231/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, watchOS 8

## Overview
This session explains how intent donations connect app actions to system-level intelligence features: Siri Suggestions, Spotlight, Focus, the Smart Stack (including Widget Suggestions new in iOS 15), Shortcuts, and the Siri Watch Face. The central concept is that every time a user performs a repeatable action in your app, you should donate an `INInteraction` to the system so that on-device ML can learn behavioral patterns and surface your app at exactly the right moment—without the user ever needing to open it.

The session walks through defining a custom intent in an Xcode intent definition file, donating it at runtime, and using the donation deletion APIs to keep the donation history relevant. A key technical lesson is covered through a counter-example: including volatile data (like a timestamp) in a donation's supported parameter combination prevents the system from recognizing repeated equivalent intents, breaking the pattern-learning pipeline.

## Key Topics
- **Intent Design Principles:** Actions should be accelerative (shortcut something the user already does), repeatable (used many times), and stateless (executable at any time without specific pre-conditions).
- **Defining Custom Intents:** Using a SiriKit Intent Definition File in Xcode to define intent parameters, custom types, supported parameter combinations for prediction, and summary strings.
- **Donating Intents:** Creating `INInteraction(intent:response:)` and calling `donate(_:)` to tell the system an action occurred. The system stores donations with context (time, location, etc.) for on-device ML.
- **Deleting Donations:** Using `INInteraction.identifier` and `groupIdentifier` to delete individual or grouped donations when they become stale or irrelevant (e.g., user deletes a saved location).
- **Smart Stack Widget Suggestions (NEW in iOS 15):** System uses intent donations matching a widget's configuration intent to proactively place the widget on top of the Smart Stack at the right time, even if not already in the stack.
- **Focus Integration (NEW in iOS 15):** `INSendMessageIntent` donations help identify people strongly associated with the user's current Focus, enabling notification allowlist suggestions.
- **Now Playing Integration (NEW in iOS 15):** `INPlayMediaIntent` donations surface in Lock screen, Control Center, and Home app Now Playing UI.
- **Structuring Donations for Success:** Supported parameter combinations in the intent definition file control which parameters the system uses for pattern matching. Exclude volatile/time-varying parameters (timestamps, dates) from supported combinations to allow the system to find consistent patterns.
- **`INUpcomingMediaManager`:** Provide a list of upcoming media content the user hasn't consumed, combined with past donations, for better media suggestions.
- **`INRelevantShortcut`:** Surface shortcuts on Siri Watch Face with time/location relevance conditions. New `widgetKind` property hints when to show the corresponding widget in a Smart Stack.

## APIs & Frameworks

**SiriKit / Intents**
- `INInteraction` – Wraps an intent for donation; has `identifier` and `groupIdentifier`
- `INInteraction.donate(_:)` – Donates the interaction to the system
- `INInteraction.delete(with: [String], completion:)` – Deletes individual donations by identifier
- `INInteraction.delete(with: String, completion:)` – Deletes a group of donations by group identifier
- `INSendMessageIntent` – Built-in; donations used for sharing suggestions and Focus allowed-contacts
- `INGetReservationDetailsIntent` – Built-in; donations used for departure time / check-in reminders
- `INPlayMediaIntent` – Built-in; donations shown in Now Playing UI in Lock screen / Control Center / Home **[NEW]**
- `INUpcomingMediaManager` – Provide list of un-consumed media content for suggestion improvement
- `INRelevantShortcut` – Surface shortcuts on Siri Watch Face with relevance conditions
- `INRelevantShortcut.widgetKind` **[NEW]** – Hints when to show corresponding widget in Smart Stack

**Widgets / WidgetKit**
- Widget Suggestions in Smart Stack **[NEW in iOS 15]** – System proactively adds widget to Smart Stack based on intent donations
- Intent-configurable widgets – Widget configuration intent matched against donations for Smart Stack intelligence

**NSUserActivity**
- Lightweight shortcut donation; integrates with Spotlight and Handoff

**Xcode Intent Definition File**
- `Intent is eligible for widgets` checkbox
- `Intent is eligible for Siri Suggestions` checkbox  
- `Intent is user-configurable in the Shortcuts app and Add to Siri` checkbox
- Supported parameter combinations – Defines which parameters the system uses for prediction/pattern-matching
- Custom parameter types (e.g., `City` with `identifier` and `displayString`)
- `Options are provided dynamically` – Parameters resolved at runtime by the app

## Code Highlights
Donating a custom intent:
```swift
let intent = CheckWeatherIntent()
intent.location = weatherLocation

let interaction = INInteraction(intent: intent, response: nil)
interaction.donate { error in
    // Handle error
}
```

Donating with identifier for later deletion:
```swift
let interaction = INInteraction(intent: intent, response: response)
interaction.identifier = "68753A44-4D6F-1226-9C60-0050E4C00067"
interaction.groupIdentifier = "san-diego"
interaction.donate { error in }

// Delete single donation
INInteraction.delete(with: ["68753A44-4D6F-1226-9C60-0050E4C00067"]) { error in }

// Delete all donations in group
INInteraction.delete(with: "san-diego") { error in }
```

## Takeaways
- Donate an intent every time the user performs a repeatable key action; the system's on-device ML learns behavioral patterns from the history to surface your app proactively.
- Never include timestamps or other per-donation-unique data in your intent's supported parameter combinations—it prevents the system from recognizing repeated equivalent actions.
- iOS 15 expands the payoff surface: Widget Suggestions in Smart Stack, Focus integration for `INSendMessageIntent`, and Now Playing UI for `INPlayMediaIntent` all draw on the same donation pipeline.
- Delete stale donations promptly using `groupIdentifier` to keep the system's model accurate and avoid surfacing irrelevant suggestions.

---
_Source: WWDC21 Session 10231 page (abstract, chapter summaries, code samples, and resource links)._
