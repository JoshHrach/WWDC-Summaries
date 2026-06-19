# Code-Along: Bring On-Device AI to Your App Using the Foundation Models Framework
**WWDC25 · Session 259** · [Watch](https://developer.apple.com/videos/play/wwdc2025/259/)

_Platforms:_ iOS 26, macOS Tahoe 26, iPadOS 26, visionOS 26 (requires Apple Intelligence)

## Overview
This code-along session guides developers through building an on-device AI feature from scratch using the `FoundationModels` framework. Starting with a simple prompt-response interaction, it progressively adds structured output with `@Generable`, guided generation with `@Guide`, streaming partial results with `PartiallyGenerated`, and autonomous tool use with the `Tool` protocol. The session also covers performance optimization with `session.prewarm()` and profiling with the new Foundation Models Instrument in Xcode 26.

The example app is a trip planning assistant that generates itineraries, structured destination data, and live recommendations using the on-device LLM.

## Key Topics

### FoundationModels Framework Basics
The `FoundationModels` framework provides direct access to an on-device large language model via Swift async/await. `SystemLanguageModel.default` is the handle to the model. Availability must be checked (`.available`, `.notSupported`, `.notEnabled`, `.notReady`) before use. `LanguageModelSession` wraps a conversation; custom instructions set the session persona.

### Structured Generation with @Generable
The `@Generable` macro conformances a Swift struct or enum to the `Generable` protocol, automatically synthesizing a JSON schema and a decodable initializer. The model uses constrained decoding to guarantee output matching the schema — no manual parsing required. `@Guide` macro variants add further constraints: natural-language descriptions, numeric ranges (`.range(1...5)`), array counts (`.count(3)`), enum membership, and regex patterns.

### Streaming with PartiallyGenerated
For `@Generable` types, the framework auto-synthesizes a `PartiallyGenerated<T>` type with optional properties. `session.streamResponse(generating: T.self)` yields `PartiallyGenerated<T>` values as generation proceeds, enabling live UI updates property by property before the full response is available.

### Tool Protocol
The `Tool` protocol lets the LLM call app-defined functions during generation. Each tool has a `name: String`, `description: String`, an inner `@Generable Arguments` struct, and a `call(arguments:) async throws -> ToolOutput` method. The model decides autonomously when to invoke tools and uses constrained decoding to generate valid arguments. Tools are passed at session creation.

### Performance
`session.prewarm()` loads the model into memory before the first request, reducing initial latency. `IncludeSchemaInPrompt` option can be set to `false` for Generable types whose schema is already known to the model, trimming token count. The Foundation Models Instrument (new in Xcode 26) visualizes token generation rate, tool call events, and latency breakdowns.

## APIs & Frameworks

**FoundationModels (iOS 26, macOS Tahoe 26)**
- **[NEW]** `SystemLanguageModel` — handle to the on-device model; `.default`
- **[NEW]** `SystemLanguageModel.availability` — `.available`, `.notSupported`, `.notEnabled`, `.notReady`
- **[NEW]** `LanguageModelSession` — stateful session; `init(instructions:tools:)`
- **[NEW]** `session.respond(to:)` — async prompt-response
- **[NEW]** `session.respond(generating: T.Type)` — async structured generation
- **[NEW]** `session.streamResponse(generating: T.Type)` — streaming `PartiallyGenerated<T>` sequence
- **[NEW]** `session.prewarm()` — preload model into memory
- **[NEW]** `@Generable` macro — synthesize schema + initializer for Swift types
- **[NEW]** `@Guide` macro — add constraints: description, `.range`, `.count`, `.anyOf`, Regex
- **[NEW]** `PartiallyGenerated<T>` — auto-synthesized type with optional properties for streaming
- **[NEW]** `Tool` protocol — `name`, `description`, `Arguments` (@Generable), `call(arguments:)`
- **[NEW]** `ToolOutput` — return type from tool `call` method
- **[NEW]** `GenerationOptions` — `.greedy` sampling, `temperature`, `IncludeSchemaInPrompt`
- **[NEW]** `LanguageModelSession.GenerationError` — `.exceededContextWindowSize`, `.unsupportedLanguageOrLocale`
- **[NEW]** `Transcript` — session history; entries array for context carry-over

**Xcode 26**
- **[NEW]** Foundation Models Instrument — profiles token generation, tool calls, latency

## Code Highlights
Check model availability and create a session:
```swift
guard SystemLanguageModel.default.availability == .available else { return }
let session = LanguageModelSession(instructions: "You are a travel planning assistant.")
```

Generate structured output:
```swift
@Generable
struct Destination {
    @Guide(description: "City and country name")
    let name: String
    @Guide(.count(3))
    let highlights: [String]
    @Guide(.range(1...5))
    let rating: Int
}

let response = try await session.respond(generating: Destination.self) {
    "Suggest a travel destination for a beach vacation."
}
let destination: Destination = response.content
```

Stream partial results:
```swift
for try await partial in session.streamResponse(generating: Destination.self, prompt: "...") {
    // partial.name may be nil until that property is generated
    updateUI(with: partial)
}
```

Define and use a tool:
```swift
struct WeatherTool: Tool {
    let name = "getWeather"
    let description = "Get current weather for a city."
    @Generable struct Arguments { let city: String }
    func call(arguments: Arguments) async throws -> ToolOutput {
        let weather = try await fetchWeather(city: arguments.city)
        return ToolOutput(weather)
    }
}
let session = LanguageModelSession(tools: [WeatherTool()], instructions: "...")
```

## Takeaways
- Check `SystemLanguageModel.default.availability` before every session; handle `.notReady` with a retry or graceful fallback.
- Use `@Generable` + `@Guide` for all structured output — constrained decoding eliminates parsing fragility and hallucinated fields.
- Stream via `session.streamResponse(generating:)` + `PartiallyGenerated<T>` to update UI progressively without waiting for full generation.
- Call `session.prewarm()` in a background task before the user triggers AI features to minimize perceived latency.

---
_Source: WWDC25 Session 259 page (abstract, chapter summaries, code samples, and resource links)._
