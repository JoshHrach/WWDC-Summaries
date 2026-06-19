# What's New in ClassKit
**WWDC19 · Session 247** · [Watch](https://developer.apple.com/videos/play/wwdc2019/247/)

_Platforms:_ iOS 12.2+, iPadOS 13

## Overview
ClassKit, introduced in iOS 11.3, enables educational apps to securely share student progress with teachers through the Schoolwork app. In iOS 12.2, three significant additions landed: a ClassKit Context Provider app extension that lets teachers browse an app's activity hierarchy without requiring a student to have launched the app first, a new API to programmatically mark an assigned activity as complete from within an app (removing the need for students to manually return to Schoolwork), and a new `CLSBinaryItem` flavor for reporting correct/incorrect answers at a per-question level.

## Key Topics

**ClassKit Fundamentals**
ClassKit centers on `CLSContext` — a tree of activity nodes that an app publishes. Each `CLSContext` has an `identifier` (never seen by users) and a `title` (shown in Schoolwork). The tree is rooted at the app's main context (parentless, created automatically by ClassKit) and extends as deep as needed for the app's activity hierarchy. Identifier paths (arrays of identifiers from root to node) uniquely address any context.

**CLSContext Tree Creation Best Practice**
Never create a `CLSContext` if one with the same identifier path already exists. The recommended pattern is to implement `CLSDataStoreDelegate.createContext(forIdentifier:parentContext:parentIdentifierPath:)` — ClassKit calls this delegate only when a context with that identifier path does not yet exist, consolidating all creation logic in one place. Calling `CLSDataStore.shared.contexts(matchingIdentifierPath:)` or `CLSContext.descendant(matchingIdentifierPath:)` asynchronously checks whether a context exists before creating it.

**ClassKit Context Provider Extension (NEW in iOS 12.2)**
A new app extension type — ClassKit Context Provider — that lets Schoolwork request context creation on demand, even if the app has never been launched. The extension implements a single required method: `updateDescendants(of:completion:)`. ClassKit calls this when a teacher drills into an app's activity list in Schoolwork, passing the parent `CLSContext` whose children should be populated. The extension must add or update those children in the context tree and call the completion block quickly (the teacher is waiting in real time). The extension cannot display UI; it must not require user interaction.

**Schoolwork Teacher/Student Workflow**
Developers can test both roles using the ClassKit API toggle in Settings → Developer (requires a developer iOS install). Switching to Teacher role in Schoolwork lets a developer create handouts with app activities; switching to Student lets them complete those activities and generate progress data; switching back to Teacher shows the recorded progress.

**Complete Activity API (NEW)**
`CLSDataStore.shared.completeAllAssignedActivities(matchingContextPath:)` **[NEW]** marks a teacher-assigned activity as complete directly from within the student's app session. The context path is an identifier path array. Once called, the next time the student opens Schoolwork, the activity is shown as complete — and the teacher sees it as complete for that student. Eliminates the extra step of requiring students to manually tap Complete in Schoolwork.

**CLSBinaryItem correct/incorrect (NEW)**
`CLSBinaryItem` gained a new `CLSBinaryItemValueType.correct` / `.incorrect` enumeration value. This joins the existing `.trueFalse` and `.passFail` flavors. Intended for per-question quiz tracking: a primary `CLSQuantityItem` can record the overall quiz score percentage, while one `CLSBinaryItem` per question (using `.correct`/`.incorrect`) gives teachers per-question insight into which items a student missed.

## APIs & Frameworks

**ClassKit**
- `CLSContext` — activity tree node; properties: `identifier`, `title`, `parent`, `isActive`
- `CLSDataStore.shared.mainAppContext` — root context for the app (system-created)
- `CLSDataStore.shared.contexts(matchingIdentifierPath:completion:)` — async lookup by identifier path
- `CLSContext.descendant(matchingIdentifierPath:completion:)` — async descendant lookup
- `CLSDataStoreDelegate.createContext(forIdentifier:parentContext:parentIdentifierPath:)` — called only when context does not yet exist; preferred creation point
- `CLSDataStore.shared.save(completion:)` — persist context tree changes
- `CLSDataStore.shared.completeAllAssignedActivities(matchingContextPath:)` **[NEW]** — mark a teacher-assigned activity complete from within the app
- `CLSActivity` — progress-recording object attached to an active `CLSContext`
- `CLSActivityItem` — base class for progress data attached to an activity
- `CLSBinaryItem` — binary progress item; `CLSBinaryItemValueType`: `.trueFalse`, `.passFail`, `.correct` / `.incorrect` **[NEW]**
- `CLSQuantityItem` — numeric progress (e.g., score percentage)
- `CLSScoreItem` — score + max score pair

**ClassKit Context Provider Extension (NEW)**
- Template available in Xcode: File → New → Target → iOS → ClassKit Context Provider
- Subclass `CLSContextProvider`
- Override `updateDescendants(of context: CLSContext, completion: @escaping (Error?) -> Void)`
- Called by Schoolwork when a teacher navigates into an app's activity hierarchy
- Must save context changes and call `completion(nil)` on success, `completion(error)` on failure
- Must complete quickly — called in real-time as the teacher browses

## Code Highlights

Implementing the Context Provider Extension:

```swift
import ClassKit

class MyContextProvider: CLSContextProvider {
    override func updateDescendants(of context: CLSContext,
                                   completion: @escaping (Error?) -> Void) {
        // If root (main app context has no parent)
        if context.parent == nil {
            // Ensure top-level lesson contexts exist
            let lessonIDs = ["1_intro", "2_swift", "3_vars", "4_structs",
                             "5_loops", "6_functions", "7_cond"]
            for id in lessonIDs {
                if context.childContext(withIdentifier: id) == nil {
                    let lesson = CLSContext(type: .task, identifier: id, title: titleFor(id))
                    context.addChildContext(lesson)
                }
            }
        }
        CLSDataStore.shared.save { error in
            completion(error)
        }
    }
}
```

Using the delegate pattern to centralize context creation:

```swift
class AppContextManager: NSObject, CLSDataStoreDelegate {
    func createContext(forIdentifier identifier: String,
                       parentContext: CLSContext,
                       parentIdentifierPath: [String]) -> CLSContext? {
        // Called only when a context with this path does not yet exist
        return CLSContext(type: .task, identifier: identifier, title: titleFor(identifier))
    }
}
// At app launch:
CLSDataStore.shared.delegate = AppContextManager.shared
```

Marking an activity complete:

```swift
// Student finished the quiz at path ["2_vars", "2_quiz"]
CLSDataStore.shared.completeAllAssignedActivities(matchingContextPath: ["2_vars", "2_quiz"])
```

Recording per-question correct/incorrect results:

```swift
let quiz = CLSActivity()
for (index, question) in quizQuestions.enumerated() {
    let item = CLSBinaryItem(identifier: "q\(index)", title: question.text, type: .correct)
    item.value = studentAnswers[index] == question.correctAnswer
    quiz.addAdditionalActivityItem(item)
}
```

## Takeaways
- The **ClassKit Context Provider Extension** is the highest-value addition: teachers can now browse and assign your app's activities without a student ever having launched the app, dramatically lowering the barrier to classroom adoption.
- Always use `CLSDataStoreDelegate.createContext(forIdentifier:...)` as the single creation point to avoid duplicate-context errors across app launches and extension calls.
- Call `CLSDataStore.shared.completeAllAssignedActivities(matchingContextPath:)` when a student finishes an assigned activity — it removes friction from the student workflow and gives teachers a cleaner view.
- The new `.correct`/`.incorrect` `CLSBinaryItem` type enables granular per-question quiz reporting, which is more actionable for teachers than a single percentage score alone.

---
_Source: WWDC19 Session 247 page (abstract, transcript, and resource links)._
