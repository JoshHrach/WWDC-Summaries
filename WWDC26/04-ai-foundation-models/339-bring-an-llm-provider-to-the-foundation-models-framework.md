# Bring an LLM Provider to the Foundation Models Framework
**WWDC26 · Session 339** · [Watch](https://developer.apple.com/videos/play/wwdc2026/339/)

_Platforms:_ iOS 27+, iPadOS 27+, macOS 27+, visionOS 27+, watchOS 27+, Linux

## Overview
This session is the deep-dive guide for anyone who wants to publish an LLM as a Swift package that plugs into the Foundation Models framework. By implementing the `LanguageModel` and `LanguageModelExecutor` protocols, any model — a cloud API, an on-device CoreAI bundle, or an MLX HuggingFace checkpoint — can back a `LanguageModelSession`, giving the full Foundation Models feature set (structured generation, tool calling, streaming, evaluations) to downstream app developers without any code changes on their side.

The session covers the two new protocols in detail, walks through packaging conventions (Swift Package Manager, platform targets, deliberate dependency management), and goes deep on production concerns: KV-cache preservation, authentication with OAuth and App Attest, custom metadata, new input/output modalities via `Transcript.CustomSegment`, and server-side tools.

Privacy is a first-class theme: the session closes with a reminder that developers deploying a cloud model must be transparent about data handling, because on-device and server models have very different privacy characteristics.

## Key Topics

### Packaging (3:37)
Structure your model as a Swift package with two targets: a private `MyModelRuntime` target housing weights and inference, and a public `MyModel` target that exposes the `LanguageModel` conformance. Declare all four Apple platforms plus Linux in `Package.swift` so developers can integrate without worrying about platform guards. Publish via a git tag; downstream developers add the URL directly in Xcode.

### The Two Core Protocols (4:48)
`LanguageModel` declares what the model can do (`capabilities`: `.toolCalling`, `.guidedGeneration`, `.reasoning`) and provides an `Executor.Configuration` value used as a cache key. `LanguageModelExecutor` is the engine: `init(configuration:)` is called once per unique configuration, `prewarm(model:transcript:)` loads weights proactively, and `respond(to:model:streamingInto:)` drives generation. The framework caches executor instances by configuration and reuses KV cache state across calls to the same executor, so keeping the same configuration is important for performance.

### Mapping the Transcript (9:00)
The `Transcript` passed in `respond(to:)` is a sequence of typed entries — `.instructions`, `.prompt`, `.toolCalls`, `.toolOutput`, `.response` — that the executor must translate to whatever format its inference engine expects. This mapping is the core of the implementation.

### Reading Request Options (10:42)
The `LanguageModelExecutorGenerationRequest` exposes `contextOptions.reasoningLevel`, `generationOptions.temperature`, and `generationOptions.maximumResponseTokens`. Honour these faithfully, approximate where the backend differs, or throw `LanguageModelError.unsupportedCapability` when the gap is irreconcilable.

### Streaming Responses (11:47)
The `LanguageModelExecutorGenerationChannel` receives response events in a defined order: (1) metadata update, (2) usage update (prompt tokens before generation starts), (3) text deltas as they arrive. Usage reporting separates cached and reasoning token counts.

### Error Handling (13:33)
Throw `LanguageModelError` cases for cross-provider semantics (`contextSizeExceeded`, `rateLimited`, `refusal`, `guardrailViolation`, `unsupportedCapability`, `timeout`, and more). Define your own `Error` type for provider-specific conditions (subscription limits, account suspension, model not provisioned).

### Authentication (14:50)
Design initializers that guide developers toward secure storage; never accept plain API key strings. Persist credentials in the Keychain. Use App Attest to verify device integrity before issuing tokens to your cloud backend.

### Customization (15:51)
Go beyond the protocol baseline by attaching custom performance metadata (`tokensPerSecond`, `timeToFirstToken`) via `channel.send(.metadataUpdate(...))`. Define `Transcript.CustomSegment` conformances for new modalities (audio, video), both as prompt inputs and streamed outputs. Implement server-side tools (web search, code execution, image generation) at three exposure levels: grounded silently, metadata-enriched, or fully surfaced through custom segments.

## APIs & Frameworks

**FoundationModels**
- `LanguageModel` protocol **[NEW]** — `capabilities: LanguageModelCapabilities`, `executorConfiguration: Executor.Configuration`
- `LanguageModelCapabilities` — `.toolCalling`, `.guidedGeneration`, `.reasoning` **[NEW]**
- `LanguageModelExecutor` protocol **[NEW]** — `init(configuration:)`, `prewarm(model:transcript:)`, `respond(to:model:streamingInto:)`
- `LanguageModelExecutorGenerationRequest` **[NEW]** — `contextOptions`, `generationOptions`, `schema`, `id`
- `LanguageModelExecutorGenerationChannel` **[NEW]** — `send(_:)` async
- Channel actions: `.response(action: .updateMetadata(_:))`, `.response(action: .updateUsage(input:output:))`, `.response(action: .appendText(_:tokenCount:))`, `.response(action: .updateCustomSegment(_:))`, `.metadataUpdate(_:)` **[NEW]**
- `Transcript.CustomSegment` protocol **[NEW]** — `id: String`, `content`
- `LanguageModelError` enum — `contextSizeExceeded`, `rateLimited`, `refusal`, `guardrailViolation`, `unsupportedCapability`, `unsupportedTranscriptContent`, `unsupportedGenerationGuide`, `unsupportedLanguageOrLocale`, `timeout`
- `ContextOptions.reasoningLevel` — `.light`, `.moderate`, `.deep`
- `GenerationOptions.temperature`, `.maximumResponseTokens`, `.sampling`
- `Transcript` entries: `.instructions`, `.prompt`, `.toolCalls`, `.toolOutput`, `.response`

**Open-source model packages (referenced)**
- `CoreAILanguageModel` — `github.com/apple/coreai-models`
- `MLXLanguageModel` — `github.com/ml-explore/mlx-swift-lm`

**Security**
- Keychain — for credential storage
- App Attest — for device attestation in cloud model services

## Code Highlights

```swift
// Declare capabilities and wire up executor
public struct MyLanguageModel: LanguageModel {
    typealias Executor = MyLanguageModelExecutor
    public var capabilities: LanguageModelCapabilities {
        LanguageModelCapabilities(capabilities: [.toolCalling, .guidedGeneration, .reasoning])
    }
    public var executorConfiguration: Executor.Configuration { .init(/* ... */) }
}

// Stream metadata → usage → text
func respond(to request: ..., streamingInto channel: ...) async throws {
    await channel.send(.response(action: .updateMetadata([...])))
    await channel.send(.response(action: .updateUsage(
        input: .init(totalTokenCount: promptTokens, cachedTokenCount: cached),
        output: .init(totalTokenCount: 0, reasoningTokenCount: 0)
    )))
    for try await token in tokens {
        await channel.send(.response(action: .appendText(token)))
    }
}

// Custom transcript segment for audio
public struct AudioSegment: Transcript.CustomSegment {
    public var id: String
    public var content: URL
}
```

## Takeaways
- The `LanguageModel` + `LanguageModelExecutor` pair is all that is needed to bring any LLM into the Foundation Models ecosystem — app developers see a `LanguageModelSession` and gain structured output, tool calling, and evaluations for free.
- Keep executor `Configuration` stable across calls to preserve KV cache; unnecessary configuration churn increases latency.
- Design auth flows around OAuth + Keychain from the start; never expose API key inputs in your public initializer.
- Be explicit with users about whether data leaves the device — model packages should document this clearly.

---
_Source: WWDC26 Session 339 page (abstract, chapter summaries, code samples, and resource links)._
