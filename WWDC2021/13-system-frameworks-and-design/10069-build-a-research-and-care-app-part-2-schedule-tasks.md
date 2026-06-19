# Build a Research and Care App, Part 2: Schedule Tasks
**WWDC21 · Session 10069** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10069/)

_Platforms:_ iOS 15, iPadOS 15

## Overview
Part 2 of the three-session "Recover" code-along adds two real study tasks to the care feed: a daily check-in survey using ResearchKit forms and a range-of-motion measurement using a ResearchKit active task. The session focuses on three skills: building multi-question forms with `ORKFormStep`, parsing nested `ORKTaskResult` trees into `OCKOutcomeValue` arrays, and composing compound `OCKSchedule` objects from multiple `OCKScheduleElement` pieces to express evolving treatment regimens.

The result-to-outcome bridge pattern is central here: after ResearchKit completes a survey, the app's `extractOutcomeValues` closure navigates the `ORKTaskResult` tree, extracts typed answers, converts them to `OCKOutcomeValue` objects (with optional `kind` labels for later lookup), and returns them so CareKit can persist them and update the UI.

## Key Topics

### Multi-Question Forms with `ORKFormStep`
- `ORKFormStep` presents multiple questions on a single screen — better UX than one question per screen for short daily check-ins
- Each question is an `ORKFormItem` with an `ORKAnswerFormat` specifying input type
- `ORKScaleAnswerFormat(maximumValue:minimumValue:defaultValue:step:vertical:maximumValueDescription:minimumValueDescription:)` — creates a `UISlider`-based answer; ideal for pain/sleep ratings
- Set `isOptional = false` on `ORKFormItem` to prevent submission without an answer
- Set `isOptional = false` on `ORKFormStep` to prevent the entire step from being skipped

### Parsing `ORKTaskResult` Trees
- `ORKTaskResult` has a nested structure: root → form result → individual question results
- Navigation: `taskResult.result(forIdentifier: formID)` → cast to `ORKStepResult` → `.results?.compactMap { $0 as? ORKScaleQuestionResult }`
- `ORKScaleQuestionResult.scaleAnswer: NSNumber?` — the actual slider value
- Extract `ORKRangeOfMotionResult` similarly: `result.range` gives the range of motion in degrees
- `OCKOutcomeValue(value)` — wrap the extracted value; set `.kind = "pain"` / `.kind = "sleep"` for later lookup by key path

### Advanced CareKit Schedules with `OCKScheduleElement`
- `OCKScheduleElement(start:end:interval:)` — defines one repeating period within a larger schedule
- Compose multiple elements into one `OCKSchedule` using `OCKSchedule(composing:[element1, element2])`
- Example: daily for week 1, weekly for weeks 2–4, then nothing after month 1 — each phase is its own `OCKScheduleElement`; `end` date terminates each phase
- `OCKTaskQuery.excludesTasksWithNoEvents = true` — filter out tasks that have no scheduled events on a given date (e.g., a weekly task on a non-scheduled day)

### ResearchKit Active Task: Range of Motion
- `ORKOrderedTask.kneeRangeOfMotionTask(withIdentifier:limbOption:optionFlags:)` — predefined active task using Core Motion + Siri audio guidance to measure knee flexion angle
  - `limbOption: ORKPredefinedTaskLimbOption` — `.left` or `.right`
  - `optionFlags: ORKPredefinedTaskOption` — pass `.excludeConclusion` to omit the default completion step and supply your own
- `ORKRangeOfMotionResult.range: Double` — measured range of motion in degrees (the primary output)
- ResearchKit provides real-time Siri voice guidance during the measurement — no custom audio code needed

### Generic Task Card Population
- Fetch all tasks for a date with `OCKTaskQuery(for: Date())`, set `excludesTasksWithNoEvents = true`
- In `prepareListViewController(for:)`, iterate tasks, dispatch to a `viewController(for:)` factory method by `task.id`
- This pattern scales cleanly as tasks are added without changing the list population loop

## APIs & Frameworks

### ResearchKit
- `ORKFormStep(identifier:title:text:)` — multi-question form screen
  - `isOptional: Bool` — prevent skipping the whole step
- `ORKFormItem(identifier:text:answerFormat:)` — one question within a form
  - `isOptional: Bool` — require an answer before submission
- `ORKScaleAnswerFormat(maximumValue:minimumValue:defaultValue:step:vertical:maximumValueDescription:minimumValueDescription:)` — slider answer
- `ORKStepResult` — result for one step; `.results: [ORKResult]?`
- `ORKScaleQuestionResult` — scale answer; `.scaleAnswer: NSNumber?`
- `ORKOrderedTask.kneeRangeOfMotionTask(withIdentifier:limbOption:optionFlags:)` — predefined knee ROM active task
- `ORKRangeOfMotionResult` — ROM task result; `.range: Double` (degrees)
- `ORKPredefinedTaskOption.excludeConclusion` — omit built-in completion step to supply custom one

### CareKit
- `OCKScheduleElement(start:end:interval:)` — one repeating period
- `OCKSchedule(composing:[OCKScheduleElement])` — compose multiple elements into a compound schedule
- `OCKTaskQuery(for: Date())` — query tasks for a specific date
  - `excludesTasksWithNoEvents: Bool` — filter out tasks with no events on that date
- `OCKOutcomeValue(value)` — typed value wrapper for persisting survey answers
  - `kind: String?` — optional label for later retrieval by key path
- `OCKSurveyTaskViewController` — unchanged from Part 1; `extractOutcomeValues` closure receives `ORKTaskResult`

## Code Highlights

Multi-question daily check-in form:
```swift
func checkInSurvey() -> ORKTask {
    let formStep = ORKFormStep(identifier: "checkin.form", title: "Daily Check-in", text: nil)
    formStep.isOptional = false

    let painItem = ORKFormItem(identifier: "checkin.form.pain", text: "Pain level (1–10)?",
                               answerFormat: ORKScaleAnswerFormat(maximumValue: 10, minimumValue: 1,
                                   defaultValue: 5, step: 1, vertical: false,
                                   maximumValueDescription: "Lots", minimumValueDescription: "None"))
    painItem.isOptional = false

    let sleepItem = ORKFormItem(identifier: "checkin.form.sleep", text: "Hours of sleep?",
                                answerFormat: ORKScaleAnswerFormat(maximumValue: 12, minimumValue: 0,
                                    defaultValue: 8, step: 1, vertical: false,
                                    maximumValueDescription: "12 hours", minimumValueDescription: "0 hours"))
    sleepItem.isOptional = false

    formStep.formItems = [painItem, sleepItem]
    return ORKOrderedTask(identifier: TaskID.checkIn, steps: [formStep])
}
```

Parsing nested ORKTaskResult into OCKOutcomeValues:
```swift
func extractOutcomeValues(from result: ORKTaskResult) throws -> [OCKOutcomeValue] {
    guard let stepResult = result.result(forIdentifier: "checkin.form") as? ORKStepResult,
          let scaleResults = stepResult.results?.compactMap({ $0 as? ORKScaleQuestionResult }),
          let painAnswer = scaleResults.first(where: { $0.identifier == "checkin.form.pain" })?.scaleAnswer,
          let sleepAnswer = scaleResults.first(where: { $0.identifier == "checkin.form.sleep" })?.scaleAnswer
    else { throw AppError.couldntExtractAnswers }

    var painValue = OCKOutcomeValue(Double(truncating: painAnswer))
    painValue.kind = "pain"

    var sleepValue = OCKOutcomeValue(Double(truncating: sleepAnswer))
    sleepValue.kind = "sleep"

    return [painValue, sleepValue]
}
```

Compound schedule (daily for week 1, weekly for weeks 2–4):
```swift
let thisMorning = Calendar.current.startOfDay(for: Date())
let nextWeek = Calendar.current.date(byAdding: .day, value: 7, to: thisMorning)!
let nextMonth = Calendar.current.date(byAdding: .month, value: 1, to: thisMorning)!

let dailyElement = OCKScheduleElement(start: thisMorning, end: nextWeek,
                                       interval: DateComponents(day: 1))
let weeklyElement = OCKScheduleElement(start: nextWeek, end: nextMonth,
                                        interval: DateComponents(weekOfYear: 1))
let schedule = OCKSchedule(composing: [dailyElement, weeklyElement])
```

## Takeaways
- `ORKFormStep` with multiple `ORKFormItem` questions and `ORKScaleAnswerFormat` sliders creates engaging daily check-ins far superior to paper surveys.
- The `extractOutcomeValues` closure is the key integration point between ResearchKit and CareKit — navigate the `ORKTaskResult` tree, cast to typed result subclasses, and wrap values in `OCKOutcomeValue` with meaningful `kind` labels.
- `OCKSchedule(composing:)` enables sophisticated evolving regimens (intensive early, tapering off) without server-side logic — encode the entire treatment protocol at task creation time.
- ResearchKit's predefined `kneeRangeOfMotionTask` uses Core Motion and Siri audio guidance to enable at-home clinical measurements that would otherwise require an in-person visit.

---
_Source: WWDC21 Session 10069 page (abstract and transcript). Part 2 of 3; see also sessions 10068 and 10282._
