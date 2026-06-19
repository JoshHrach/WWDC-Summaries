# LLM Search Using Core Spotlight
**WWDC26 · Session 246** · [Watch](https://developer.apple.com/videos/play/wwdc2026/246/)

_Platforms:_ iOS, iPadOS, macOS, visionOS

## Overview
This session shows how to upgrade basic keyword search into a retrieval-augmented generation (RAG) system by connecting a `LanguageModelSession` to your app's Core Spotlight index via the new `SpotlightSearchTool`. The language model translates natural language queries into Spotlight searches, retrieves relevant indexed items, and generates a grounded, contextual response — all on-device.

Using a hiking trails app as the running example, the session covers the complete integration: donating searchable content, configuring the tool, implementing an index delegate for full item hydration, customizing guidance profiles, implementing contact resolution for person references, building custom pipeline stages for domain-specific computation (e.g., sentiment scoring), and evaluating search quality with the Evaluations framework.

## Key Topics

### Grounding Answers with Spotlight Tool-Calling
A plain `LanguageModelSession` answers from world knowledge; `SpotlightSearchTool` grounds its answers in your app's indexed content. The tool implements the Foundation Models `Tool` protocol, allowing the LLM to call it when a query requires specific app data.

### Configure and Add SpotlightSearchTool
Create `SpotlightSearchTool()` — optionally with a `Configuration` specifying data sources (e.g., `.files` for sandbox file paths). Add it to a `LanguageModelSession(model:tools:)`. The model then calls the tool automatically when user queries require indexed content.

### Displaying Results and Partial Replies
Results arrive as an async sequence on `tool.searchResults`. Each `SearchReply` has a `queryToken` (to detect new queries across multiple tool calls per session response) and a `.content` with typed payload. Display the `session.respond()` result for an assistant UI and `searchItems` for a list UI.

### Index Delegate for Full Item Hydration
Some Spotlight attributes are stored compactly and aren't passed to the model. Implement `CSSearchableIndexDelegate.searchableItems(forIdentifiers:)` to re-fetch full `CSSearchableItem` objects on demand, attaching additional attributes for the model to reason over.

### Guidance Profiles
`SpotlightSearchTool.GuidanceProfile` scopes what search capabilities the model is guided to use — specific attribute names, date matching, people matching — important for fitting the smaller context window of on-device models. Use `.focused(.items)` for tightest guidance with on-device models.

### Contact Resolver
Implement `ContactResolver.userIdentity()` returning a `ResolvedContact` with `displayName`, `emailAddresses`, and `names`. Assign to `tool.contactResolver` so the model can resolve references like "who did I go hiking with?" to concrete identities.

### Custom Pipeline Stages
Register `@Generable` types conforming to `CustomStage` to add domain-specific computation to the search pipeline. The model generates stage parameters and the stage returns typed `SearchPipelineData` (e.g., `scoredItems`, `groupedItems`, `table`, `statistic`). Stages are registered in `SpotlightSearchTool.Configuration(customStages:)`.

### Evaluating Response Quality
Use the `Evaluations` framework with `ModelSampleProtocol` to define typed test cases. Specify `TrajectoryExpectation` with `ToolExpectation` entries to assert the model called the right tool with the right arguments. Use `Sample Generation APIs` to expand a seed dataset. Assert aggregate metrics like result coverage mean in standard Swift `@Test` functions.

## APIs & Frameworks

**CoreSpotlight** **[Updated]**
- `SpotlightSearchTool` — **[NEW]** Foundation Models tool backed by Core Spotlight index
  - `init()` — default init; searches app's Core Spotlight index
  - `init(configuration:)` — custom configuration
  - `.searchResults` — `AsyncSequence` of `SearchReply`
  - `.contactResolver` — assigned `ContactResolver` for person reference resolution **[NEW]**
- `SpotlightSearchTool.Configuration` — **[NEW]** tool configuration
  - `init(sources:)` — data sources (e.g., `.files`)
  - `init(guide:)` — guidance level configuration
  - `init(customStages:)` — custom pipeline stage registration
- `SpotlightSearchTool.GuidanceProfile` — **[NEW]** scopes model guidance
  - `init(textMatch:dates:people:attributes:)` — enable/disable specific guidance features
- `SpotlightSearchTool.Guide` — **[NEW]** wraps a guidance level
  - `.level` — `GuidanceLevel`: `.dynamic(profile)`, `.focused(.items)`, etc.
- `SearchReply` — **[NEW]** individual result from `tool.searchResults`
  - `.queryToken` — opaque token identifying the originating query
  - `.content` — `SearchReplyContent` enum
  - `.label` — display label
- `SearchReplyContent` — **[NEW]** enum of payload types
  - `.items([CSSearchableItem])` — matched searchable items
  - `.scoredItems([ScoredItem])` — items with relevance scores
  - `.groupedItems([GroupedItems])` — items organized into groups
  - `.count(Int)` — result count
  - `.table(SearchTable)` — tabular result
  - `.statistic(SearchStatistic)` — computed statistic
  - `.text(String)` — plain text result
- `CustomStage` protocol — **[NEW]** defines a custom pipeline stage
  - `static var name: String` — stage name
  - `static var description: String` — stage description
  - `static var inputTypes: [SearchPipelineDataType]` — expected input types
  - `static var outputTypes: [SearchPipelineDataType]` — produced output types
  - `func execute(on:) async throws -> SearchPipelineData` — stage execution
- `SearchPipelineData` — **[NEW]** typed payload for pipeline stage input/output
- `SearchPipelineDataType` — **[NEW]** enum: `.items`, `.scoredItems`, `.groupedItems`, etc.
- `ContactResolver` protocol — **[NEW]** provides user identity for person reference resolution
  - `func userIdentity() -> ResolvedContact`
- `ResolvedContact` — **[NEW]** struct with `displayName`, `emailAddresses`, `names`
- `CSSearchableIndexDelegate.searchableItems(forIdentifiers:) async -> [CSSearchableItem]` — delegate method for full item hydration (existing protocol, new async variant implied)
- `CSSearchableItem` — existing Spotlight item type
- `CSSearchableItemAttributeSet` — existing attribute set

**FoundationModels**
- `LanguageModelSession(model:tools:instructions:)` — session with tools
- `session.respond(to:)` — generates grounded response using tool results
- `SystemLanguageModel` — on-device Apple Foundation Model
- `Tool` protocol — `SpotlightSearchTool` conforms to this
- `@Generable` — used for `CustomStage` argument types and evaluation types
- `@Guide(description:)` — provide LLM guidance for generated parameters

**Evaluations framework** **[NEW]**
- `ModelSampleProtocol` — define typed evaluation test cases
  - `associatedtype ExpectedValue`
  - `associatedtype Expectation`
  - `var input: ModelSampleInput`
  - `var output: ModelSampleOutput<ExpectedValue, Expectation>`
- `TrajectoryExpectation` — **[NEW]** assert the sequence of tool calls
  - `init(unordered:)` — unordered set of tool call expectations
- `ToolExpectation` — **[NEW]** assert a specific tool call
  - `init(_:arguments:)` — tool name and argument expectations
  - `.keyOnly(argumentName:)` — assert argument key presence without value check
- Sample Generation APIs — expand seed evaluation datasets
- `Metric` — named evaluation metric
- `evaluation.run()` — execute evaluation suite
- `result.aggregateValue(.mean(of:))` — compute aggregate metric value

**Related Resources**
- [Spotlight search tool documentation](https://developer.apple.com/documentation/CoreSpotlight/Spotlight-search-tool)
- [Making your indexed content available to Foundation Models](https://developer.apple.com/documentation/CoreSpotlight/making-your-indexed-content-available-to-foundation-models)

## Code Highlights

Minimal setup:
```swift
import CoreSpotlight
import FoundationModels

let tool = SpotlightSearchTool()
let session = LanguageModelSession(model: model, tools: [tool], instructions: instructions)
let response = try await session.respond(to: "What hikes have I gone on?")
```

Dynamic guidance profile for on-device models:
```swift
let profile = SpotlightSearchTool.GuidanceProfile(
    textMatch: true, dates: true, people: false,
    attributes: [.title, .altitude, .completionDate]
)
let tool = SpotlightSearchTool(configuration: .init(guide: .init(level: .dynamic(profile))))
```

Custom sentiment-scoring pipeline stage:
```swift
@Generable
struct HappinessStage: CustomStage {
    static var name = "happiness"
    static var inputTypes: [SearchPipelineDataType] = [.items]
    static var outputTypes: [SearchPipelineDataType] = [.scoredItems]
    @Guide(description: "Minimum happiness score (0.0-1.0) to include")
    var threshold: Double?
    func execute(on input: SearchPipelineData) async throws -> SearchPipelineData { ... }
}
let tool = SpotlightSearchTool(configuration: .init(customStages: [.happinessBoost(threshold: 0.5)]))
```

Evaluation test:
```swift
@Test func trailSearchEval() async throws {
    let evaluation = TrailSearchEvaluation(tool: Self.makeSearchTool(), dataset: ArrayLoader(samples: samples))
    let result = try await evaluation.run()
    let coverageMean = result.aggregateValue(.mean(of: Metric("ResultCoverage")))
    #expect(coverageMean >= 0.5)
}
```

## Takeaways
- `SpotlightSearchTool` is the fastest path to RAG in an Apple app — one line to create the tool, one more to add it to a session, and the LLM handles query translation automatically.
- Implement `CSSearchableIndexDelegate.searchableItems(forIdentifiers:)` to ensure the model receives full, rich item attributes — compact stored metadata limits answer quality.
- Use focused or dynamic guidance profiles for on-device models to stay within their smaller context window; always profile guidance impact on result quality.
- The Evaluations framework enables regression testing of LLM search quality as a standard Swift `@Test` — adopt it early to prevent quality regressions as you iterate on indexing and stage configurations.

---
_Source: WWDC26 Session 246 page (abstract, chapter summaries, code samples, and resource links)._
