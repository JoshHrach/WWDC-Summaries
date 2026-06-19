# Introducing ClassKit
**WWDC18 · Session 215** · [Watch](https://developer.apple.com/videos/play/wwdc2018/215/)

_Platforms:_ iOS 11.4+

## Overview
ClassKit is a new iOS framework (iOS 11.4) that allows educational apps to expose their content as an assignable context tree, report student progress on assigned activities, and integrate with Apple's Schoolwork app. This session covers ClassKit's data model (contexts, activities, activity items), the teacher/student workflow via Schoolwork, Developer Mode for testing without managed Apple IDs, and a complete live-coding walkthrough of adopting ClassKit in a sample math quiz app.

Progress reporting is privacy-aware by design: teachers only receive progress data for content they explicitly assigned — not for any other activity within the app.

## Key Topics

### The ClassKit Workflow

1. **App declares contexts** — the content tree representing assignable pieces of your app (chapters, quizzes, activities).
2. **Teacher opens Schoolwork** — browses available contexts from apps that support ClassKit; creates a handout (assignment) pointing to a specific context; assigns to a class.
3. **Student taps handout** — Schoolwork deep-links into the app; app navigates to the assigned content.
4. **App reports progress** — activity data (time, score, per-question answers) is sent to the teacher's device and surfaced in the handout's progress report.

Authorization model: only contexts that a teacher explicitly includes in an assigned handout are authorized to record progress. All other contexts within the app are invisible to teachers.

### CLS Context — Building the Content Tree

- `CLSContext` represents a navigable node in the app's content hierarchy.
- One root context per app is provided automatically by `CLSDataStore.shared.mainAppContext`.
- Every piece of assignable content should be a context node.
- Key properties:
  - `identifier` — unique among siblings (not globally); context identity is the full `identifierPath` (array of identifiers from root to the node).
  - `title` — shown to teachers and students; must be clear and concise.
  - `type` — `CLSContextType` (e.g., `.chapter`, `.quiz`, `.exercise`, `.lesson`).
  - `displayOrder` — integer controlling sort order in Schoolwork's assignment picker.
  - `universalLinkURL` — set if your app supports Universal Links for deep-linking; otherwise use `NSUserActivity`.
- Context tree design principles:
  - Do not create a context for content that makes no sense to assign (e.g., a scoreboard of other users' high scores).
  - Collapse contexts when a parent and its sole child represent the same thing.
  - Design for the future: leave room for sibling contexts (e.g., future subtraction and division quizzes).

### Querying the Context Tree

- `CLSDataStore.shared.contexts(matching:)` — query by `NSPredicate` (e.g., all children of a given parent).
- `CLSDataStore.shared.contexts(matchingIdentifierPath:)` — absolute path query; returns all contexts along the path, stopping at the first missing node.
- `CLSContext.descendant(matchingIdentifierPath:)` — relative path query from an already-fetched context; returns the matched descendant or `nil`.
- `CLSDataStore.Delegate` — `createContext(forIdentifier:parentContext:parentIdentifierPath:)` — called when a path query hits a missing node; create and return the context on demand. Useful for apps with large or dynamic content catalogs.

### CLSActivity — Recording Progress

- Created by calling `context.createNewActivity()` — each call starts a new attempt (a new progress report row for the teacher).
- `currentActivity` property on `CLSContext` returns the in-progress activity, if any.
- `activity.start()` / `activity.stop()` — begin/end the duration timer.
- `activity.progress` — 0.0–1.0 scalar; set directly or via `addProgressRange(fromStart:toEnd:)`. Overlapping ranges are deduplicated automatically.
- Always call `CLSDataStore.shared.save(completion:)` after modifying the context tree or activity data.

### CLSActivityItem — Reporting Structured Data

- Abstract base class; three concrete subclasses:
  - `CLSQuantityItem` — a single scalar value (hints used, XP earned, etc.).
  - `CLSScoreItem` — a score out of a maximum (e.g., 8 out of 10 questions correct). **Best choice for primary activity item.**
  - `CLSBinaryItem` — a yes/no value (e.g., was this question answered correctly?).
- `activity.primaryActivityItem` — the highlighted metric in Schoolwork's report UI; always use the **same subclass** for a given context across all devices, otherwise Schoolwork cannot generate an aggregated class report.
- `activity.addAdditionalActivityItem(_:)` — attach supplementary items (per-question results, hints, etc.) that appear in the expanded detail view.
- Titles on activity items are visible to teachers — use descriptive, localized strings.

### Deep-Linking

- Schoolwork launches the app and passes a `NSUserActivity` with `CLSContextIdentifierPath` attached.
- In `application(_:continue:restorationHandler:)`: extract `userActivity.contextIdentifierPath`, look up or create the matching context, navigate to the corresponding UI.
- If your app already uses Universal Links: set `context.universalLinkURL` instead and Schoolwork will use that URL.

### Developer Mode

- In Settings → Developer Settings → ClassKit APIs: switch between "Teacher" and "Student" roles without managed Apple IDs or Apple School Manager.
- In Teacher mode: create handouts in Schoolwork, assign content, authorize contexts.
- In Student mode: consume handouts; progress data flows from the app back through ClassKit.
- "Reset Development Data" clears all test progress data.

### Best Practices for Education Apps

- **No StoreKit dependencies** in the assigned workflow — in-app purchases do not work in managed school environments.
- **Support purgeable storage** — shared iPads in schools have limited space; mark caches as purgeable.
- **Managed App Config** — implement `UserDefaults` key reading from the MDM-pushed configuration so school IT can pre-configure your app without user interaction.

## APIs & Frameworks

**ClassKit** **[NEW — iOS 11.4]**
- `CLSDataStore` — `shared` singleton; `mainAppContext`, `save(completion:)`, `contexts(matching:)`, `contexts(matchingIdentifierPath:)`, `delegate`
- `CLSDataStore.Delegate` — `createContext(forIdentifier:parentContext:parentIdentifierPath:)`
- `CLSContext` — `init(type:identifier:title:)`, `identifier`, `title`, `type` (`CLSContextType`), `displayOrder`, `universalLinkURL`, `isAssignable`, `becomeActive()`, `resignActive()`, `createNewActivity()`, `currentActivity`, `addChildContext(_:)`
- `CLSContextType` — `.none`, `.app`, `.chapter`, `.section`, `.level`, `.page`, `.task`, `.challenge`, `.quiz`, `.exercise`, `.lesson`, `.book`, `.game`, `.document`
- `CLSActivity` — `start()`, `stop()`, `progress`, `addProgressRange(fromStart:toEnd:)`, `primaryActivityItem`, `addAdditionalActivityItem(_:)`
- `CLSActivityItem` — abstract base; `identifier`, `title`
- `CLSScoreItem` — `init(identifier:title:score:maxScore:)`, `score`, `maxScore`
- `CLSQuantityItem` — `init(identifier:title:quantity:)`, `quantity`
- `CLSBinaryItem` — `init(identifier:title:value:valueType:)`, `value`, `valueType` (`CLSBinaryValueType`: `.trueFalse`, `.passFail`, `.yesNo`)

**Entitlement required**: ClassKit capability must be enabled in the app target's Capabilities pane in Xcode.

## Code Highlights

Declaring the context tree at app launch:
```swift
func publishContexts() {
    let additionCtx = CLSContext(type: .quiz, identifier: "addition-quiz", title: "Addition Quiz")
    additionCtx.displayOrder = 0
    let multiCtx = CLSContext(type: .quiz, identifier: "multiplication-quiz", title: "Multiplication Quiz")
    multiCtx.displayOrder = 1

    let contextMap = ["addition-quiz": additionCtx, "multiplication-quiz": multiCtx]
    let parent = CLSDataStore.shared.mainAppContext

    let predicate = NSPredicate(format: "parent = %@", parent)
    CLSDataStore.shared.contexts(matching: predicate) { existing, error in
        var toCreate = contextMap
        existing?.forEach { toCreate.removeValue(forKey: $0.identifier) }
        toCreate.values.forEach { parent.addChildContext($0) }
        CLSDataStore.shared.save(completion: nil)
    }
}
```

Handling the deep-link from Schoolwork:
```swift
func application(_ application: UIApplication,
                 continue userActivity: NSUserActivity,
                 restorationHandler: @escaping ([UIUserActivityRestoring]?) -> Void) -> Bool {
    guard let path = userActivity.contextIdentifierPath else { return false }
    CLSDataStore.shared.mainAppContext.descendant(matchingIdentifierPath: path) { context, error in
        guard let context = context else { return }
        context.becomeActive()
        DispatchQueue.main.async {
            // Navigate to the content represented by this context
            self.navigateToQuiz(for: context.identifier)
        }
    }
    return true
}
```

Recording a quiz attempt with per-question answers and a final score:
```swift
// Quiz started
let activity = quizContext.createNewActivity()
activity.start()
CLSDataStore.shared.save(completion: nil)

// Student answered a question
let binary = CLSBinaryItem(identifier: "q\(index)", title: "Question \(index)", value: isCorrect, valueType: .trueFalse)
activity.addAdditionalActivityItem(binary)

// Quiz ended
let score = CLSScoreItem(identifier: "total-score", title: "Total Score",
                         score: Double(correct), maxScore: Double(total))
activity.primaryActivityItem = score
activity.stop()
CLSDataStore.shared.save(completion: nil)
```

## Takeaways
- ClassKit's privacy model is opt-in by assignment: teachers authorize specific contexts when they create a handout, so only that data is ever reported — the rest of the app's activity remains private.
- Design the context tree to reflect what teachers would actually assign, not a 1:1 mapping of your view hierarchy. Remove non-assignable content (scoreboards, settings) and combine redundant single-child nodes.
- Always use the same `CLSActivityItem` subclass for a given context's `primaryActivityItem` across all installs; Schoolwork uses this to aggregate class-wide reports.
- Test the full teacher/student workflow using Developer Mode in Settings — no Apple School Manager or managed Apple IDs required.

---
_Source: WWDC18 Session 215 page (abstract, full transcript, and resource links)._
