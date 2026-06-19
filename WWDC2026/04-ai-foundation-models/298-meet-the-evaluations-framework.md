# Meet the Evaluations Framework
**WWDC26 · Session 298** · [Watch](https://developer.apple.com/videos/play/wwdc2026/298/)

_Platforms:_ iOS 26+, iPadOS 26+, macOS 26+, watchOS 26+, visionOS 26+

## Overview
Session 298 introduces the Evaluations framework, Apple's new answer to a fundamental problem with generative AI features: the same input can produce different outputs on different runs, so conventional unit tests can't verify correctness. The framework provides a structured pipeline — dataset, subject, evaluators, metrics, aggregation — that runs over many samples, quantifies how often the feature behaves correctly, and integrates with Swift Testing and Xcode's new Evaluation Report.

The session uses the BookTracker demo app throughout. Its `BookTaggingService` auto-generates tags from reader reviews; the session builds an evaluation from scratch: first heuristic metrics (tag count, word count, genre presence), then a `ModelJudgeEvaluator` that uses Private Cloud Compute to assess qualitative tag quality on a numeric scale, then `ScoreDimension`s to split the judge's single score into separate Relevance and Usefulness assessments. By the end, the evaluation suite drives a real prompt-improvement cycle — evaluation-driven development.

The session closes with best practices: start with 20–30 focused samples, use code-based evaluators for anything measurable, use model judges for qualitative judgements, and let rationales guide the next change.

## Key Topics

### Why Standard Tests Fail (0:00)
Generative AI features are probabilistic: a tag generator might produce perfect output 90% of the time and subtly wrong output 10%. Unit tests catch the 10% only if the developer anticipated it. Evaluations measure the overall rate of expected behaviour across a representative sample set.

### Building Your First Evaluation (4:31)
Conform a struct to `Evaluation`. Five steps: implement `subject(from:)` (the code under test — call your feature, return a `ModelSubject`), define `dataset` (an `ArrayLoader` of `ModelSample` values with prompts and optional expected values), declare one or more `Metric` instances, add `Evaluator` closures that call `metric.passing(rationale:)` or `metric.failing(rationale:)`, and implement `aggregateMetrics(using:)` to produce summary statistics.

### Running Evaluations and Reading Reports (8:06)
Attach `.evaluates(evaluation, info:)` to a Swift Testing `@Test` and assert on aggregated values with `#expect`. Xcode 27 shows a dedicated Evaluation Report: per-sample breakdowns, prompts, measurements, and the full model response for each entry.

### Building Robust Datasets (10:57)
Two samples are not enough. Aim for thousands, with variety across genres, review lengths, fiction/non-fiction, and personal styles. Hand-authoring doesn't scale, so `SampleGenerator.makeSamples(_:targetCount:)` synthesises new samples from a seed set using the model itself.

### Evaluators and Metrics (14:20)
Multiple evaluators compose in the `Evaluators` result builder. Go beyond pass/fail by using a scoring evaluator with a numeric `Metric` value for things like tag count distribution. Aggregate with `.computeMean`, `.computeStandardDeviation`, `.computeVariance`, and grouping.

### Model Judges (16:12)
A `ModelJudgeEvaluator` applies a second model — at least as capable as the one being evaluated — to score output qualitatively and consistently. Provide a 1-to-4 scale (even number avoids a neutral middle), a judge model (e.g., `PrivateCloudComputeLanguageModel()`), and optionally a `ModelJudgePrompt` with app-specific context and reference values. The judge returns a score plus a rationale string.

### Score Dimensions (21:19)
When a single quality question is too broad, split it into `ScoreDimension`s: separate `Relevance` and `Usefulness` instances each with their own description and numeric scale. Pass the array to `ModelJudgeEvaluator(judge:dimensions:prompt:)`. The Evaluation Report separates the scores so you can see which dimension is failing independently.

### Evaluation-Driven Development (15:41)
The loop: observe a failing `#expect`, analyse per-sample rationales, make exactly one change (an instruction, a `@Guide` parameter, a tool), re-run, and check the delta. Centering development on this loop is evaluation-driven development.

## APIs & Frameworks

**Evaluations (new framework)**
- `Evaluation` protocol **[NEW]** — `subject(from:)`, `dataset`, `evaluators`, `aggregateMetrics(using:)`
- `ModelSubject<T>` **[NEW]** — wraps the output of the feature under test
- `ModelSample<T>` **[NEW]** — `prompt`, `expected`
- `ArrayLoader(samples:)` **[NEW]** — in-memory dataset loader
- `Metric` **[NEW]** — `passing(rationale:)`, `failing(rationale:)`, scoring variant
- `Evaluator` **[NEW]** — closure-based; receives `(sample, subject)` → `MetricResult`
- `Evaluators` result builder **[NEW]**
- `MetricsAggregator` **[NEW]** — `.computeMean(of:)`, `.computeStandardDeviation(of:)`, `.computeVariance(of:)`, `.group(_:_:)`
- `ModelJudgeEvaluator` **[NEW]** — `judge:`, `scale:`, `dimensions:`, `prompt:`
- `ModelJudgePrompt` **[NEW]** — `instructions:`, `evaluationTarget:`, `reference:`
- `ScoreDimension` **[NEW]** — `name`, `description`, `scale: .numeric([Int: String])`
- `SampleGenerator` **[NEW]** — `.makeSamples(_:targetCount:)` async stream
- `EvaluationContext.current.result` **[NEW]**
- `result.aggregateValue(.mean(of:))` **[NEW]**
- `.evaluates(_:info:)` Swift Testing trait **[NEW]**

**FoundationModels**
- `@Generable` / `@Guide(description:_:)` — `.count(3...8)` range guide
- `LanguageModelSession(model:tools:instructions:)`
- `SystemLanguageModel(guardrails: .permissiveContentTransformations)`
- `PrivateCloudComputeLanguageModel()` — recommended judge model

## Code Highlights

```swift
// Minimal Evaluation conformance
struct BookTaggingEvaluation: Evaluation {
    func subject(from sample: ModelSample<BookTags>) async throws -> ModelSubject<BookTags> {
        let result = try await BookTaggingService.generateTags(for: sample.promptDescription)
        return ModelSubject(value: result)
    }
    var dataset = ArrayLoader(samples: Book.sampleBooks.map {
        ModelSample(prompt: $0.review, expected: BookTags(tags: $0.tags))
    })
    let tagCount = Metric("TagCount")
    var evaluators: Evaluators {
        Evaluator { _, subject in
            let count = subject.value.tags.count
            return (3...8).contains(count) ? tagCount.passing(rationale: "\(count) tags")
                                           : tagCount.failing(rationale: "Got \(count)")
        }
    }
    func aggregateMetrics(using aggregator: inout MetricsAggregator) {
        aggregator.computeMean(of: tagCount)
    }
}

// Swift Testing integration
@Test("Book Tag Evaluations", .evaluates(evaluation, info: evaluationInfo))
func evaluateBookTagging() async throws {
    #expect(EvaluationContext.current.result.aggregateValue(.mean(of: evaluation.tagCount)) >= 0.8)
}
```

## Takeaways
- Start every intelligence feature with an `Evaluation` — even 20 hand-written samples reveal systematic problems that single-run testing misses.
- Use code-based `Evaluator` closures for measurable properties (count, format, vocabulary) and `ModelJudgeEvaluator` for qualitative assessments — they compose in the same `Evaluators` builder.
- Split broad quality questions into multiple `ScoreDimension`s so rationales pinpoint the specific dimension that is failing.
- Make the Evaluation loop the driver of development: failing `#expect` → analyse rationales → change one thing → re-run; that is hill-climbing in practice.

---
_Source: WWDC26 Session 298 page (abstract, chapter summaries, code samples, and resource links)._
