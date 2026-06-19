# Feature Your Actions in the Shortcuts App
**WWDC20 · Session 10084** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10084/)

_Platforms:_ iOS 14, iPadOS 14, watchOS 7

## Overview
This session explains how to get an app's actions featured throughout the Shortcuts app — in automation suggestions, the Shortcuts Gallery, and the Shortcuts Editor. The core mechanism for all three surfaces is donation: calling `INInteraction.donate()` whenever a user performs a significant action in the app teaches the system to suggest that action in the right context.

iOS 14 extends automation suggestions — previously limited to Daily Routines around commute — to support all actions including custom intents, system intents like `INPlayMediaIntent`, and user activities. The system learns from donation patterns and proposes personalized automations (e.g., "Practice French every weekday at 9 PM") without requiring any additional developer work beyond donating.

Two specific surfaces in the Shortcuts app are covered: the Gallery's "Shortcuts from Your Apps" section (populated by `INVoiceShortcutCenter.setShortcutSuggestions`) and the Editor's actions panel (populated by donations, suggestions, configurable intents, and a small set of always-present system intents).

## Key Topics

**Automation Suggestions (iOS 14 — New)**
- Extended in iOS 14 to support all action types: custom intents, system intents, and `NSUserActivity` **[NEW]**
- System learns from `INInteraction.donate()` calls and correlates them with time, location, Bluetooth events, etc.
- Automation types: time of day, arrive/leave location, connect Bluetooth device, connect CarPlay, receive message, open app, and more
- Developer action required: only donation — no additional API

**Daily Routines**
- Personalized step-by-step flows for "Going to Work," "Going Home," and "At the Gym"
- Media apps appear in Daily Routines by adopting and donating `INPlayMediaIntent`
- Workout apps appear by adopting `INStartWorkoutIntent`
- System auto-correlates media playback donations with commute patterns to surface in the flow

**Shortcuts Gallery — "Shortcuts from Your Apps"**
- Two mechanisms for appearing in gallery:
  1. `INVoiceShortcutCenter.setShortcutSuggestions(_:)` — explicit programmatic suggestions
  2. Donations — system surfaces recently used and predicted actions
- Combine both for best coverage

**Shortcuts Editor**
- Suggested actions panel: populated by donations (recent, personalized) and `setShortcutSuggestions`
- Intents marked as configurable appear in the Editor even without donations or suggestions
- Key parameters enable parameter-option rows (e.g., "Run Five Miles", "Study French")
- "Apps" button shows all per-app available actions
- Always-present system intents (no donation required): `INSendPaymentIntent`, `INRequestPaymentIntent`, `INRequestRideIntent`

## APIs & Frameworks

### Intents / SiriKit
- `INInteraction` — wraps an intent for donation
- `INInteraction.donate(completion:)` — registers the action with the system for suggestions **[KEY]**
- `INIntent` — base class for all intents
- `INObject` — typed value for intent parameters (e.g., representing a soup item)
- `INImage` — image associated with a parameter for display in Shortcuts UI
- `INShortcut` — wraps an intent or user activity for use with `INVoiceShortcutCenter`
- `INVoiceShortcutCenter.shared.setShortcutSuggestions(_:)` — sets explicit shortcut suggestions shown in Gallery and Editor
- `INPlayMediaIntent` — system intent; enables media app to appear in Daily Routines "Going Home/Work" flow
- `INStartWorkoutIntent` — system intent; enables workout app to appear in "At the Gym" Daily Routine
- `INSendPaymentIntent` — always shows in Editor without donation
- `INRequestPaymentIntent` — always shows in Editor without donation
- `INRequestRideIntent` — always shows in Editor without donation

### NSUserActivity
- `NSUserActivity` — alternative to intents for donation; supported in automation suggestions
- Wrap in `INShortcut` for `setShortcutSuggestions` use

## Code Highlights

Donating a custom intent during app usage:
```swift
// When user places a soup order
let intent = PlaceOrderIntent()
let soup = order.soup
intent.soup = INObject(identifier: soup.id, display: soup.name)
intent.soup?.setImage(INImage(named: "soup-icon"), forParameterNamed: \.soup)
intent.deliveryLocation?.setImage(INImage(named: "location-icon"), forParameterNamed: \.deliveryLocation)
let interaction = INInteraction(intent: intent, response: nil)
interaction.donate { error in
    // Handle error if needed
}
```

Setting explicit shortcut suggestions for the Gallery:
```swift
let orderStatusShortcut = INShortcut(intent: OrderStatusIntent())
let topSoupsShortcut = INShortcut(userActivity: TopSoupsUserActivity)
INVoiceShortcutCenter.shared.setShortcutSuggestions([orderStatusShortcut, topSoupsShortcut])
```

## Takeaways
- Donation via `INInteraction.donate()` is the single most impactful action — it enables automation suggestions, Gallery curation, and Editor suggestions all at once.
- iOS 14 extends automation suggestions to all custom and system intents; media apps should adopt `INPlayMediaIntent` and workout apps `INStartWorkoutIntent` to appear in Daily Routines flows.
- Use `INVoiceShortcutCenter.setShortcutSuggestions` in addition to donations to guarantee a baseline set of actions always appears in the Gallery and Editor.
- Mark intents as "configurable" and define key parameters to ensure they surface in the Editor with parameter-option rows, even before the user has donated those specific interactions.

---
_Source: WWDC20 Session 10084 page (abstract, chapter summaries, code samples, and resource links)._
