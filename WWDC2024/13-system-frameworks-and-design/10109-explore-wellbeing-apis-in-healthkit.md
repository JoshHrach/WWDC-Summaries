# Explore Wellbeing APIs in HealthKit
**WWDC24 · Session 10109** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10109/)

_Platforms:_ iOS 18, iPadOS 18, watchOS 11, visionOS 2

## Overview
HealthKit gains three new mental wellbeing data types: **State of Mind** (representing momentary emotions and daily moods), **Depression Risk** (derived from PHQ-9 questionnaire results), and **Anxiety Risk** (derived from GAD-7 questionnaire results). Apps can read and write all three, enabling anything from simple emoji-based mood logging after a calendar event to clinical-grade screening tools.

The session grounds the API design in emotion science: reflecting on feelings — naming them specifically — provides real clinical benefit regardless of whether the feeling is pleasant or unpleasant. The more specific the label, the more benefit. The demo builds a calendar app that lets users log how each event felt and then surfaces personalized insights (Calendar Quality score, Most Meaningful Event) by querying historical State of Mind samples.

## Key Topics

### State of Mind
`HKStateOfMind` captures a person's emotional state with four properties: **kind** (momentary emotion or daily mood), **valence** (continuous scale from -1 to +1 representing unpleasant to pleasant), **labels** (qualitative descriptors like Passionate, Overwhelmed, Relieved, Happy, Calm), and **associations** (life-area tags like Work, Family, Hobbies, Identity). Momentary emotions are short-lived (seconds to minutes); daily moods last hours or days. Apps write samples via the standard `HKHealthStore.save()` method.

### Depression Risk (PHQ-9) and Anxiety Risk (GAD-7)
The GAD-7 (7 questions, anxiety) and PHQ-9 (9 questions, depression) are standardized Pfizer questionnaires used worldwide for mental health screening. New API lets apps read and write results from these assessments to HealthKit, enabling users to track treatment efficacy or save results from a doctor's office. These must be presented in accordance with Pfizer's standards.

### State of Mind Predicates
Four new predicates enable targeted queries: by **kind** (emotion vs. mood), by **valence** (how pleasant), by **label** (e.g., `.happy`), and by **association** (e.g., `.work`). These compose with `NSCompoundPredicate` for complex queries. Results come back via `HKSampleQueryDescriptor` or `HKAnchoredObjectQueryDescriptor`.

### Demo: Calendar Quality and Most Meaningful Event
The demo shows two insights powered by State of Mind queries. "Calendar Quality" fetches all samples in a date range matching the app's calendar associations, averages their valence (shifted 0–2 range for percentage), and displays it alongside the existing Work-Life Balance score. "Most Meaningful Event" queries for the highest-valence `.happy`-labeled sample and finds the nearest calendar event — replacing naive "longest event" heuristics with feelings-based significance.

## APIs & Frameworks

**HealthKit**
- `HKStateOfMind` **[NEW]** — new sample type for mood and emotion
  - `init(date:kind:valence:labels:associations:)` **[NEW]**
  - `kind: HKStateOfMind.Kind` — `.momentaryEmotion` or `.dailyMood` **[NEW]**
  - `valence: Double` — range -1.0 to +1.0 **[NEW]**
  - `labels: [HKStateOfMind.Label]` — exhaustive set including `.happy`, `.calm`, `.stressed`, `.overwhelmed`, `.relieved`, `.passionate`, `.satisfied`, `.angry`, `.sad`, `.indifferent` and many more **[NEW]**
  - `associations: [HKStateOfMind.Association]` — `.work`, `.family`, `.hobbies`, `.identity`, `.dating`, `.health`, `.money`, `.partner`, `.selfCare`, `.tasks` and more **[NEW]**
- `HKStateOfMind.Kind` enum **[NEW]**
- `HKStateOfMind.Label` enum **[NEW]**
- `HKStateOfMind.Association` enum **[NEW]**
- `HKQuery.predicateForStatesOfMind(with:)` — filter by `Association` **[NEW]**
- `HKQuery.predicateForStatesOfMind(with:)` — filter by `Label` **[NEW]**
- `HKSamplePredicate.stateOfMind(_:)` — wraps an `NSPredicate` for use in query descriptors **[NEW]**
- `HKSampleQueryDescriptor(predicates:sortDescriptors:)` — used to execute State of Mind queries (existing, new predicate support)
- `HKAnchoredObjectQueryDescriptor(predicates:anchor:)` — alternative query for State of Mind (existing)
- Depression Risk / Anxiety Risk types (read/write PHQ-9 and GAD-7 results) **[NEW]**
- `HKHealthStore.save(_:)` async — saves `HKStateOfMind` samples (existing)

**HealthKitUI**
- `healthDataAccessRequest(store:shareTypes:readTypes:trigger:completion:)` — SwiftUI view modifier for HealthKit authorization (existing)

**Sample App**
- "Visualizing HealthKit State of Mind in visionOS" — linked sample code

## Code Highlights

Create and save a State of Mind sample for a calendar event:
```swift
func createSample(for event: EventModel, emojiType: EmojiType) -> HKStateOfMind {
    let kind: HKStateOfMind.Kind = .momentaryEmotion
    let valence: Double = emojiType.valence
    let label = emojiType.label
    let association = event.association
    return HKStateOfMind(date: event.endDate,
                         kind: kind,
                         valence: valence,
                         labels: [label],
                         associations: [association])
}

func save(sample: HKSample, healthStore: HKHealthStore) async {
    do { try await healthStore.save(sample) }
    catch { /* handle error */ }
}
```

Query samples by association to compute Calendar Quality:
```swift
let associationsPredicate = NSCompoundPredicate(
    orPredicateWithSubpredicates: associations.map {
        HKQuery.predicateForStatesOfMind(with: $0)
    }
)
let compoundPredicate = NSCompoundPredicate(
    andPredicateWithSubpredicates: [datePredicate, associationsPredicate]
)
let stateOfMindPredicate = HKSamplePredicate.stateOfMind(compoundPredicate)
let descriptor = HKSampleQueryDescriptor(predicates: [stateOfMindPredicate], sortDescriptors: [])
let results: [HKStateOfMind] = try await descriptor.result(for: healthStore)

// Shift valence from -1...1 to 0...2, compute average as percentage
let adjustedValenceResults = results.map { $0.valence + 1.0 }
let averageAdjustedValence = adjustedValenceResults.reduce(0.0, +) / Double(results.count)
let adjustedValenceAsPercent = Int(100.0 * (averageAdjustedValence / 2.0))
```

Query for happiest moment (Most Meaningful Event):
```swift
let label: HKStateOfMind.Label = .happy
let labelPredicate = HKQuery.predicateForStatesOfMind(with: label)
let compoundPredicate = NSCompoundPredicate(
    andPredicateWithSubpredicates: [datePredicate, associationPredicate, labelPredicate]
)
let stateOfMindPredicate = HKSamplePredicate.stateOfMind(compoundPredicate)
let descriptor = HKAnchoredObjectQueryDescriptor(predicates: [stateOfMindPredicate], anchor: nil)
let results = descriptor.results(for: healthStore)
let samples: [HKStateOfMind] = try await results.reduce([]) { $1.addedSamples }

let happiestSample = samples.max { $0.valence < $1.valence }
let happiestEvent = findClosestEvent(startDate: happiestSample?.startDate,
                                     endDate: happiestSample?.endDate)
```

## Takeaways
- Use `.momentaryEmotion` for post-event check-ins (after a meeting, workout, or social event) and `.dailyMood` for end-of-day reflection — context determines which kind is appropriate.
- Populate both `labels` and `associations` for maximum expressivity; these power the Health app's trend views and make your own queries more precise.
- State of Mind is not only for dedicated mental health apps — any app where a person may pause and reflect (calendar, fitness, journaling, social) can benefit from this data.
- The four new predicates (kind, valence, label, association) compose with `NSCompoundPredicate` to build precise, clinically meaningful queries without fetching excess data.

---
_Source: WWDC24 Session 10109 page (abstract, chapter summaries, code samples, and resource links)._
