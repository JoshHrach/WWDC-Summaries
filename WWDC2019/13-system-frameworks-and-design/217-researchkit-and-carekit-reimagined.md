# ResearchKit and CareKit Reimagined
**WWDC19 · Session 217** · [Watch](https://developer.apple.com/videos/play/wwdc2019/217/)

_Platforms:_ iOS 13

## Overview
ResearchKit and CareKit are Apple's open-source frameworks for building health research and patient care apps. This session covers significant updates to both frameworks: ResearchKit receives new visual health active tasks (visual acuity and contrast sensitivity) and enhanced speech recognition results, while CareKit 2.0 is completely rewritten in Swift with a new modular architecture, redesigned UI components, and a reactive synchronization engine built on the Combine framework.

CareKit 2.0 is split into three independently usable sub-frameworks — CareKit UI, CareKit Store, and CareKit itself — each addressing a distinct layer of a care plan app. The new architecture enables powerful customization: any conforming data store and any UI view can be plugged into the synchronization pipeline.

## Key Topics

### ResearchKit UI Improvements
- Updated card styling with an intra-step progress indicator and a "Learn More" button for contextual definitions.
- `ORKLearnMoreStep` with an associated `ORKLearnMoreItem` adds a detail disclosure sheet to any step.
- All steps now support a `topContentImageView` for rich media display.
- `ORKBodyItem` enables composed, automatically formatted lists within steps (bullet point or plain text styles).
- Updated `ORKFormItem` initializer accepts section title, detail text, learn-more item, and a progress indicator toggle for grouping form items under sections.

### ResearchKit Active Tasks — Vision
- **Visual Acuity (Landolt C):** `ORKLandoltCStep` with `testType = .acuity` — user aligns dial to the letter C opening; produces a visual acuity rating.
- **Visual Acuity (Tumbling E):** `ORKTumblingEStep` — user swipes in the direction of the letter E arms; testing distance controlled in real-time by TrueDepth camera; produces MAR (Minimum Angle of Resolution) score.
- **Contrast Sensitivity (Landolt C):** `ORKLandoltCStep` with `testType = .contrastSensitivity` — fixed size, decreasing contrast.
- **Contrast Sensitivity (Gabor Patch):** `ORKCSFStep` — adaptive algorithm varies spatial frequency; TrueDepth camera controls distance; produces contrast sensitivity function curve.
- TrueDepth-based tasks available via Apple sample code license on developer.apple.com.

### ResearchKit Active Tasks — Hearing & Speech
- Across-the-board algorithm improvements to tone audiometry, speech-in-noise, and SPL meter tasks.
- Results from hearing tasks can now be written directly to HealthKit (new HealthKit data types).
- Speech recognition task now exposes enhanced `SFTranscription` properties: speaking rate, average pause duration.
- New `SFVoiceAnalytics` object available from speech recognition results.

### CareKit 2.0 Architecture **[NEW]**
- Completely rewritten in Swift; leverages Combine for reactive data propagation.
- Three independently compilable sub-frameworks:
  - **CareKit UI** — pre-built `UIView` subclasses for tasks, charts, and contacts.
  - **CareKit Store** — Core Data-backed persistence layer for care plans.
  - **CareKit** — synchronization engine tying UI and store together via a synchronizer object.

### CareKit UI Components **[NEW]**
- **Tasks:** `OCKSimpleTaskView`, `OCKInstructionsTaskView`, `OCKGridTaskView` (exposes a `UICollectionView`), `OCKChecklistTaskView` (stack-view-based checklist), `OCKSimpleLogTaskView` (event logging with timestamps).
- **Charts:** `OCKCartesianChartCardView` — supports multiple chart types; toggle type to switch visualization for the same data.
- **Contacts:** `OCKContactCardView` — auto-constrained contact card.
- All views are plain `UIView` subclasses — can be placed anywhere in an existing app hierarchy.

### CareKit Store **[NEW]**
- `OCKStore` — wraps Core Data; persisted within the app container.
- Core entities: `OCKPatient`, `OCKCarePlan`, `OCKContact`, `OCKTask`, `OCKSchedule`, `OCKOutcome`, `OCKOutcomeValue`, `OCKNote`.
- `OCKSchedule` — composable schedule elements (e.g., daily at 7 AM + every-other-day at noon merged into one schedule).
- Tasks and schedules are versionable — updates are persisted with history.
- `fetchInsights(for:query:dailyAggregator:completion:)` — aggregates outcomes over a date range; `dailyAggregator` block called per day, `completion` returns computed values for charting.
- Returns Swift `Result` types in all async callbacks.

### CareKit Synchronization Engine **[NEW]**
- `OCKSynchronizer` — propagates UI events to the store, then publishes store updates back to all subscribed UI views via Combine streams.
- The data store layer is protocol-based (`OCKStore` protocol) — swap in any conforming store (CloudKit, custom backend, etc.).
- UI views are bound to store entities by the synchronizer; custom `UIView` subclasses can participate by describing their bindings.
- Views update independently and asynchronously upon store changes.

## APIs & Frameworks

### ResearchKit **[UPDATED]**
- `ORKLearnMoreStep` — step with detail disclosure **[NEW]**
- `ORKLearnMoreItem` — triggers detail sheet; initializers accept a step or plain text (hyperlink) **[NEW]**
- `ORKBodyItem` — formatted list item for steps (bullet or text style) **[NEW]**
- `ORKFormItem` — updated initializer with section grouping support **[UPDATED]**
- `ORKLandoltCStep` — visual acuity / contrast sensitivity (Landolt C) **[NEW]**
- `ORKTumblingEStep` — visual acuity with TrueDepth (Tumbling E) **[NEW]**
- `ORKCSFStep` — contrast sensitivity function (Gabor Patch) **[NEW]**
- `ORKOrderedTask` — composes steps into a task
- `ORKTaskViewController` — presents tasks

### Speech Framework
- `SFTranscription` — speaking rate, average pause duration properties **[NEW]**
- `SFVoiceAnalytics` — voice analysis object from speech recognition **[NEW]**

### CareKit UI **[NEW in CareKit 2.0]**
- `OCKSimpleTaskView`
- `OCKInstructionsTaskView`
- `OCKGridTaskView`
- `OCKChecklistTaskView`
- `OCKSimpleLogTaskView`
- `OCKCartesianChartCardView`
- `OCKContactCardView`

### CareKit Store **[NEW in CareKit 2.0]**
- `OCKStore` — Core Data wrapper
- `OCKPatient`, `OCKCarePlan`, `OCKContact`, `OCKTask`, `OCKSchedule`, `OCKScheduleElement`
- `OCKOutcome`, `OCKOutcomeValue`, `OCKNote`
- `OCKStore` protocol — conformance enables custom backends
- `addPatient(_:callbackQueue:completion:)`, `addCarePlan(_:callbackQueue:completion:)`, `addTask(_:callbackQueue:completion:)` — async ingestion methods
- `fetchInsights(for:query:dailyAggregator:completion:)` — outcome aggregation

### CareKit (Synchronization) **[NEW in CareKit 2.0]**
- Combine framework integration for reactive streams
- Synchronizer object — UI↔Store event propagation

## Code Highlights

Creating a ResearchKit learn-more step:
```swift
let learnMoreStep = ORKLearnMoreStep(identifier: "learnMore")
learnMoreStep.topContentImage = UIImage(named: "info")
let bodyItem = ORKBodyItem(text: "Details", detailedText: nil, image: nil, learnMoreItem: nil, isBulleted: true)
learnMoreStep.bodyItems = [bodyItem]
let learnMoreItem = ORKLearnMoreItem(text: nil, learnMoreInstructionStep: learnMoreStep)
```

Creating a Tumbling E visual acuity task:
```swift
let step = ORKTumblingEStep(identifier: "tumblingE")
step.minimumViewingDistance = 0.3
step.maximumViewingDistance = 0.5
let task = ORKOrderedTask(identifier: "visualTask", steps: [step])
let vc = ORKTaskViewController(task: task, taskRun: nil)
present(vc, animated: true)
```

Adding a patient to CareKit Store:
```swift
import CareKitStore
let store = OCKStore(name: "careStore")
let patient = OCKPatient(identifier: "p1", givenName: "Jane", familyName: "Doe")
store.addPatient(patient) { result in
    switch result {
    case .success(let saved): print(saved)
    case .failure(let error): print(error)
    }
}
```

## Takeaways
- CareKit 2.0 is a complete Swift rewrite with Combine-driven reactive sync — existing CareKit apps should plan a migration.
- CareKit UI, CareKit Store, and CareKit can each be used independently, enabling incremental adoption.
- ResearchKit's new vision tasks (Landolt C, Tumbling E, Gabor Patch) extend clinical-grade ophthalmic testing to iOS, with TrueDepth-controlled testing distances.
- Hearing task results now write directly to HealthKit; speech tasks expose richer analytics via `SFVoiceAnalytics`.

---
_Source: WWDC19 Session 217 page (abstract, chapter summaries, code samples, and resource links)._
