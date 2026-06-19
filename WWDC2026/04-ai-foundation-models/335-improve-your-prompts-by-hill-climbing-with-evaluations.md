# Improve Your Prompts by Hill-Climbing with Evaluations
**WWDC26 · Session 335** · [Watch](https://developer.apple.com/videos/play/wwdc2026/335/)

_Platforms:_ iOS 26+, iPadOS 26+, macOS 26+

## Overview
Session 335 picks up where "Meet the Evaluations framework" leaves off, teaching a systematic, scientific approach to improving intelligence features using evaluation scores as a feedback signal — a technique the session calls hill-climbing. The running example is a BookTracker app whose auto-tagging feature produces tags that fail on relevance and usefulness; every iterative change is validated (or falsified) with a quantitative metric before being accepted.

A central concern is judge alignment: when a `ModelJudgeEvaluator` disagrees with an expert human rater, the evaluation is untrustworthy. The session introduces Cohen's kappa as the correct alignment metric, then builds a full judge-calibration evaluation to measure and fix the drift. Three improvement strategies are demonstrated in order — refining score-dimension descriptions, adding few-shot worked examples to the judge, and moving beyond prompts to adding a `BookLookupTool` — each compared with the previous baseline using Xcode 27's side-by-side evaluation comparison view.

The core message is methodological: treat every change (instruction text, score dimension wording, judge examples, tool calling) as a controlled variable, change one thing at a time, let the numbers decide, and expect failed experiments.

## Key Topics

### BookTracker's Tagging Problem (2:42)
The existing `BookTaggingEvaluation` scores tags with heuristic evaluators (tag count, genre presence, no duplicates) and a `ModelJudgeEvaluator` on two `ScoreDimension`s — Relevance and Usefulness. After adding two new books to the dataset, the human expert and model judge disagree on usefulness for one entry, surfacing a drift problem.

### Judge-Human Drift (8:26)
Drift — divergence between judge and expert — grows as the dataset expands. A drifted judge produces misleading aggregate scores, making hill-climbing unreliable. The fix is to align the judge to expert opinion before trusting evaluation numbers.

### Measuring Drift with Cohen's Kappa (9:37)
Accuracy alone is misleading on skewed score distributions (a judge that always picks 4 can score 80% accuracy by luck). Cohen's kappa subtracts the probability of chance agreement from observed agreement and normalises the result. The session targets κ ≥ 0.6 (moderate agreement) as a threshold for trusting the judge.

### Building a Judge Alignment Evaluation (12:26)
Extract summary/tag pairs from a prior evaluation run's JSON attachment, add expert ratings, and build a `BookTagJudgmentCalibration` evaluation: the subject just returns the already-generated tags, the evaluator is the same `ModelJudgeEvaluator`, and `aggregateMetrics` computes Cohen's kappa alongside mean and standard deviation. A Swift Testing `#expect` asserts κ > 0.6 for both dimensions.

### Comparative Evaluation (17:16)
Xcode 27 can run a control evaluation and an experimental evaluation side-by-side, showing per-dimension delta. This makes it easy to see when a prompt change improves relevance but hurts usefulness — a tradeoff to weigh explicitly.

### Refining Score Dimensions (19:12)
Tightening `ScoreDimension` descriptions (adding concrete good/bad tag examples, calling out genre as critical, and naming failure modes like "reader reactions" and "meta-commentary") increases both relevance and usefulness alignment without changing the judge model.

### Few-Shot Examples in the Judge (21:23)
Ground the judge with 4–6 worked examples (book + tags + expert ratings) in the `ModelJudgePrompt.instructions`. Keep the set small to avoid overfitting the alignment score on those specific books. With calibrated examples, both κ scores clear the 0.6 threshold.

### Going Beyond Prompts: Adding a Tool (23:38)
A `BookLookupTool` gives the on-device tagger access to the book's title and author from distinguishing review details. `BookTaggingService.generateTags(for:tools:)` grows a `tools` parameter (default empty for backwards compatibility). A `BookTaggingWithLookupEvaluation` compares with versus without the tool, and the tool version scores higher — though a 13-sample dataset is too small for statistical confidence.

## APIs & Frameworks

**Evaluations framework**
- `Evaluation` protocol — `subject(from:)`, `dataset`, `evaluators`, `aggregateMetrics(using:)`
- `ModelSample<T>` — `prompt`, `expected`
- `Metric` — `.passing(rationale:)`, `.failing(rationale:)`
- `Evaluator` — closure-based pass/fail checker
- `ModelJudgeEvaluator` **[NEW usage]** — `judge:`, `dimensions:`, `prompt:`
- `ModelJudgePrompt` — `instructions:`, `evaluationTarget:`, `reference:`
- `ScoreDimension` — `name`, `description`, `scale: .numeric([Int: String])`
- `MetricsAggregator` — `.computeMean(of:)`, `.computeStandardDeviation(of:)`, `.custom(of:label:_:)`
- `EvaluationContext.current.result`
- `result.aggregateValue(.mean(of:))`, `result.aggregateValue(.custom(label:))`
- `.evaluates(_:info:)` Swift Testing trait
- `ArrayLoader(samples:)` — dataset loader
- `cohensKappa(ratings1:ratings2:)` — custom aggregation metric

**FoundationModels**
- `SystemLanguageModel(guardrails: .permissiveContentTransformations)` — used in BookTaggingService
- `LanguageModelSession(model:tools:instructions:)`
- `session.respond(to:generating:)`
- `Tool` protocol — `name`, `description`, `@Generable struct Arguments`, `call(arguments:)`
- `@Generable` / `@Guide`
- `PrivateCloudComputeLanguageModel()` — used as judge model

## Code Highlights

```swift
// Cohen's kappa custom aggregation
aggregator.custom(of: relevance.metric, label: "Relevance Alignment Score") { judgeScores in
    cohensKappa(ratings1: expertRelevance, ratings2: judgeScores) ?? 0
}

// Alignment test threshold
#expect(result.aggregateValue(.custom(label: "Relevance: Judge vs Expert")) > 0.6)

// BookLookupTool — grounds tagger with book metadata
struct BookLookupTool: Tool {
    let name = "lookupBook"
    let description = "Looks up title and author given distinguishing details from a review."
    @Generable struct Arguments { var details: String }
    func call(arguments: Arguments) async throws -> Output { /* fuzzy search */ }
}
```

## Takeaways
- Verify judge alignment with Cohen's kappa before using model-judge scores to guide decisions; a κ < 0.6 means the judge cannot be trusted.
- Build a dedicated judge-calibration evaluation using your own expert ratings — this is distinct from the feature evaluation and needs its own dataset and `#expect` assertions.
- Change one variable per hill-climbing iteration (dimension description, few-shot examples, tool usage) and use Xcode's side-by-side comparison to isolate the effect.
- When prompts plateau, consider adding tools — but scale up the dataset before drawing statistical conclusions from small sample sizes.

---
_Source: WWDC26 Session 335 page (abstract, chapter summaries, code samples, and resource links)._
