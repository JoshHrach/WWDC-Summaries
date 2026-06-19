# Build with the New Apple Foundation Model on Private Cloud Compute
**WWDC26 · Session 319** · [Watch](https://developer.apple.com/videos/play/wwdc2026/319/)

_Platforms:_ iOS 26+, iPadOS 26+, macOS 26+, watchOS 26+, visionOS 26+

## Overview
Private Cloud Compute (PCC) makes a powerful, frontier-class server model available to third-party apps for the first time, without compromising user privacy. Data is never stored on Apple servers, is used only for the duration of the request, and is independently verifiable — the system is integrated with iCloud so there are no API keys, no per-developer billing, and no account setup for the end user.

This session explains the practical details: how PCC differs from the on-device model (context size, reasoning, connectivity), how to integrate it with a one-line change to existing Foundation Models code, how to handle the per-user daily quota gracefully in SwiftUI, and how to choose between on-device and PCC for a given feature. The session also covers the new `contextSize` and `tokenCount` APIs, reasoning levels, and the Xcode debug tool for simulating availability states.

PCC is available to apps with fewer than 2 million downloads, requires applying on the developer website, and supports watchOS 27 this year. The unified Foundation Models API means all existing patterns — `@Generable` structured output, tool calling, streaming — work identically on both models.

## Key Topics

### What is Private Cloud Compute (1:23)
PCC delivers a high-capacity server model under Apple's privacy guarantees: requests are not logged, data is deleted after use, and the mechanism is open to independent cryptographic audit. Access is gated through the OS and iCloud — no authentication or API tokens for the developer, no token cost — with a daily per-user quota that is higher for iCloud+ subscribers.

### Integrating PCC with Foundation Models (2:43)
Switching from the on-device model to PCC requires changing exactly one line:

```swift
// On-device
let session = LanguageModelSession()
// PCC
let session = LanguageModelSession(model: PrivateCloudComputeLanguageModel())
```

`@Generable` types, `Tool` conformances, and `.respond(to:generating:)` all work without modification.

### Checking Availability (3:51)
`PrivateCloudComputeLanguageModel().isAvailable` is a synchronous `Bool` that returns `false` on non-Apple-Intelligence devices or when connectivity is unavailable. Check it before showing related UI; use Xcode's "Simulate Apple Foundation Models Availability" debug toggle to test both states.

### Deciding Between On-Device and PCC (4:00)
On-device: works offline, no request limits, 4 K–8 K context (depending on device). PCC: requires a connection, daily per-user limit, 32 K context, supports reasoning levels. Choose based on measured evaluation data, not assumptions.

### Reasoning Levels and Context Size (4:32)
Set reasoning on a per-request basis via `ContextOptions(reasoningLevel:)`. Three levels: `.light`, `.moderate`, `.deep`. Reasoning tokens are generated as part of the model's thinking process and count against the context window. The `contextSize` property is now readable on both `SystemLanguageModel` and `PrivateCloudComputeLanguageModel`:

```swift
SystemLanguageModel().contextSize           // 4096 (26.0), 8192 (27.0)
PrivateCloudComputeLanguageModel().contextSize // 32768
```

### Evaluating and Combining Models (6:15)
Use the Evaluations framework to compare on-device vs. PCC on your specific task — the updated on-device model may be sufficient. `DynamicProfile` makes it trivial to assign PCC for compute-intensive phases and `SystemLanguageModel` for fast, lightweight ones within a single session.

### Handling Usage Limits (7:10)
The `PrivateCloudComputeLanguageModel().quotaUsage` property exposes the daily quota state. Show persistent, actionable UI rather than alerts: disable the feature with a brief message when `isLimitReached`, surface an "approaching limit" warning (`isApproachingLimit`) in orange, and provide an upgrade path via `limitIncreaseSuggestion?.show()`.

## APIs & Frameworks

**FoundationModels**
- `PrivateCloudComputeLanguageModel` **[NEW]** — server model type
- `PrivateCloudComputeLanguageModel().isAvailable: Bool` **[NEW]**
- `PrivateCloudComputeLanguageModel().contextSize: Int` **[NEW]** — 32768
- `PrivateCloudComputeLanguageModel().quotaUsage` **[NEW]**
  - `.status` — `.belowLimit(QuotaInfo)` or limit reached
  - `QuotaInfo.isApproachingLimit: Bool`
  - `.isLimitReached: Bool`
  - `.limitIncreaseSuggestion` — optional, call `.show()` to present upgrade UI
- `SystemLanguageModel().contextSize: Int` **[NEW]** — 4096 / 8192
- `ContextOptions(reasoningLevel:)` — `.light`, `.moderate`, `.deep`
- `LanguageModelSession(model:)` — accepts any `LanguageModel`
- `session.respond(to:generating:)` — unchanged API
- `@Generable` — works identically on PCC
- `Tool` protocol — works identically on PCC

**Xcode**
- "Simulate Apple Foundation Models Availability" — debug toggle for availability/quota states

## Code Highlights

```swift
// One-line switch to PCC
let session = LanguageModelSession(model: PrivateCloudComputeLanguageModel())

// Reasoning level
let response = try await session.respond(
    to: prompt,
    contextOptions: ContextOptions(reasoningLevel: .light)
)

// Quota handling in SwiftUI
var body: some View {
    if model.quotaUsage.isLimitReached {
        Text("Usage limit exceeded.").foregroundStyle(.red)
    }
    if let suggestion = model.quotaUsage.limitIncreaseSuggestion {
        Button("Show options") { suggestion.show() }
    }
}
```

## Takeaways
- Use `PrivateCloudComputeLanguageModel` for summarisation, reasoning over large documents, and any task that benefits from a 32 K context — no new auth code required.
- Always check `isAvailable` before making requests and show graceful fallback UI on unsupported devices.
- Handle the quota lifecycle explicitly: detect `isApproachingLimit` early and surface actionable upgrade paths rather than silent failures.
- Let the Evaluations framework, not intuition, decide whether PCC or the on-device model is better for a given feature.

---
_Source: WWDC26 Session 319 page (abstract, chapter summaries, code samples, and resource links)._
