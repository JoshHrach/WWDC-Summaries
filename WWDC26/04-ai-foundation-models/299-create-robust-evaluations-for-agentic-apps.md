# Create Robust Evaluations for Agentic Apps
**WWDC26 · Session 299** · [Watch](https://developer.apple.com/videos/play/wwdc2026/299/)

_Platforms:_ iOS 26+, iPadOS 26+, macOS 26+

## Overview
Session 299 covers advanced Evaluations framework features aimed at two specific challenges: scaling datasets beyond what can be hand-written, and evaluating the multi-step tool-calling behaviour of agentic features. It builds directly on the BookTracker app from "Meet the Evaluations framework" and "Improve your prompts by hill-climbing with Evaluations", and is particularly useful for features where getting the right final answer via the wrong tool-call path would be unacceptable.

The first half addresses the dataset problem: 13 hand-written samples are not enough to catch distribution failures. `SampleGenerator` with a custom `sessionProvider` and validator produces hundreds of structurally valid, diverse synthetic samples. Two sampling strategies — `random` and `slidingWindow` — control how seed examples are shown to the generator model.

The second half addresses agentic evaluation: a correct answer can come from the wrong sequence of tool calls, so "what" the model said is insufficient — you also need to verify "how" it got there. `TrajectoryExpectation` and `ToolCallEvaluator` express and score the expected call graph, with argument matchers ranging from strict (`exact`) to semantic (`naturalLanguage`) to structural (`keyOnly`, `range`, `contains`, `oneOf`, `pattern`). Because `ModelSample` and `TrajectoryExpectation` are both `Generable`, the synthetic-data pipeline applies to tool evaluations too.

## Key Topics

### The Dataset Problem (2:21)
Thirteen curated books cover only a narrow slice of the space of possible reviews. The model's evaluation scores on those 13 samples may look strong while failing on genres, review lengths, or writing styles not represented. Coverage matters more than raw sample count.

### Generating Synthetic Data with makeSamples (3:46)
`dataset.makeSamples(prompt, targetCount: 100)` is an async stream that generates new `ModelSample` values from a seed dataset. `targetCount` is the total desired size including seeds; the stream closes when reached. The `prompt` describes what kinds of samples to generate.

### Customising Generation with SampleGenerator (6:27)
`SampleGenerator<ModelSample<BookTags>>(prompt, samples:, targetCount:, sessionProvider:)` provides full control: choose a more capable model (PCC) and add detailed generation instructions. The session is reused across batches; because it can exhaust its context, make the `sessionProvider` closure self-contained so it can be called again fresh.

### Sampling Strategies (8:38)
`samplingStrategy`: `.random` (varied subset of seeds shown as in-context examples — the default) or `.slidingWindow` (sequential seeds — useful for datasets with temporal or logical ordering). Controls which prior samples the generator sees when producing the next one.

### Validating Synthetic Samples (10:11)
Pass a `validator: (sample) -> Bool` closure to `SampleGenerator`. Check review length, tag count, tag casing, and any other invariant. Valid samples accumulate in `generator.samples`; rejected ones in `generator.invalidSamples`, both accessible as async properties during and after the run.

### Comparing Evaluation Results (13:04)
Xcode 27 Evaluations Report lets you compare two runs side by side. The 100-sample evaluation reveals quality scores that were inflated on the 13-sample dataset — the feature only looked good because the seed books were easy. A score drop can mean the prompt needs work, the dataset is harder, or the evaluation criteria have shifted.

### Tool Calling and Tool Evaluations (15:09)
Agentic features call one or more tools in sequence; the right answer via the wrong path is a bug. `ToolCallEvaluator` verifies the session transcript's call structure against a `TrajectoryExpectation`.

### Trajectory Expectations (18:54)
`TrajectoryExpectation` has three lists: `unordered` (calls that must happen, any order), `ordered` (calls that must happen in sequence), and `disallowed` (calls that must not happen). Each entry is a `ToolExpectation(name:, arguments:)` with argument matchers:
- `.exact(argumentName:value:)` — precise match
- `.naturalLanguage(argumentName:criteria:)` — semantic match via model judge
- `.keyOnly(argumentName:)` — any value is acceptable
- `.range(argumentName:range:)` — numeric range
- `.contains` / `.hasPrefix` / `.hasSuffix` / `.oneOf` / `.pattern` — string matchers

### Building a Tool Call Evaluation (21:26)
`ModelSample` with an `expectations: TrajectoryExpectation` parameter feeds a `ToolCallEvaluator(allPass:percentagePass:)`. The evaluator spins up a `LanguageModelSession` with the production tools, runs the prompt, captures the transcript, and scores the call graph. Results appear alongside heuristic and judge metrics in the same Xcode report.

### Synthetic Data for Tool Evaluations (22:02)
`ModelSample` and `TrajectoryExpectation` are `Generable`, so `SampleGenerator` can produce tool-evaluation samples too. Describe the available tools, ordering rules, and argument matchers in the generator prompt. The validator ensures each generated sample has at least one expectation, uses only known tool names, and references real argument matchers.

## APIs & Frameworks

**Evaluations framework**
- `SampleGenerator<S>` **[NEW]** — `init(_:samples:targetCount:sessionProvider:samplingStrategy:validator:)`
  - `sessionProvider: () -> LanguageModelSession` closure
  - `samplingStrategy: .random` / `.slidingWindow` **[NEW]**
  - `validator: (S) -> Bool` closure
  - `run()` → `AsyncStream<S>`
  - `samples: [S]` async property
  - `invalidSamples: [S]` async property
- `dataset.makeSamples(_:targetCount:)` **[NEW]** — shorthand on any dataset
- `Prompt` **[NEW]** — wraps a string generation prompt for `SampleGenerator`
- `TrajectoryExpectation` **[NEW]** — `ordered:`, `unordered:`, `disallowed:`
- `ToolExpectation` **[NEW]** — `name:`, `arguments:`
- Argument matchers **[NEW]**:
  - `.exact(argumentName:value:)` — `.string(_:)`, numeric variants
  - `.naturalLanguage(argumentName:criteria:)`
  - `.keyOnly(argumentName:)`
  - `.range(argumentName:range:)`
  - `.contains(argumentName:substring:)`
  - `.hasPrefix(argumentName:prefix:)` / `.hasSuffix(argumentName:suffix:)`
  - `.oneOf(argumentName:values:)`
  - `.pattern(argumentName:pattern:)`
- `ToolCallEvaluator(allPass:percentagePass:)` **[NEW]**
- `ModelSample(prompt:instructions:expectations:)` **[NEW]** overload — for tool evaluations
- Xcode 27 Evaluations Report — comparison view between two runs **[NEW]**

**FoundationModels**
- `@Generable` — `ModelSample` and `TrajectoryExpectation` conform (enabling synthetic generation)
- `LanguageModelSession(model:tools:instructions:)`
- `PrivateCloudComputeLanguageModel()` — recommended for synthetic data generation

## Code Highlights

```swift
// Synthetic data generation with custom session
let generator = SampleGenerator<ModelSample<BookTags>>(
    prompt,
    samples: dataset,
    targetCount: 100,
    sessionProvider: {
        LanguageModelSession(
            model: PrivateCloudComputeLanguageModel(),
            instructions: "Generate diverse book reviews with 3-8 lowercase tags..."
        )
    },
    validator: { sample in
        guard let book = sample.expected else { return false }
        return sample.promptDescription.count >= 100 && (3...8).contains(book.tags.count)
    }
)

// Ordered tool-call trajectory
TrajectoryExpectation(
    ordered: [
        ToolExpectation("searchBooks", arguments: [.exact(argumentName: "tag", value: .string("gothic"))]),
        ToolExpectation("getBookDetails", arguments: [.keyOnly(argumentName: "bookId")])
    ]
)

// Disallow unexpected tool calls
TrajectoryExpectation(
    unordered: [ToolExpectation("searchBooks", arguments: [.naturalLanguage(argumentName: "genre", criteria: "science fiction")])],
    disallowed: [ToolExpectation("findSimilarBooks")]
)
```

## Takeaways
- Synthetic data generation is not optional for production evaluation suites — a 13-sample dataset can make a poor feature look strong; aim for 100+ diverse samples to get representative scores.
- Add a `validator` to every `SampleGenerator` to enforce structural invariants (length, count, casing) automatically; rejected samples accumulate in `invalidSamples` for inspection.
- For agentic features, build both an output evaluation (what the model returns) and a tool-call evaluation (how it gets there) and run them in the same Swift Testing suite for end-to-end coverage.
- Because `TrajectoryExpectation` is `Generable`, you can synthesise tool-evaluation test cases — scaling agentic evaluation the same way you scale output evaluation.

---
_Source: WWDC26 Session 299 page (abstract, chapter summaries, code samples, and resource links)._
