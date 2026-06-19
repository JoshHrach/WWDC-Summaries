# What's New in the Foundation Models Framework
**WWDC26 · Session 241** · [Watch](https://developer.apple.com/videos/play/wwdc2026/241/)

_Platforms:_ iOS 26+, iPadOS 26+, macOS 26+, watchOS 26+, visionOS 26+

## Overview
Session 241 is the annual "state of the union" for the Foundation Models framework, covering everything new in the 2026 cycle. The talk spans model improvements (on-device and server), two brand-new cross-platform tooling surfaces (the `fm` CLI and Python SDK), and the freshly open-sourced utilities package.

The on-device model receives a major upgrade — better reasoning, improved tool calling, and, for the first time, full vision support. Apple's Private Cloud Compute model is now accessible to developers, offering a 32 K-token context window with deep reasoning, and it now extends to watchOS 27. A new `LanguageModel` protocol ties together on-device, PCC, CoreAI, and MLX models under one unified session API, with partner packages from Anthropic and Google joining the ecosystem.

Two major productivity and quality themes close the session: Dynamic Profiles, a declarative primitive that lets a single `LanguageModelSession` swap instructions, tools, and even the backing model at runtime; and the new Evaluations framework, which brings statistical rigour to prompt engineering so developers can ship intelligence features with confidence.

## Key Topics

### New On-Device Model (2:34)
Rebuilt foundation model with stronger reasoning and tool calling. New APIs (iOS 26.4+): `SystemLanguageModel().contextSize` returns `8192` on newer devices, and `model.tokenCount(for:)` counts tokens before sending a request. Guardrails are also refined to reduce false positives.

### Vision – Image Understanding (3:21)
The on-device model gains multimodal input. Attach images to any prompt via `Attachment(UIImage(...))`, `NSImage`, `CGImage`, `CIImage`, `CVPixelBuffer`, or file `URL`. Any image size is accepted; larger images consume more tokens.

### Private Cloud Compute (4:20)
`PrivateCloudComputeLanguageModel()` surfaces Apple's server model — 32 K context, reasoning levels, no API keys or billing. Now available on watchOS 27. Usage statistics (input/output/cached/reasoning token counts) are exposed on the response object.

### Model Abstraction Layer (6:46)
A new `LanguageModel` protocol makes `LanguageModelSession` model-agnostic. New conformers: `CoreAILanguageModel` (Neural Engine) and `MLXLanguageModel` (Neural Engine + GPU). Existing `SystemLanguageModel` and `PrivateCloudComputeLanguageModel` already conform.

### Partner Model Integrations (7:32)
Anthropic and Google publish first-party Swift packages. Authentication uses OAuth + Keychain — no plain API key strings in source. Per-response usage reports include cached and reasoning token counts.

### System Tools: Vision and Spotlight (9:40)
Three new built-in tools: `BarcodeReaderTool` and `OCRTool` (backed by the Vision framework), and a Spotlight-powered search tool enabling fully on-device RAG with no network round-trip.

### Dynamic Profiles for Agentic Apps (10:57)
`DynamicProfile` is a declarative protocol where a struct's `body` returns `Profile` blocks containing `Instructions`, tools, and optional `.model()` / `.reasoningLevel()` modifiers. The profile is re-evaluated on each prompt, transparently updating the session's active persona.

### Composing Models and Configurations (13:46)
Profile branches can target different models — `SystemLanguageModel` for fast analysis and PCC with `.reasoningLevel(.deep)` for brainstorming — while preserving the full conversation `Transcript`. `Transcript.dropFirstInstructions()` lets you carry history across session re-creations.

### Evaluations Framework (15:30)
A new Swift framework for measuring intelligence-feature quality. Structured around `Evaluation`, `Metric`, `Evaluator`, and `MetricsAggregator` types; integrates with Swift Testing via the `.evaluates()` trait.

### The `fm` CLI (16:02)
Ships with macOS 27. `fm chat` opens an interactive session; `fm respond` emits one-shot output to stdout, making it trivially pipeable in shell scripts. Supports `--model pcc` to target Private Cloud Compute.

### Foundation Models Python SDK (17:13)
`apple_fm_sdk` (`pip install apple_fm_sdk`) mirrors the Swift API: `SystemLanguageModel`, `LanguageModelSession`, `session.respond()`, availability checks, and `@fm.generable` structured output.

### Open Source and Utilities Package (17:55)
The core framework is open-sourced and runs on Linux (servers). The companion utilities package provides transcript management, a `Skills` API, rolling-window history helpers, and a chat-completions compatibility layer.

## APIs & Frameworks

**FoundationModels**
- `SystemLanguageModel` — on-device model; `.contextSize` **[NEW]** property
- `PrivateCloudComputeLanguageModel` **[NEW]** — Private Cloud Compute server model
- `CoreAILanguageModel` **[NEW]** — local CoreAI model (Neural Engine)
- `MLXLanguageModel` **[NEW]** — local MLX model (Neural Engine + GPU)
- `LanguageModel` protocol **[NEW]** — abstraction over all model types
- `LanguageModelSession` — session type; `init(model:)`, `init(profile:)` **[NEW]**
- `LanguageModelSession.DynamicProfile` protocol **[NEW]**
- `DynamicProfile` / `Profile` / `Instructions` result-builder types **[NEW]**
- `ContextOptions(reasoningLevel:)` — `.light`, `.moderate`, `.deep`
- `Transcript` — `dropFirstInstructions()` **[NEW]**
- `Attachment` — `init(UIImage:)`, `init(NSImage:)`, `init(CGImage:)`, `init(CIImage:)`, `init(CVPixelBuffer:)`, `init(URL:)` **[NEW]**
- `response.usage.input.totalTokenCount` / `.cachedTokenCount` **[NEW]**
- `response.usage.output.totalTokenCount` / `.reasoningTokenCount` **[NEW]**
- `model.tokenCount(for:)` async **[NEW]**
- `BarcodeReaderTool` **[NEW]** — Vision-backed built-in tool
- `OCRTool` **[NEW]** — Vision-backed built-in tool
- Spotlight search tool **[NEW]** — on-device RAG

**Evaluations (new framework)**
- `Evaluation` protocol
- `Metric`, `Evaluator`, `MetricsAggregator`
- `.evaluates()` Swift Testing trait

**Foundation Models Utilities (open source)**
- `Skills` / `Skill` API
- Rolling-window history modifier (`.rollingWindow(size:)`)
- Chat-completions interface

**Python SDK (`apple_fm_sdk`)**
- `fm.SystemLanguageModel`, `fm.LanguageModelSession`
- `session.respond(prompt:)` async
- `model.is_available()`
- `@fm.generable` decorator

## Code Highlights

```swift
// Token counting (new)
let model = SystemLanguageModel()
print(model.contextSize) // 8192
let count = try await model.tokenCount(for: "...")

// Image attachment
let response = try await session.respond {
    "What animal is this?"
    Attachment(UIImage(...))
}

// Dynamic Profile with model switching
struct CraftProfile: LanguageModelSession.DynamicProfile {
    let states: CraftProjectStates
    var body: some DynamicProfile {
        switch states.mode {
        case .brainstorm:
            Profile { ... }
                .model(states.privateCloudCompute)
                .reasoningLevel(.deep)
        case .craftAnalysis:
            Profile { ... } // uses default on-device model
        }
    }
}
```

## Takeaways
- Adopt `PrivateCloudComputeLanguageModel` for tasks needing a larger context window or deep reasoning — no auth setup required, and the same Swift API applies.
- Use the new `LanguageModel` protocol to write model-agnostic code that can swap between on-device, PCC, CoreAI, MLX, and partner models with one-line changes.
- Replace ad-hoc session teardown/recreation with `DynamicProfile` to cleanly express multi-phase agentic flows while preserving conversation history.
- Build an `Evaluation` suite before shipping any intelligence feature so you can quantify the impact of every prompt or model change.

---
_Source: WWDC26 Session 241 page (abstract, chapter summaries, code samples, and resource links)._
