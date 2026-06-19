# Introduction to Siri Shortcuts
**WWDC18 · Session 211** · [Watch](https://developer.apple.com/videos/play/wwdc2018/211/)

_Platforms:_ iOS 12, watchOS 5

## Overview
Siri Shortcuts — introduced in iOS 12 — allow apps to expose their functionality to Siri so the system can suggest relevant actions at the right moment, and users can assign custom voice phrases to trigger them. This session is the introductory overview covering the two adoption paths (NSUserActivity and custom INIntent), how Siri learns about actions via donation, how the Shortcuts app and voice-phrase assignment work, and the full developer workflow including the Add to Siri button.

The companion engineering sessions (Building for Voice with Siri Shortcuts, Session 214) cover the INIntent extension and Intents UI in depth.

## Key Topics

### What Siri Shortcuts Are

- Shortcuts expose specific in-app actions to the system so Siri can proactively suggest them at relevant moments (Lock Screen, Spotlight, Siri watch face, etc.).
- Users can additionally assign any shortcut a custom voice phrase in the Shortcuts app, letting them trigger the action hands-free.
- Two adoption paths:
  1. **NSUserActivity** — lightweight; uses existing activity types already adopted for Handoff and Spotlight; runs the app in the foreground.
  2. **Custom INIntent** — richer; can run in the background via an Intents extension; enables parameterized actions and custom Siri responses.

### Adoption Path 1: NSUserActivity

- Set `isEligibleForPrediction = true` on the `NSUserActivity` to opt it into Siri's suggestion engine.
- Set `suggestedInvocationPhrase` to hint a natural-language phrase to the user when they add the shortcut to Siri.
- Set `isEligibleForSearch = true` to also appear in Spotlight.
- `persistentIdentifier` — unique string identifying this activity type; used to delete stale donations.
- Mark the activity type in `NSUserActivityTypes` in `Info.plist`.
- Donation happens automatically when the activity becomes the app's `current` user activity or when `becomeCurrent()` is called.
- When Siri invokes the shortcut: the app is launched with this activity passed to `application(_:continue:restorationHandler:)` or `scene(_:continue:)`.

### Adoption Path 2: Custom INIntent

- Define a custom intent in an `.intentdefinition` file in Xcode — Xcode generates the `INIntent` subclass, `INIntentHandling` protocol, and `INIntentResponse` subclass.
- Intents can have parameters (e.g., a coffee drink type, a soup order) that Siri can resolve or that are pre-filled from the donation.
- Donate an intent by creating an `INInteraction` wrapping the intent and calling `INInteraction.donate(completion:)`.
- Shortcut invocation can happen:
  - **Foreground** — app is launched.
  - **Background** — `IntentsExtension` handles the intent silently, no UI required.
  - **Custom UI** — `IntentsUIExtension` provides a custom Siri response card.
- `INShortcut` wraps either an `NSUserActivity` or an `INIntent` as a unified shortcut type.

### Siri Suggestions and Donations

- Donation is the act of telling the system "this action was performed right now." The system learns patterns (time of day, location, device state) and proactively surfaces suggestions.
- Best practice: donate every time the user performs the action, not just once. Frequency and recency affect suggestion prominence.
- Use `NSUserActivity.persistentIdentifier` + `NSUserActivityTypes.deleteSavedUserActivities(_:completionHandler:)` to remove outdated donations (e.g., a deleted order).
- For intents: `INInteraction.deleteAll(completion:)` removes all donated interactions.

### The Add to Siri Button

- `INUIAddVoiceShortcutButton` — a pre-styled button (`INUIAddVoiceShortcutButtonStyle`) for adding a shortcut to Siri. Handles the presentation of `INUIAddVoiceShortcutViewController` internally.
- `INUIAddVoiceShortcutViewController` — the system sheet where users record a custom voice phrase. Delegate: `INUIAddVoiceShortcutViewControllerDelegate`.
- `INUIEditVoiceShortcutViewController` — shown when the user already has a voice shortcut; lets them re-record or delete it.
- Query existing voice shortcuts with `INVoiceShortcutCenter.shared.getAllVoiceShortcuts(completion:)` to show the Edit variant instead of the Add variant.

### Shortcuts App Integration

- The Shortcuts app (new in iOS 12, replaces Workflow) allows users to chain multiple shortcuts into multi-step automations.
- Parameterized INIntents work best in the Shortcuts app: Siri can prompt for parameter values when the intent is run.
- Output types from one shortcut can be chained as input to another.
- `INIntent.isEligibleForWidgets` — surface a shortcut on the widget; not all intents are appropriate here.

### Design Guidance

- Keep donated actions specific and personal — "Order my usual latte" rather than "Order coffee."
- `suggestedInvocationPhrase` should be short (3–5 words), natural, and reflect the specific action.
- Ensure the app can navigate correctly when launched from a Siri shortcut (same as Handoff / Spotlight continuation).
- Do not donate actions the user did not actually perform (e.g., browsing a product without adding to cart).

## APIs & Frameworks

**Foundation / UIKit**
- `NSUserActivity` — `isEligibleForPrediction` **[NEW]**, `isEligibleForSearch`, `suggestedInvocationPhrase` **[NEW]**, `persistentIdentifier`, `requiredUserInfoKeys`, `userInfo`
- `NSUserActivityTypes` key in `Info.plist`
- `NSUserActivity.deleteSavedUserActivities(withPersistentIdentifiers:completionHandler:)` **[NEW]**
- `NSUserActivity.deleteAllSavedUserActivities(completionHandler:)` **[NEW]**

**Intents**
- `INIntent` — base class for custom and system intents; generated subclass from `.intentdefinition`
- `INInteraction` — `init(intent:response:)`, `donate(completion:)`, `deleteAll(completion:)` **[NEW]**
- `INShortcut` — `init(userActivity:)`, `init(intent:)` **[NEW]**
- `INVoiceShortcut` — `shortcut`, `invocationPhrase` **[NEW]**
- `INVoiceShortcutCenter.shared.getAllVoiceShortcuts(completion:)` **[NEW]**

**IntentsUI**
- `INUIAddVoiceShortcutButton` **[NEW]** — system-styled Add to Siri button
- `INUIAddVoiceShortcutButtonStyle` — `.white`, `.whiteOutline`, `.black`, `.blackOutline`
- `INUIAddVoiceShortcutViewController` **[NEW]** — sheet for recording a voice phrase
- `INUIAddVoiceShortcutViewControllerDelegate` **[NEW]** — `addVoiceShortcutViewController(_:didFinishWith:error:)`, `addVoiceShortcutViewControllerDidCancel(_:)`
- `INUIEditVoiceShortcutViewController` **[NEW]** — sheet for editing/deleting an existing voice shortcut
- `INUIEditVoiceShortcutViewControllerDelegate` **[NEW]**

## Code Highlights

Donating an NSUserActivity shortcut when the user views their order:
```swift
let activity = NSUserActivity(activityType: "com.example.app.viewOrder")
activity.title = "View My Order"
activity.suggestedInvocationPhrase = "Show my order"
activity.isEligibleForPrediction = true
activity.isEligibleForSearch = true
activity.persistentIdentifier = "order-\(order.id)"
activity.userInfo = ["orderId": order.id]
view.userActivity = activity
activity.becomeCurrent()
```

Presenting the Add to Siri button in a view controller:
```swift
let shortcut = INShortcut(userActivity: activity)
let addButton = INUIAddVoiceShortcutButton(style: .black)
addButton.shortcut = shortcut
addButton.delegate = self
view.addSubview(addButton)

// Delegate
func present(_ addVoiceShortcutViewController: INUIAddVoiceShortcutViewController, for addVoiceShortcutButton: INUIAddVoiceShortcutButton) {
    addVoiceShortcutViewController.delegate = self
    present(addVoiceShortcutViewController, animated: true)
}

func addVoiceShortcutViewController(_ controller: INUIAddVoiceShortcutViewController,
                                    didFinishWith voiceShortcut: INVoiceShortcut?,
                                    error: Error?) {
    controller.dismiss(animated: true)
}
```

Donating a custom INIntent interaction:
```swift
let intent = OrderSoupIntent()
intent.soup = INObject(identifier: "tomato-soup", display: "Tomato Soup")
intent.suggestedInvocationPhrase = "Order my soup"

let interaction = INInteraction(intent: intent, response: nil)
interaction.donate { error in
    if let error = error { print("Donation error: \(error)") }
}
```

## Takeaways
- Shortcuts adoption starts with `NSUserActivity`: set `isEligibleForPrediction = true` and `suggestedInvocationPhrase` on activities you already donate for Handoff and Spotlight.
- Custom `INIntent` shortcuts enable background execution (via an Intents extension) and parameterized actions usable in the Shortcuts app — use these for actions where foreground launch is jarring.
- Donate every time the user performs an action; the system learns from frequency and context. Never donate actions the user did not perform.
- Add the `INUIAddVoiceShortcutButton` to relevant detail screens so users can easily assign a voice phrase — this is the primary adoption surface visible to users.

---
_Source: WWDC18 Session 211 page (abstract, full transcript, and resource links)._
