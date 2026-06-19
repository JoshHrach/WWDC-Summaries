# Integrate your app with Wind Down
**WWDC20 · Session 10083** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10083/)

_Platforms:_ iOS 14, watchOS 7

## Overview
Wind Down is a new feature introduced in iOS 14 as part of the Sleep experience in the Health app. It helps users prepare for sleep by surfacing quick-access shortcuts on the lock screen before bedtime. These "Wind Down Shortcuts" let users launch calming or preparatory actions with just a couple of taps, without fully unlocking their device.

Apps can participate in Wind Down by exposing their relevant actions through Siri Intents or `NSUserActivity`, using a new `shortcutAvailability` property. The system uses these declared categories to surface your app's actions during the Health app's sleep setup flow, where users can choose which shortcuts appear on their lock screen.

Developers must be selective—only actions that genuinely help users wind down (meditations, sleep audio, journaling, reviewing tomorrow's schedule) should be surfaced. Actions like starting a workout or joining a meeting are poor choices for Wind Down. Both shortcut suggestions and intent donations should be used together for best discoverability.

## Key Topics

### Wind Down Experience
- Wind Down mode surfaces shortcuts on the lock screen during a pre-sleep window configured in the Health app
- Users pick Wind Down Shortcuts from suggested app actions during the Health app sleep setup flow
- All Wind Down shortcuts also appear in a "Sleep Mode" collection in the Shortcuts app

### New `shortcutAvailability` Property
- New property on `INIntent` and `NSUserActivity` that signals which Wind Down category an action belongs to
- Categories include: mindfulness, sleep music, yoga and stretching, prepare for tomorrow, and more
- Only actions matching a category are surfaced during Wind Down setup

### Suggesting Shortcuts
- Call `INVoiceShortcutCenter.shared.setShortcutSuggestions(_:)` with an array of `INShortcut` objects
- Suggested shortcuts are shown in the Wind Down setup flow regardless of prior user activity
- Update suggestions whenever your app's relevant actions change

### Donating Intents and Activities
- Donate intents via `INInteraction.donate(completion:)` when a user performs an action in your app
- Donate `NSUserActivity` by assigning it to a view controller's `userActivity` property
- Donations help the system learn what's important and can surface shortcuts in search and lock screen suggestions

### Suggested Invocation Phrases
- Must be short, concise, and descriptive enough to identify the shortcut on the lock screen
- Example: "Play Counting Sleepy Dinosaurs" vs. the too-generic "Play sound"
- Used both as the shortcut name and the Siri voice trigger phrase

## APIs & Frameworks

- **SiriKit / Intents framework**
  - `INIntent` — base class for system and custom intents
  - `INIntent.shortcutAvailability` **[NEW]** — property to declare Wind Down category eligibility
  - `INShortcutAvailabilityOptions` **[NEW]** — enum options: `.sleepMusic`, `.mindfulness`, `.yogaAndStretching`, `.prepareForTomorrow`, etc.
  - `INPlayMediaIntent` — system intent used to initiate audio playback sessions
  - `INCreateNoteIntent` — system intent for note creation
  - `INShortcut` — wrapper that packages an intent or user activity as a shortcut
  - `INVoiceShortcutCenter` — manages shortcut suggestions with the system
  - `INVoiceShortcutCenter.shared.setShortcutSuggestions(_:)` — registers suggested shortcuts with the system
  - `INInteraction` — wraps an intent for donation
  - `INInteraction.donate(completion:)` — donates an intent interaction to the system
  - `INIntent.suggestedInvocationPhrase` — the phrase displayed on lock screen and used for Siri voice trigger
- **Foundation / UIKit**
  - `NSUserActivity` — alternative to intents for representing app actions
  - `NSUserActivity.shortcutAvailability` **[NEW]** — same Wind Down category property on user activities
  - `NSUserActivity.isEligibleForSearch` — marks activity as searchable
  - `NSUserActivity.isEligibleForPrediction` — enables system prediction of the activity
  - `NSUserActivity.suggestedInvocationPhrase` — invocation phrase for voice and display

## Code Highlights

Suggesting a Wind Down shortcut:
```swift
let intent = INPlayMediaIntent(/* ... */)
intent.shortcutAvailability = .sleepMusic
intent.suggestedInvocationPhrase = "Play Counting Sleepy Dinosaurs"
let shortcut = INShortcut(intent: intent)
INVoiceShortcutCenter.shared.setShortcutSuggestions([shortcut])
```

Donating an intent when an action is performed:
```swift
let intent = INPlayMediaIntent(/* ... */)
intent.shortcutAvailability = .sleepMusic
intent.suggestedInvocationPhrase = "Play Counting Sleepy Dinosaurs"
let interaction = INInteraction(intent: intent, response: nil)
interaction.donate(completion: nil)
```

Donating an `NSUserActivity`:
```swift
let activity = NSUserActivity(activityType: "com.example.playTrack")
activity.isEligibleForSearch = true
activity.isEligibleForPrediction = true
activity.title = "Play Sleepy Dinosaur Sounds"
activity.suggestedInvocationPhrase = "Play Sleepy Dinosaur Sounds"
activity.shortcutAvailability = .sleepMusic
viewController.userActivity = activity
```

## Takeaways
- Use the new `shortcutAvailability` property on `INIntent` or `NSUserActivity` to opt into Wind Down categories **[NEW in iOS 14]**.
- Only surface actions that genuinely support sleep preparation—poor category choices create a bad user experience.
- Both `setShortcutSuggestions` and intent donation should be used together: suggestions guarantee presence in setup; donations improve ranking and broader discoverability.
- Always set a concise `suggestedInvocationPhrase`—it is displayed on the lock screen and used as the Siri voice trigger.

---
_Source: WWDC20 Session 10083 page (abstract, transcript, and resource links)._
