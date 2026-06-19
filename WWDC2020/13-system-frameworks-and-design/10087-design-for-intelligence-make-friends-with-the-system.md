# Design for Intelligence: Make Friends with "The System"
**WWDC20 · Session 10087** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10087/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, watchOS 7

## Overview
This is the third session in the four-part "Design for Intelligence" series. Presented by multiple members of the Proactive Intelligence team (Sofiane, Mert, Chad), it explains the three foundational building blocks that every Apple intelligence technology shares: **Define, Learn, Execute**. The session illustrates each concept with concrete examples — the Soup Chef sample app for Shortcuts, Smart Stack widget rotation, and Siri Event Suggestions — and explains how they all use the same Intents framework infrastructure.

This is the most technically substantive session of the series, bridging design philosophy and API usage. It explains why `INIntent`, donations (`INInteraction`), and extensibility are all part of the same coherent system.

## Key Topics

**The Three Concepts: Define, Learn, Execute**

**1. Define**
Identify the key, repeatable actions users perform in your app. Express each as an `INIntent` with typed parameters describing all relevant attributes. Example: an "order coffee" intent with parameters for coffee type and size. Intents allow your app and the system to "speak the same language."

**2. Learn**
Donate each intent execution to the system as an `INInteraction`. Donations are signals — snapshots in time of a completed action with its parameter values. The system uses on-device machine learning to detect patterns across donations (time, location, day of week, device state, etc.) and make predictions. Donations are processed entirely on-device; the information never leaves the device.

**3. Execute**
When the system predicts a need and the user engages with a suggestion, the system reconstructs the intent and passes it back to your app for execution. Two execution paths:
- **Background execution** (preferred): perform the action without launching the foreground app; optionally display UI via `INUIAddVoiceShortcutViewController` or a custom intent UI extension
- **Foreground launch**: launch the app and navigate directly to the relevant screen with the intent parameters pre-filled

**Shortcuts — Soup Chef Example (Mert)**
The session uses the Soup Chef sample app to illustrate Shortcuts. Custom intents are defined in Xcode and appear as actions in the Shortcuts app. Users configure a shortcut by setting parameter values (or leaving them empty for Siri to ask each time). The "Add to Siri" button should appear at the point of completion — e.g., on an order confirmation screen — so users adopt it at the moment of highest intent.

Custom intents can output data to other apps: the Soup Chef "order history" intent outputs weekly order counts that can feed into a graphing app (Charty) to create a chart — demonstrating how intents compose across apps in the Shortcuts app.

Donations drive lock screen Siri Suggestions (e.g., "Order soup when leaving work on Thursdays") and automation suggestions in the Shortcuts app.

**Widgets and Smart Stack — Intelligent Rotation (Chad)**
Widgets in iOS 14 are backed by intents, allowing users to configure exactly what data to display. The Smart Stack automatically rotates to show the most relevant widget based on donations from main app usage — no additional API required. If a user consistently checks Weather for New York each morning, the stack automatically promotes the New York weather widget at that time.

**Siri Event Suggestions (Sofiane)**
When a user views a reservation in your app, donate the reservation details using `INReservation` subclasses. New in iOS 14 / macOS Big Sur: web markup for Mail and Safari lets reservation data flow in without any app code, making it accessible from web and email channels.

**Privacy as Architecture**
Donations stay on-device. The patterns the system learns from donations are computed locally using on-device intelligence. "What happens on your device stays on your device."

## APIs & Frameworks

### Intents Framework
- `INIntent` — base class for defining app actions; parameters describe the action's attributes
- `INIntentHandler` — handles incoming intents from Siri/Shortcuts at execution time
- `INInteraction` — wraps an `INIntent` with state; used to donate usage to the system
- `INInteraction.donate(completion:)` — call after each user action to record a donation
- Custom Intent Definition (`.intentdefinition` file in Xcode) — defines custom intents with typed parameters
- `INShortcut` — wraps an intent for Add to Siri / lock screen suggestion
- `INUIAddVoiceShortcutViewController` — standard "Add to Siri" sheet UI

### Shortcuts App Integration
- Intents actions in Shortcuts — custom intents appear as drag-and-drop actions
- Input/Output — intents can accept inputs from and provide outputs to other apps' intents
- Shortcut automation suggestions — derived from donation patterns; appear in Shortcuts app automatically

### WidgetKit
- `Widget` / `WidgetConfiguration` — widget declaration
- `IntentConfiguration` — widget backed by an `INIntent` for user-configurable parameters
- Smart Stack — automatically rotates using the same donation signals used for Siri Suggestions

### Siri Event Suggestions
- `INReservation` and subclasses (`INRestaurantReservation`, `INFlightReservation`, `INRentalCarReservation`, `INTrainReservation`, `INHotelReservation`) — structured reservation data
- `INGetReservationDetailsIntent` — used to donate reservation details
- Web Markup (new in iOS 14 / macOS Big Sur) — structured data in Safari and Mail for reservations without app code

### Execution
- Intents Extension (`INExtension` subclass) — handles background intent execution; can run without foregrounding app
- Intents UI Extension — displays custom UI inside Siri / Shortcuts response
- App launch with intent — system calls `application(_:continue:restorationHandler:)` or scene-based equivalent with the reconstructed `NSUserActivity`

## Code Highlights

Defining a custom intent in code (conceptual; actual definition is in `.intentdefinition`):
```swift
// Generated from Xcode's .intentdefinition
class OrderCoffeeIntent: INIntent {
    var coffeeType: CoffeeItem?
    var size: CoffeeSize?
}
```

Donating an intent interaction after a user action:
```swift
let intent = OrderCoffeeIntent()
intent.coffeeType = CoffeeItem(identifier: "iced-latte", display: "Iced Latte")
intent.size = CoffeeSize(identifier: "large", display: "Large")

let interaction = INInteraction(intent: intent, response: nil)
interaction.donate { error in
    if let error = error {
        print("Donation failed: \(error)")
    }
}
```

Adding an "Add to Siri" button at the point of task completion:
```swift
let shortcut = INShortcut(intent: intent)
let addVC = INUIAddVoiceShortcutViewController(shortcut: shortcut)
addVC.delegate = self
present(addVC, animated: true)
```

## Takeaways
- Define your app's key repeatable actions as custom intents with typed parameters — this is the contract between your app and the intelligent system, and everything else (Shortcuts, lock screen suggestions, Smart Stack rotation) builds on it.
- Donate an `INInteraction` every time a user performs an action — even if the user never sets up a Shortcut, donations are how the system learns patterns and generates automatic Siri Suggestions without any user configuration.
- Place the "Add to Siri" button at the moment of task completion (e.g., order confirmation screen), not buried in Settings — this is where user intent is highest.
- Prefer background execution over app launch when handling incoming intents — it reduces friction and lets users accomplish tasks without interrupting what they're doing.

---
_Source: WWDC20 Session 10087 page (abstract and full transcript)._
