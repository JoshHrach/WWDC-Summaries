# Build Agentic App Experiences with the Foundation Models Framework
**WWDC26 · Session 242** · [Watch](https://developer.apple.com/videos/play/wwdc2026/242/)

_Platforms:_ iOS 26+, iPadOS 26+, macOS 26+, watchOS 26+, visionOS 26+

## Overview
This session provides the engineering deep-dive into `DynamicProfile` — the declarative primitive introduced in WWDC26 for building agentic, multi-phase apps with the Foundation Models framework. The running example is an Origami crafting app whose three phases (brainstorming, planning, reviewing) share a single `LanguageModelSession` but need different instructions, tools, models, and even privacy boundaries at each step.

The session covers the full lifecycle of a Dynamic Profile: declaring profiles with `DynamicInstructions`, assigning different models per phase, managing and trimming conversation history with `historyTransform` and custom modifiers, using lifecycle hooks (`onResponse`, `onToolCall`) and session-scoped properties to persist state across turns, and three orchestration patterns — baton-pass, phone-a-friend, and the Skills pattern.

The final section ties evaluations and performance together: KV cache invalidation, why `historyTransform` is lower risk than direct transcript mutation, and how to use the Foundation Models Xcode Instrument alongside the Evaluations framework to measure the impact of any change before shipping.

## Key Topics

### Declaring a Dynamic Profile (3:47)
A `DynamicProfile` is a Swift struct conforming to `LanguageModelSession.DynamicProfile` with a `body` computed property that returns `Profile` blocks. The profile body is re-evaluated before every `respond` call, so it always reflects current app state. Initialize the session once with `LanguageModelSession(profile: CraftProfile(orchestrator: orchestrator))`.

### Dynamic Instructions (4:45)
`DynamicInstructions` groups related `Instructions` and tools into a composable unit. Nest one inside another to concatenate their contents — for example, an `OrigamiExpert` providing origami-specific knowledge and tools can be conditionally embedded wherever needed.

### Per-Phase Model Configuration (5:36)
Use `.model(_:)` and `.reasoningLevel(_:)` / `.temperature(_:)` modifiers on a `Profile` to assign different models per branch. PCC with `.reasoningLevel(.deep)` for brainstorming, PCC with `.reasoningLevel(.deep)` for planning, and `SystemLanguageModel` for lightweight reviewing.

### Transcript Management and History Transforms (7:21)
`historyTransform` applies a stateless, per-request closure over the session history before it is sent to the model. Use it to drop completed tool calls, limit context to recent turns, or redact sensitive entries — without permanently mutating the session transcript.

### Custom Modifiers (8:50)
Encapsulate reusable transform logic in a type conforming to `LanguageModelSession.DynamicProfileModifier`, then expose it as a fluent extension method (e.g., `.droppingCompletedToolCalls()`). The `FoundationModelsUtilities` package ships ready-made modifiers like `.rollingWindow(size:)`.

### Lifecycle Modifiers and Session Properties (9:39)
`onResponse` runs imperative code after each turn, such as trimming history or updating app state. `@SessionProperty` reads and writes session-scoped values (the built-in `\.history`, or custom entries declared via `@SessionPropertyEntry`) from inside profile `body` or tools.

### Orchestration Patterns (12:52–15:18)
Three patterns: **Baton-pass** — multiple profiles share the full transcript; a tool toggles the active profile, and the receiving profile produces the final answer. **Phone-a-friend** — a tool spawns a short-lived child `LanguageModelSession` with an isolated transcript for a sub-task; the parent always answers. **Skills** — the `Skills` / `Skill` API (from `FoundationModelsUtilities`) loads relevant procedural context into instructions on demand, a lightweight form of RAG.

### Tool Calling Mode (15:18)
`ToolCallingMode` values `.allowed`, `.disallowed`, and `.required` can be set as a profile modifier or per-respond `GenerationOptions`. When `.required`, the model loops until it calls a tool, so always provide an exit condition — either toggle the mode via `onToolCall`, or define a final-answer tool that throws `CancellationError` to break the loop.

### Transcript Error Handling (17:12)
`TranscriptErrorHandlingPolicy` controls what happens to the session transcript when a tool throws or a request is cancelled: `.revertTranscript` (default) rolls back, `.preserveTranscript` keeps the error state. The session's `transcript` property is now settable (when `isResponding == false`), enabling manual repair.

### Performance and Evaluations (18:27)
Appending to the transcript preserves KV cache; rewriting history discards it, which increases latency. Measure with the Foundation Models Instrument in Xcode, and quantify the accuracy impact of every change with the Evaluations framework before shipping.

## APIs & Frameworks

**FoundationModels**
- `LanguageModelSession.DynamicProfile` protocol **[NEW]**
- `LanguageModelSession.DynamicInstructions` protocol **[NEW]**
- `LanguageModelSession.DynamicProfileModifier` protocol **[NEW]**
- `Profile` **[NEW]** — result-builder container
- `Instructions` **[NEW]** — result-builder for instruction text
- `.model(_:)` modifier **[NEW]** — per-profile model selection
- `.reasoningLevel(_:)` modifier **[NEW]**
- `.temperature(_:)` modifier **[NEW]**
- `.historyTransform(_:)` modifier **[NEW]** — stateless history transform closure
- `.onResponse(_:)` lifecycle modifier **[NEW]**
- `.onToolCall(_:)` lifecycle modifier **[NEW]**
- `.toolCallingMode(_:)` modifier **[NEW]**
- `ToolCallingMode` **[NEW]** — `.allowed`, `.disallowed`, `.required`
- `LanguageModelSession.TranscriptErrorHandlingPolicy` **[NEW]** — `.revertTranscript`, `.preserveTranscript`
- `LanguageModelSession.transcriptErrorHandlingPolicy` **[NEW]** — settable property
- `LanguageModelSession.transcript` — now settable **[NEW]**
- `LanguageModelSession.isResponding` **[NEW]** — guard before transcript mutation
- `@SessionProperty(_:)` property wrapper **[NEW]**
- `@SessionPropertyEntry` **[NEW]** — declares custom session property keys
- `SessionPropertyValues` — keyed storage for session properties

**FoundationModelsUtilities (open source)**
- `.rollingWindow(size:)` modifier
- `Skills` / `Skill` — procedural context loading (RAG-lite)
- Custom modifier extension pattern

## Code Highlights

```swift
// Per-phase model assignment in a DynamicProfile
struct CraftProfile: LanguageModelSession.DynamicProfile {
    var orchestrator: CraftOrchestrator
    var body: some DynamicProfile {
        switch orchestrator.mode {
        case .brainstorming:
            Profile { BrainstormFacilitator(orchestrator: orchestrator) }
                .model(orchestrator.pccLanguageModel)
                .temperature(1)
        case .reviewing:
            Profile { CraftCoach() }
                .model(orchestrator.systemLanguageModel)
        }
    }
}

// Session property for cross-turn state
extension SessionPropertyValues {
    @SessionPropertyEntry var summary: String?
}

// Baton-pass pattern
Profile { BrainstormInstructions(); BatonPassTool() }
    .onToolCall { orchestrator.mode = .tutorial }
    .model(orchestrator.serverModel)
```

## Takeaways
- Replace imperative session teardown with `DynamicProfile` — the framework re-evaluates the body per prompt, automatically updating instructions and tools without losing conversation history.
- Use `historyTransform` for privacy boundaries and context trimming; prefer it over direct transcript mutation to keep KV caches intact and avoid confusing the model.
- Match orchestration pattern to task: baton-pass for sequential hand-offs sharing context, phone-a-friend for isolated sub-tasks, Skills for procedural context loading.
- Always pair profile changes with an Evaluations run to catch regressions in model accuracy before they reach users.

---
_Source: WWDC26 Session 242 page (abstract, chapter summaries, code samples, and resource links)._
