# Create Engaging Content for Swift Playgrounds
**WWDC22 · Session 110349** · [Watch](https://developer.apple.com/videos/play/wwdc2022/110349/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13

## Overview
This session introduces a new instructional authoring system built into Swift Playgrounds 4 that lets content creators embed guided learning experiences directly alongside SwiftPM app projects. Authors write a guide file using a directive-based extension of Markdown, which drives a Learning Center panel, welcome messages, walkthrough tasks, and interactive experiment tasks — all within the Swift Playgrounds UI.

The session walks through converting an existing SwiftUI app ("Creature Party") into a guided learning product by adding a `Guide` module alongside the `App` module in a SwiftPM project. The guide file defines the structure using nested directives such as `@GuideBook`, `@Step`, `@ContentAndMedia`, `@TaskGroup`, `@Task`, and `@Page`. Source files use special comment markers (`/*#-code-walkthrough(...)*/` and `//#-learning-task(...)`) to link guide directives to specific lines of code.

Two task types are covered in detail: **walkthrough** tasks (read-only, highlight code, paginated explanations) and **experiment** tasks (optional, can inject addable code snippets directly into the source editor).

## Key Topics

### Project Structure for Guided Content
A standard SwiftPM project must be reorganized: all source code and assets move into an `App` module, and a new `Guide` module is added at the same level as `Package.swift`. The `Guide` module contains a `.guide` file written in the directive/Markdown language.

### Guide File Directives
- `@GuideBook(title:icon:background:firstFile:)` — top-level container; sets title, branding images, and the file opened on launch
- `@WelcomeMessage(title:)` — optional animated intro shown when the project first opens
- `@Guide` — container for all steps
- `@Step(title:)` — maps to one state of the Learning Center; contains `@ContentAndMedia` and optional `@TaskGroup` directives
- `@ContentAndMedia` — Markdown body rendered in the Learning Center scroll view (supports text, images, links)
- `@TaskGroup(title:)` — optional grouping of tasks with a subtitle; displays as a section in the Learning Center
- `@Task(type:id:title:file:)` — defines a single task button in the Learning Center; `type` is `walkthrough` or `experiment`; `file` specifies which source file to open
- `@Page(id:title:isAddable:)` — a single page within a task card; `isAddable: true` enables code injection for experiment tasks

### Code Markers in Source Files
- `/*#-code-walkthrough(<pageID>)*/` — pair of block comments bracketing lines to highlight when a walkthrough page is active
- `//#-learning-task(<taskID>)` — single-line comment marking the insertion point for an addable experiment page's code snippet

### Task Types
- **Walkthrough**: paginated explanations with code highlights; sequential tasks auto-advance with a "Next Walkthrough" button
- **Experiment**: optional; pages can include code blocks (triple-backtick fenced); when `isAddable: true`, an Add button appears that inserts the code at the `#-learning-task` marker

## APIs & Frameworks

### Swift Playgrounds Guide Authoring (Directive Language)
- `@GuideBook(title:icon:background:firstFile:)` **[NEW]** — root directive
- `@WelcomeMessage(title:)` **[NEW]** — optional welcome overlay
- `@Guide` **[NEW]** — container directive
- `@Step(title:)` **[NEW]** — learning center state
- `@ContentAndMedia` **[NEW]** — Learning Center prose/images body
- `@TaskGroup(title:)` **[NEW]** — optional task section with subtitle
- `@Task(type:id:title:file:)` **[NEW]** — task entry in Learning Center; types: `walkthrough`, `experiment`
- `@Page(id:title:isAddable:)` **[NEW]** — single page within a task card; `isAddable` enables code injection

### Source File Annotation Comments
- `/*#-code-walkthrough(<pageID>)*/` **[NEW]** — bracket pair for code highlighting
- `//#-learning-task(<taskID>)` **[NEW]** — insertion marker for experiment code snippets

## Code Highlights

Minimal guide file structure:
```
@GuideBook(title: "Creature Party!", icon: icon.png, background: background.png, firstFile: CreatureDance.swift) {
    @WelcomeMessage(title: "Welcome to Creature Party!") {
        Tonight, the creatures are gonna party like it's 2022!
    }
    @Guide {
        @Step(title: "Pump up the jams") {
            @ContentAndMedia {
                Learn about custom view modifiers and animations.
            }
            @TaskGroup(title: "Walkthroughs") {
                @Task(type: walkthrough, id: "partyMode", title: "Setting up the Party", file: CreatureDance.swift) {
                    @Page(id: "1.modifier", title: "") {
                        This is a [view modifier](https://developer.apple.com/documentation/swiftui/viewmodifier).
                    }
                }
            }
            @TaskGroup(title: "Experiments") {
                @Task(type: experiment, id: "colors", title: "Dancing in the Strobe Light", file: CreatureDance.swift) {
                    @Page(id: "3.code", title: "", isAddable: true) {
                        ```
                        .colorMultiply(creatureColor)
                        ```
                    }
                }
            }
        }
    }
}
```

Source file code walkthrough marker:
```swift
/*#-code-walkthrough(1.modifier)*/
.animatedScalingEffect(startAnimation: startParty)
/*#-code-walkthrough(1.modifier)*/
```

Experiment task insertion marker:
```swift
.opacity(startParty ? 1.0 : 0.0)
//#-learning-task(colors)
```

## Takeaways
- The `Guide` module alongside the `App` module transforms a standard SwiftPM project into a guided learning product without changing the app's source files beyond adding comment markers.
- Walkthrough tasks are ideal for explaining existing code; experiment tasks enable learners to add optional code and see results immediately in the live preview.
- All directive IDs must be globally unique within the guide file; keep `@Page` content short (≤10 lines of code) as the task card has limited vertical space.
- Content authors do not need any special entitlements or additional tooling — the guide file is plain text and the project is editable in both Xcode and Swift Playgrounds.

---
_Source: WWDC22 Session 110349 page (abstract, chapter summaries, code samples, and resource links)._
