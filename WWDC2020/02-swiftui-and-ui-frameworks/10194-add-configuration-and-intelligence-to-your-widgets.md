# Add Configuration and Intelligence to Your Widgets
**WWDC20 · Session 10194** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10194/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11

## Overview
This session builds on the foundational WidgetKit introduction to cover two advanced topics: configurable widgets (using SiriKit intents to power the widget configuration UI) and system intelligence (Smart Stacks that automatically surface the right widget at the right time). Together these features let users personalize their Home screen and let the system proactively show relevant information.

Configurable widgets replace `StaticConfiguration` with `IntentConfiguration`. Intent parameters drive the widget edit UI shown on the back of the widget — supporting strings, booleans, numbers, contacts, locations, enumerations, and custom types with dynamic options fetched from the app. Fixed-size arrays (new in iOS 14) let a widget define exactly how many items a user can select per widget size.

System intelligence for Smart Stacks works through two complementary mechanisms: behavior-based predictions driven by `INInteraction` donations from the host app, and relevance-based surfacing driven by `TimelineEntryRelevance` scores supplied alongside `TimelineEntry` objects in WidgetKit.

## Key Topics
- **`IntentConfiguration`** — Replaces `StaticConfiguration`; takes an intent type as a generic parameter; the intent's parameters become the widget's configuration UI rows.
- **Intent Definition File** — Declared in Xcode; drives both widget configuration UI and Shortcuts/Siri support; compiling into the app lets the system read intent metadata.
- **`IntentTimelineProvider`** — Replaces `TimelineProvider`; provider methods receive the configured intent instance at runtime so the widget knows what to display.
- **Parameter types** — String (text field), Bool (switch), Int (stepper/number field), Decimal (slider/number field), Person (contact picker), Location (location picker), Enum (list), custom types with dynamic options.
- **Dynamic options** — Checked in Intent Definition File; the system calls the app's intent handler (`provideXOptionsCollection`) at configuration time; supports flat lists, sectioned lists, and real-time search results as the user types.
- **Fixed-size arrays** **[NEW]** — Intent parameters can define a fixed number of selectable items; configurable per widget size in the intent editor.
- **UI customization** — `configurationDisplayName` and `description` SwiftUI modifiers; accent color and background color via named colors in the widget extension's asset catalog (`Global Accent Color Name`, `Widget Background Color Name` build settings).
- **Parameter dependencies** — Parameters can be shown/hidden based on the value of a parent parameter (e.g., show a "Calendar" picker only when "Mirror Calendar App" is off).
- **Intent donations for Smart Stacks** — Host app calls `INInteraction(intent:response:).donate(completion:)` when users view information; the system uses donations to predict when to surface matching widgets.
- **Supported Combinations** — Subset of intent parameters that the system uses for prediction and widget matching; controls which widget configurations are surfaced by a given donation.
- **`TimelineEntryRelevance`** **[NEW]** — Accompanies `TimelineEntry`; `score` (float, relative to app's own entries; 0 or below = not relevant) and `duration` (how long the score applies); lets the system surface a widget when it has timely, important information.

## APIs & Frameworks

### WidgetKit
- **`IntentConfiguration`** **[NEW]** — `init(kind:intent:provider:content:)` — configurable widget configuration
- **`IntentTimelineProvider`** **[NEW]** — Protocol; `getSnapshot(for:in:completion:)` and `getTimeline(for:in:completion:)` receive the configured intent
- **`TimelineEntry`** — Protocol; `date: Date`, optional `relevance: TimelineEntryRelevance?`
- **`TimelineEntryRelevance`** **[NEW]** — `init(score: Float, duration: TimeInterval)`; `score: Float`, `duration: TimeInterval`
- **`StaticConfiguration`** — Unchanged; now the non-configurable counterpart to `IntentConfiguration`
- **`configurationDisplayName(_:)`** — SwiftUI modifier on widget body
- **`description(_:)`** — SwiftUI modifier on widget body

### Intents / SiriKit
- **`INInteraction`** — `init(intent:response:)`; `.donate(completion:)` — donates widget configuration intent from host app
- **Intent Definition File** — Xcode `.intentdefinition` file; "Intent is eligible for widgets" checkbox; "Intent is eligible for Siri Suggestions" checkbox; "Supported Combinations" list
- **Intent handler protocol** (generated) — e.g., `ViewRecentPurchasesIntentHandling`; `provideCardOptionsCollection(for:completion:)`, `defaultCard(for:completion:)`
- **`INObjectCollection`** — `init(items:)` or `init(sections:)`; used to return dynamic option lists from intent handlers
- **`INObjectSection`** — Groups items in a dynamic options list into labeled sections

## Code Highlights

Switch from `StaticConfiguration` to `IntentConfiguration`:
```swift
// Before:
StaticConfiguration(kind: "RecentPurchases", provider: Provider()) { entry in
    WidgetView(entry: entry)
}
// After:
IntentConfiguration(kind: "RecentPurchases",
                    intent: ViewRecentPurchasesIntent.self,
                    provider: IntentProvider()) { entry in
    WidgetView(entry: entry)
}
```

Donate an intent from the host app to inform Smart Stack predictions:
```swift
let intent = ViewRecentPurchasesIntent()
intent.card = selectedCard
let interaction = INInteraction(intent: intent, response: nil)
interaction.donate { error in
    if let error { print("Donation failed: \(error)") }
}
```

Provide relevance for a high-value transaction:
```swift
let relevance = TimelineEntryRelevance(score: 1.0, duration: 0)
let entry = PurchaseEntry(date: .now, purchase: largePurchase, relevance: relevance)
```

## Takeaways
- Replace `StaticConfiguration` with `IntentConfiguration` and `IntentTimelineProvider` to make widgets user-configurable; the system builds the configuration UI automatically from the intent's parameters.
- Use dynamic options (`provideXOptionsCollection`) for parameters whose values come from app data (accounts, contacts, calendars, etc.); support real-time search for large datasets.
- Donate `INInteraction` objects from the host app whenever a user views a piece of content; the system uses these to predict when to rotate a matching widget to the top of a Smart Stack.
- Supply `TimelineEntryRelevance` with a non-zero `score` on `TimelineEntry` objects when the widget has timely, high-value information; a score of 0 signals no relevance.

---
_Source: WWDC20 Session 10194 page (abstract, chapter summaries, code samples, and resource links)._
