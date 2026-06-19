# Meet the Foundation Models framework
**WWDC25 · Session 286** · [Watch](https://developer.apple.com/videos/play/wwdc2025/286/)

_Platforms:_ iOS 26, iPadOS 26, macOS Tahoe 26

## Overview
The Foundation Models framework gives developers direct, on-device access to the same large language model that powers Apple Intelligence. Running entirely on-device with no network call, the model supports natural language generation, summarization, classification, extraction, and tool-calling — all within a privacy-preserving, latency-friendly local inference pipeline.

The framework provides a straightforward Swift API: create a `LanguageModelSession`, send prompts, and receive streamed or batch responses. A structured output system (`@Generable` macro) lets the model return strongly typed Swift values rather than raw strings, and a `Tool` protocol enables function-calling so the model can invoke app-defined capabilities during reasoning.

## Key Topics

### LanguageModelSession
The central object for interacting with the on-device model. Sessions maintain conversation context across multiple turns, enabling multi-turn dialogue. Each session can be configured with a `SystemPrompt` that establishes persona, constraints, and capabilities.

### Streaming Responses
`session.streamResponse(to:)` returns an `AsyncThrowingStream<String>` that delivers tokens as they are generated, enabling real-time UI updates without waiting for full completion.

### @Generable — Structured Output
Applying `@Generable` to a Swift struct or enum instructs the model to produce output that conforms to that type's schema. The framework handles schema generation, constrained decoding, and JSON-to-Swift mapping automatically. No parsing code needed.

```swift
@Generable
struct TravelPlan {
    var destination: String
    var durationDays: Int
    var activities: [String]
}

let plan: TravelPlan = try await session.respond(
    to: "Plan a 3-day trip to Kyoto",
    generating: TravelPlan.self
)
```

### Tool Protocol — Function Calling
Conforming a Swift type to `Tool` registers it as a callable capability. The model decides when to invoke tools during reasoning, calls them with structured arguments, and incorporates the result into its response. Tools are ideal for live data (weather, calendar, search) that the static model cannot know.

```swift
struct WeatherTool: Tool {
    static let name = "get_weather"
    static let description = "Returns current weather for a city"
    
    struct Arguments: Codable {
        var city: String
    }
    
    func call(arguments: Arguments) async throws -> String {
        // Fetch real weather data
        return "Sunny, 22°C"
    }
}
```

### Guardrails and Safety
The model applies Apple's on-device safety filters. Applications cannot disable these filters. The session describes the categories of content the model will decline to generate and how apps should handle refusal responses.

### Model Availability
The on-device model requires an Apple Intelligence-capable device (iPhone 16 or later, M1 iPad or later, M1 Mac or later) running iOS/iPadOS/macOS 26. The framework reports availability through `LanguageModel.isAvailable`.

## APIs & Frameworks

- **Foundation Models framework** **[NEW]** — on-device LLM access
  - `LanguageModelSession` **[NEW]** — conversational session with context retention
  - `SystemPrompt` **[NEW]** — persona and constraint configuration
  - `session.respond(to:)` **[NEW]** — single-turn prompt, batch response
  - `session.streamResponse(to:)` **[NEW]** — streaming token-by-token response
  - `@Generable` macro **[NEW]** — structured Swift output from model responses
  - `Tool` protocol **[NEW]** — function-calling capability registration
  - `LanguageModel.isAvailable` **[NEW]** — availability check for Apple Intelligence devices

## Code Highlights

```swift
import FoundationModels

// Create a session with a system prompt
let session = LanguageModelSession(
    systemPrompt: "You are a helpful assistant for a recipe app."
)

// Streaming response
let stream = session.streamResponse(to: "Suggest a quick pasta recipe")
for try await token in stream {
    print(token, terminator: "")
}
```

```swift
// Structured output with @Generable
@Generable
struct RecipeSuggestion {
    var name: String
    var ingredients: [String]
    var prepTimeMinutes: Int
}

let recipe = try await session.respond(
    to: "Quick weeknight pasta",
    generating: RecipeSuggestion.self
)
print(recipe.name, recipe.prepTimeMinutes)
```

## Takeaways

- Foundation Models runs entirely on-device — no API key, no network call, no per-request cost, and no user data leaves the device.
- `@Generable` eliminates JSON parsing boilerplate; strongly typed model outputs are the recommended pattern for any structured use case.
- The `Tool` protocol enables real-time data integration (weather, search, calendar) that supplements the model's static knowledge.
- Check `LanguageModel.isAvailable` before presenting AI features; gracefully degrade on unsupported hardware.

---
_Source: WWDC25 Session 286 page (abstract, chapter summaries, code samples, and resource links)._
