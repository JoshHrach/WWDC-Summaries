# Meet Trust Insights
**WWDC26 · Session 379** · [Watch](https://developer.apple.com/videos/play/wwdc2026/379/)

_Platforms:_ iOS 27

## Overview
Trust Insights is a new iOS 27 framework that helps apps detect when a user may be under social coercion — being coached or manipulated by a scammer into taking risky actions such as transferring money or changing account credentials. The framework uses privacy-preserving, on-device machine learning to generate real-time signals without exposing personal data.

This session introduces the Trust Insights framework API, explains how to integrate it into an app, describes how to interpret the framework's output signals, and covers best practices for designing thoughtful user interventions that protect people while fully respecting their privacy. The framework is particularly relevant for apps handling financial transactions, resource transfers, or other high-risk operations.

Trust Insights reflects Apple's broader WWDC26 focus on platform-level trust and safety, making it easy for developers to add a protective layer without building their own coercion-detection ML models.

## Key Topics

**Introduction** (0:00)
Overview of the problem Trust Insights solves: social scams and coercion attacks are increasingly sophisticated, and apps that handle money or sensitive data are prime targets. Trust Insights gives developers a privacy-preserving signal they can act on.

**Generating Insights** (2:35)
How to integrate the Trust Insights client-side Swift API: declaring the required entitlement, constructing an `InsightEvaluator.InsightContext` with an operation category and a parameter pack of requested insights, performing the asynchronous authorization check, and calling `requestEvaluation(context:)` to get an `InsightEvaluation`.

**Feedback Requirements** (6:50)
Trust Insights requires two types of mandatory feedback to keep its models accurate:
1. **Real-time consumption feedback** — report how your app responded to each assessment by calling `assessment.reportConsumption(_:)` with a value such as `.usedIncreasedFriction`.
2. **Offline feedback** — for transactions that later prove fraudulent, submit feedback after the fact so the model can learn.

**Privacy** (9:25)
All Trust Insights signals are computed on-device. Data is minimized, no personal information leaves the device, and users have full control. The framework is designed from the ground up around Apple's privacy principles.

**Best Practices** (10:34)
Where Trust Insights adds the most value in an app's flow, how to combine its signals with existing risk logic, and how to design user-facing interventions that are helpful without being alarming or intrusive.

**Next Steps** (12:48)
Guidance on adoption: identify moments where Trust Insights fits alongside existing logic, follow the documentation and best practices, and register on Apple Business Register to learn about Partner Data Services.

## APIs & Frameworks

**TrustInsights framework** **[NEW]**
- `InsightEvaluator` — the primary type for requesting and evaluating insights
- `InsightEvaluator.InsightContext` — context object specifying the operation category and requested evaluations
  - `operationCategory: .resourceUse` — and other operation categories
  - `requestedEvaluations:` — parameter pack of insight request types
- `IsLikelyBeingCoachedInsight` **[NEW]** — the insight type that detects potential coercion
  - `IsLikelyBeingCoachedInsight.request(schema:modelVersion:)` — creates an insight request
  - `.schema: .version1`
  - `.modelVersion: .current`
- `InsightEvaluation<IsLikelyBeingCoachedInsight>` — the result type from `requestEvaluation`
  - `.insight.outcome` — a `Result` with outcomes: `.unknown`, `.medium`, `.high`
- `evaluator.requestAuthorization(for:)` — async authorization check; must return `.authorized` before evaluation
- `evaluator.requestEvaluation(context:)` — async method returning the assessment
- `assessment.reportConsumption(_:)` **[NEW]** — mandatory real-time feedback method
  - `.usedIncreasedFriction` — example consumption value indicating the app showed friction to the user

**Related**
- App entitlement — required declaration to use Trust Insights
- Apple Business Register — for Partner Data Services enrollment (referenced in Next Steps)

## Code Highlights

Requesting and evaluating a Trust Insights assessment:

```swift
import TrustInsights

let request = IsLikelyBeingCoachedInsight.request(schema: .version1, modelVersion: .current)
let context = InsightEvaluator.InsightContext(
    operationCategory: .resourceUse,
    requestedEvaluations: request
)

let evaluator = InsightEvaluator()
guard try await evaluator.requestAuthorization(for: context) == .authorized else { return }

let assessment = try await evaluator.requestEvaluation(context: context)
do {
    try handleAssessment(assessment)
} catch {
    // Handle error
}

// Required: report how the app responded
assessment.reportConsumption(.usedIncreasedFriction)
```

Handling the insight outcome:

```swift
func handleAssessment(_ assessment: InsightEvaluation<IsLikelyBeingCoachedInsight>) throws {
    switch try assessment.insight.outcome.get() {
    case .unknown:
        // Proceed normally
    case .medium:
        // Show gentle friction or informational UI
    case .high:
        // Show stronger intervention or block the action
    @unknown default:
        break
    }
}
```

## Takeaways
- Any app handling financial transactions, resource transfers, or account changes should evaluate Trust Insights — it provides a meaningful safety signal with minimal integration effort.
- The feedback loop is mandatory, not optional — `reportConsumption(_:)` must be called after every evaluation to keep the model accurate.
- Design interventions carefully: the best practices chapter emphasizes that interventions should be helpful and non-alarming; heavy-handed blocking could harm user experience.
- Trust Insights is on-device only — no data leaves the device, which means no backend integration is required and privacy compliance is built in.

---
_Source: WWDC26 Session 379 page (abstract, chapter list and summaries, code samples, and resource links)._
