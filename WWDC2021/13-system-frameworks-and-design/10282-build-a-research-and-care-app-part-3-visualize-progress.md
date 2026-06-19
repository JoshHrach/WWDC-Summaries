# Build a Research and Care App, Part 3: Visualize Progress
**WWDC21 · Session 10282** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10282/)

_Platforms:_ iOS 15, iPadOS 15

## Overview
The final session in the three-part "Recover" code-along completes the app with three additions: enhanced survey task cards that display the answers participants gave, CareKit charts plotting pain/sleep check-in data alongside range-of-motion progress, and a 3D anatomical model viewer using ResearchKit's `ORK3DModelStep` with USDZ assets. It also covers restricting editing so participants cannot modify past outcomes or pre-answer future surveys.

Together, the three sessions show a complete path from onboarding consent → scheduled task collection → data visualization — using ResearchKit for UI and CareKit for scheduling, persistence, and charting.

## Key Topics

### Custom View Synchronizers for Task Cards
- CareKit task cards are rendered by view synchronizers — subclass `OCKSurveyTaskViewSynchronizer` to customize card appearance
- Override `updateView(_:context:)`: always call `super` first, then apply customizations
- Access task outcomes through the event's `outcome.values` array; use `OCKOutcomeValue.kind` (set in Part 2) to find the right value by label
- Display past answers as text in `instructionsLabel` — this is the direct answer to participant feedback about not seeing what they answered

### Restricting Past and Future Editing
- Implement `OCKSurveyTaskViewControllerDelegate.viewController(_:shouldAllowDeletingOutcomeForEvent:) -> Bool`
- Return `false` if `event.scheduleEvent.start` is before today — prevents deleting/redoing past survey results
- Before appending task cards for a future date, check `Calendar.current.compare(date, to: Date(), toGranularity: .day) == .orderedDescending`, then set `taskCard.isUserInteractionEnabled = false` and `taskCard.alpha = 0.4`

### CareKit Charts (`OCKCartesianChartViewController`)
- `OCKDataSeriesConfiguration(taskID:legendTitle:gradientStartColor:gradientEndColor:markerSize:eventAggregator:)` — defines one data series on a chart
  - `taskID`: which CareKit task to query
  - `markerSize`: bar/point width
  - `eventAggregator`: closure `(OCKStore.Event) -> Double` — extracts the Y value from an event's outcomes using `outcome.values.first(where: { $0.kind == "pain" })?.doubleValue`
- `OCKCartesianChartViewController(plotType:selectedDate:configurations:storeManager:)` — builds the chart
  - `plotType: OCKCartesianGraphView.PlotType` — `.bar` or `.scatter`
  - Automatically subscribes to store changes — chart updates when outcomes are added/removed
- Each chart is appended to `InsightsViewController`'s list view controller

### 3D Model Educational Content with `ORK3DModelStep`
- Not tied to a CareKit schedule (no outcomes to persist) — present directly as a standalone `ORKTaskViewController`
- `ORK3DModelManager` — protocol; supply USDZ file URL and display configuration; can be subclassed or replaced with third-party model managers (e.g., BioDigital)
- `ORKUSDZ3DModelManager(url:)` — built-in implementation using SceneKit/RealityKit to load a USDZ file
- `ORK3DModelStep(identifier:modelManager:)` — step that renders the 3D model with pinch-to-zoom and rotation gestures
- Combine with an `ORKInstructionStep` (context/educational text) in an `ORKOrderedTask`
- Present via `ORKTaskViewController`; on completion, dismiss the view controller in the `taskViewController(_:didFinishWith:error:)` delegate method
- Use `OCKFeaturedContentView` (introduced WWDC20) as an entry point card in the Insights tab

## APIs & Frameworks

### CareKit — View Customization
- `OCKSurveyTaskViewSynchronizer` — base class for survey task card view synchronizers
  - `makeView(in:) -> OCKSurveyTaskView` — create the card view
  - `updateView(_:context:)` — apply data-driven customizations; call `super.updateView` first
- `OCKSurveyTaskViewController(taskID:eventQuery:storeManager:survey:viewSynchronizer:extractOutcomeValues:)` — pass custom synchronizer
- `OCKSurveyTaskViewControllerDelegate`
  - `viewController(_:shouldAllowDeletingOutcomeForEvent:) -> Bool` — restrict deletion of past events
- `OCKOutcomeValue.kind: String?` — label for lookup; set during extraction in Part 2
- `OCKOutcomeValue.doubleValue: Double?` — typed accessor for numeric outcome values

### CareKit — Charts
- `OCKDataSeriesConfiguration(taskID:legendTitle:gradientStartColor:gradientEndColor:markerSize:eventAggregator:)` — series definition
- `OCKCartesianChartViewController(plotType:selectedDate:configurations:storeManager:)` — chart view controller
  - `plotType: OCKCartesianGraphView.PlotType`: `.bar`, `.scatter`, `.line`
- `OCKCartesianGraphView` — underlying chart view

### ResearchKit — 3D Model Step
- `ORK3DModelManager` protocol — provides 3D model content to `ORK3DModelStep`
- `ORKUSDZ3DModelManager` — built-in USDZ model manager
  - `init(url: URL)` — load from local USDZ file URL
- `ORK3DModelStep(identifier:modelManager:)` — step presenting the 3D model interactively
- `OCKFeaturedContentView` (CareKit) — tap-to-expand content card; use as educational content entry point

## Code Highlights

Custom view synchronizer showing past answers:
```swift
class SurveyViewSynchronizer: OCKSurveyTaskViewSynchronizer {
    override func updateView(_ view: OCKSurveyTaskView,
                              context: OCKSynchronizationContext<OCKTaskEvents>) {
        super.updateView(view, context: context)
        if let event = context.viewModel.first?.first, event.outcome != nil {
            let pain = event.outcome?.values.first(where: { $0.kind == "pain" })?.integerValue
            let sleep = event.outcome?.values.first(where: { $0.kind == "sleep" })?.integerValue
            view.instructionsLabel.text = "Pain: \(pain ?? 0)/10 · Sleep: \(sleep ?? 0)h"
        }
    }
}
```

Prevent deleting outcomes for past events:
```swift
func viewController(_ vc: OCKSurveyTaskViewController,
                    shouldAllowDeletingOutcomeForEvent event: OCKAnyEvent) -> Bool {
    let calendar = Calendar.current
    return calendar.isDate(event.scheduleEvent.start, inSameDayAs: Date()) ||
           event.scheduleEvent.start > Date()
}
```

Bar chart for pain and sleep:
```swift
let painSeries = OCKDataSeriesConfiguration(
    taskID: TaskID.checkIn, legendTitle: "Pain",
    gradientStartColor: .systemRed, gradientEndColor: .systemOrange,
    markerSize: 10) { event in
    event.outcome?.values.first(where: { $0.kind == "pain" })?.doubleValue ?? 0
}
let sleepSeries = OCKDataSeriesConfiguration(
    taskID: TaskID.checkIn, legendTitle: "Sleep",
    gradientStartColor: .systemBlue, gradientEndColor: .systemTeal,
    markerSize: 10) { event in
    event.outcome?.values.first(where: { $0.kind == "sleep" })?.doubleValue ?? 0
}
let chartVC = OCKCartesianChartViewController(plotType: .bar, selectedDate: Date(),
                                               configurations: [painSeries, sleepSeries],
                                               storeManager: storeManager)
```

3D model step:
```swift
func kneeModelTask() -> ORKTask {
    let intro = ORKInstructionStep(identifier: "knee.intro")
    intro.title = "Your Knee"
    intro.detailText = "Learn about the anatomy of your knee and the meniscus injury."

    let modelManager = ORKUSDZ3DModelManager(url: Bundle.main.url(forResource: "knee", withExtension: "usdz")!)
    let modelStep = ORK3DModelStep(identifier: "knee.model", modelManager: modelManager)

    return ORKOrderedTask(identifier: "kneeModel", steps: [intro, modelStep])
}
```

## Takeaways
- Subclassing `OCKSurveyTaskViewSynchronizer` and using `OCKOutcomeValue.kind` labels is the recommended pattern for surfacing survey answers directly in CareKit task cards — no additional storage or lookup tables required.
- `OCKCartesianChartViewController` integrates directly with `OCKStoreManager` for reactive updates — charts automatically reflect outcome add/delete operations without manual refresh calls.
- `ORK3DModelStep` with a USDZ asset adds interactive 3D educational content to any ResearchKit flow without requiring a CareKit schedule — useful for any static educational material in a care or research app.
- Disabling user interaction on future-date task cards (combined with reduced opacity) gives participants a clear preview of their upcoming schedule while preventing premature completion.

---
_Source: WWDC21 Session 10282 page (abstract and transcript). Part 3 of 3; see also sessions 10068 and 10069._
