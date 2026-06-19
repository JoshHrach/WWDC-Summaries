# Enhanced Suggestions for Your Journaling App
**WWDC24 · Session 10209** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10209/)

_Platforms:_ iOS 18

## Overview
The Journaling Suggestions API receives four new suggestion types in iOS 18: `StateOfMind`, `MotionActivity`, `GenericMedia`, and `Reflection`. These additions let journaling apps surface richer, more contextual prompts — from emotional check-ins backed by HealthKit data, to movement tracking and media listening history, to structured reflection prompts with visual styling. The session also adds landscape orientation support automatically across all devices.

Journaling Suggestions uses privacy-preserving on-device processing; no raw health or activity data is surfaced directly — the API returns structured, anonymized suggestion objects that the app can present without reading underlying sensor data.

## Key Topics

### State of Mind
`JournalingSuggestion.StateOfMind` wraps a `HKStateOfMind` HealthKit sample. The suggestion exposes:
- `valence: Double` — a -1 to 1 score (-1 = unpleasant, +1 = pleasant)
- `labels: [HKStateOfMind.Label]` — qualitative descriptors (e.g., `.excited`, `.calm`, `.anxious`)
- `associations: [HKStateOfMind.Association]` — what the mood is linked to (e.g., `.work`, `.family`)
- `kind` — whether it is a daily summary or a momentary check-in

Apps can display the `valence` value as a color gradient and let users fine-tune and save it back to HealthKit. See the companion "Explore wellbeing APIs in HealthKit" session for the write-back API.

### Motion Activity
`JournalingSuggestion.MotionActivity` now includes `movementType: JournalingSuggestion.MotionActivity.MovementType`, which distinguishes `.walk`, `.run`, and `.mixed` workouts. Previously only step-count summary data was available. Also exposes `steps`, `distance`, and `duration` for display.

### Generic Media
`JournalingSuggestion.GenericMedia` covers media types beyond Apple Music — any audio or video a user consumed through a supported app. Properties: `title: String`, `artistName: String?`, `albumName: String?`, `appIcon: UIImage?`. Apps render these in a standard media suggestion card; the `appIcon` conveys the source app.

### Reflection
`JournalingSuggestion.Reflection` is a structured writing prompt. It carries a `prompt: String` (the displayed question) and a `backgroundColor: UIColor` (a suggested card background). The app renders the prompt text and, optionally, applies the background color for visual consistency with the system Journaling experience.

### Landscape Support
No code changes needed — the `JournalingPickerViewController` automatically adapts to landscape on iPhone and all iPad orientations in iOS 18.

## APIs & Frameworks

**JournalingSuggestions**
- `JournalingPickerViewController` (existing) — now supports landscape automatically
- `JournalingSuggestion.Content` protocol (existing)
- `JournalingSuggestion.StateOfMind` **[NEW]**
  - `.valence: Double` (range -1 to 1)
  - `.labels: [HKStateOfMind.Label]`
  - `.associations: [HKStateOfMind.Association]`
  - `.kind: HKStateOfMind.Kind`
- `JournalingSuggestion.MotionActivity` **[updated]**
  - `.movementType: JournalingSuggestion.MotionActivity.MovementType` **[NEW]**
  - `MovementType` cases: `.walk`, `.run`, `.mixed` **[NEW]**
  - `.steps: Int`
  - `.distance: Measurement<UnitLength>`
  - `.duration: TimeInterval`
- `JournalingSuggestion.GenericMedia` **[NEW]**
  - `.title: String`
  - `.artistName: String?`
  - `.albumName: String?`
  - `.appIcon: UIImage?`
- `JournalingSuggestion.Reflection` **[NEW]**
  - `.prompt: String`
  - `.backgroundColor: UIColor`

**HealthKit (companion)**
- `HKStateOfMind` (existing, surfaced through `StateOfMind` suggestion)
- `HKStateOfMind.Label` (existing)
- `HKStateOfMind.Association` (existing)
- `HKStateOfMind.Kind` (existing)

## Code Highlights

```swift
// Handle State of Mind suggestion
func picker(_ picker: JournalingPickerViewController,
            didSelectSuggestion suggestion: JournalingSuggestion) {
    for content in suggestion.items {
        if let stateOfMind = content as? JournalingSuggestion.StateOfMind {
            displayValenceGradient(valence: stateOfMind.valence)
            displayLabels(stateOfMind.labels)
        }
        if let media = content as? JournalingSuggestion.GenericMedia {
            entryBuilder.addMediaCard(title: media.title, icon: media.appIcon)
        }
        if let reflection = content as? JournalingSuggestion.Reflection {
            entryBuilder.setPrompt(reflection.prompt,
                                   backgroundColor: reflection.backgroundColor)
        }
        if let motion = content as? JournalingSuggestion.MotionActivity {
            entryBuilder.addMovementSummary(type: motion.movementType,
                                            steps: motion.steps,
                                            distance: motion.distance)
        }
    }
}
```

## Takeaways
- Render `StateOfMind.valence` as a gradient color bar to visually communicate mood without showing raw numbers.
- Use `GenericMedia.appIcon` to clearly attribute the source app so users remember where they heard or watched the content.
- Display `Reflection.backgroundColor` as a subtle card tint — it's chosen to complement the prompt's emotional tone.
- No sensor data is exposed; adopt these types without requesting HealthKit or fitness entitlements specifically for suggestions.

---
_Source: WWDC24 Session 10209 page (abstract, chapter summaries, code samples, and resource links)._
