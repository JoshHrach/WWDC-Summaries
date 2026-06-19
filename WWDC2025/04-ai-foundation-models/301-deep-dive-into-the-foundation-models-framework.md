# Deep Dive into the Foundation Models Framework
**WWDC25 · Session 301** · [Watch](https://developer.apple.com/videos/play/wwdc2025/301/)

_Platforms:_ iOS 26, macOS Tahoe 26, iPadOS 26, visionOS 26 (requires Apple Intelligence)

## Overview
This session goes deep on the FoundationModels framework, building on the introductory session. It covers how `LanguageModelSession` works internally (tokenization, the context window, stateful transcript), sampling and determinism controls, structured generation internals (constrained decoding), dynamic schemas for runtime-defined output shapes, and autonomous tool calling. The running example is a pixel art coffee shop game where NPCs, dialog, riddles, and contacts-based personalization are all generated on-device.

## Key Topics

### Sessions and the Context Window
Each `LanguageModelSession` maintains a stateful `transcript` of all prompts and responses. The model tokenizes both instructions and prompts before generating output. Context has a finite limit — hitting it throws `.exceededContextWindowSize`. Recovery involves creating a new session with a condensed transcript (e.g., keeping first and last entries). `GenerationOptions` controls sampling: `.greedy` for deterministic output, `temperature` for variance. Language detection errors throw `.unsupportedLanguageOrLocale`. `SystemLanguageModel.default.supportedLanguages` lists supported locales.

### Generable and Constrained Decoding
`@Generable` synthesizes a JSON schema at compile time and an initializer. The model uses constrained decoding: at each token step, impossible tokens (invalid JSON keys, out-of-range numbers) are masked, preventing hallucinated structure. `@Guide` adds per-property constraints: `.range`, `.count`, natural-language descriptions, `.anyOf`, and regex patterns (including regex builder syntax). Enum associated values work in Generable. Properties generate in declaration order — order matters for cross-property influence. Streaming is property-by-property via `PartiallyGenerated<T>`.

### Dynamic Schemas
`DynamicGenerationSchema` constructs schemas at runtime. Properties have names and typed schemas (string, array with element schema, references to other dynamic schemas by name). Multiple schemas are assembled into `GenerationSchema(root:dependencies:)`. The session responds with `GeneratedContent`, queried with `.value(_:forProperty:)`. The model still uses constrained decoding — output matches the dynamic schema exactly.

### Tool Calling
The `Tool` protocol lets the model invoke app-defined code during generation. A tool has `name`, `description`, an inner `@Generable Arguments` struct, and a `call(arguments:) async throws -> ToolOutput` method. Tools are passed at session init. The model determines when to call a tool based on instructions and context; it generates valid arguments via constrained decoding. Tools can be called multiple times per request, and calls may be parallelized. Tools are instances — they can hold state (use `class` for mutable state). `ToolOutput` wraps the return value.

## APIs & Frameworks

**FoundationModels (iOS 26, macOS Tahoe 26)**
- `LanguageModelSession` — stateful conversation; `init(instructions:tools:)`
- `LanguageModelSession.transcript` — `Transcript` with `.entries` array
- `Transcript(entries:)` — create condensed transcript for context carry-over
- `session.respond(to:)` — async text response
- `session.respond(generating: T.Type)` — async structured response
- `session.streamResponse(generating: T.Type)` — `PartiallyGenerated<T>` async sequence
- `GenerationOptions` — `sampling: .greedy`, `temperature: Double`
- `LanguageModelSession.GenerationError.exceededContextWindowSize`
- `LanguageModelSession.GenerationError.unsupportedLanguageOrLocale`
- `SystemLanguageModel.default.supportedLanguages`
- `@Generable` macro — compile-time schema + initializer synthesis
- `@Guide` — property constraints: description, `.range(_:)`, `.count(_:)`, `.anyOf`, `Regex`
- `PartiallyGenerated<T>` — streaming partial output type (auto-generated)
- `DynamicGenerationSchema` — runtime-defined output schema
- `DynamicGenerationSchema.Property(name:schema:)` — property definition
- `DynamicGenerationSchema(type: String.self)` — scalar schema
- `DynamicGenerationSchema(arrayOf:)` — array schema
- `DynamicGenerationSchema(referenceTo:)` — reference to named schema
- `GenerationSchema(root:dependencies:)` — validated schema for inference
- `GeneratedContent` — dynamic response container
- `GeneratedContent.value(_:forProperty:)` — typed property access
- `Tool` protocol — `name`, `description`, `Arguments` (@Generable), `call(arguments:) async throws -> ToolOutput`
- `ToolOutput(_:)` — tool return value wrapper

## Code Highlights
Context window recovery:
```swift
} catch LanguageModelSession.GenerationError.exceededContextWindowSize {
    let entries = previousSession.transcript.entries
    let condensed = [entries.first, entries.last].compactMap { $0 }
    session = LanguageModelSession(transcript: Transcript(entries: condensed))
}
```

Regex guide on a Generable property:
```swift
@Generable struct NPC {
    @Guide(Regex {
        ChoiceOf { "Mr"; "Mrs" }
        ". "
        OneOrMore(.word)
    })
    let name: String
}
```

Dynamic schema construction:
```swift
let riddleSchema = DynamicGenerationSchema(name: "Riddle", properties: [
    .init(name: "question", schema: DynamicGenerationSchema(type: String.self)),
    .init(name: "answers", schema: DynamicGenerationSchema(arrayOf: DynamicGenerationSchema(referenceTo: "Answer")))
])
let schema = try GenerationSchema(root: riddleSchema, dependencies: [answerSchema])
let response = try await session.respond(to: "Generate a coffee riddle", schema: schema)
let question = try response.content.value(String.self, forProperty: "question")
```

Define a tool:
```swift
struct FindContactTool: Tool {
    let name = "findContact"
    let description = "Finds a contact from a specified age generation."
    @Generable struct Arguments {
        let generation: Generation
        @Generable enum Generation { case millennial; case genZ; case genX; case babyBoomers }
    }
    func call(arguments: Arguments) async throws -> ToolOutput {
        // Query Contacts framework...
        return ToolOutput(contactName)
    }
}
let session = LanguageModelSession(tools: [FindContactTool()], instructions: "...")
```

## Takeaways
- Condense the transcript (keep instructions + last response) to recover from `.exceededContextWindowSize` without losing all conversational context.
- Use `@Guide(Regex { ... })` for fine-grained string format constraints — the model generates text matching the regex pattern, not just close to it.
- Use `DynamicGenerationSchema` when output structure is determined at runtime (e.g., player-defined game entities, form builders, dynamic report schemas).
- Tools are invoked in parallel by the model — make tool `call` methods thread-safe; use `class` when stateful tracking (e.g., deduplication) is required.

---
_Source: WWDC25 Session 301 page (abstract, chapter summaries, code samples, and resource links)._
