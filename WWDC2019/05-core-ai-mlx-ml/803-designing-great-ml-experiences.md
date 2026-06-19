# Designing Great ML Experiences
**WWDC19 · Session 803** · [Watch](https://developer.apple.com/videos/play/wwdc2019/803/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15

## Overview
This design-focused session establishes that building a great ML-powered feature requires designing three interconnected layers: the data used to train the model, the metrics used to evaluate it, and the interface that surfaces results and collects feedback. Apple designers from the ML design team walk through each layer with real product examples (Photos search, Face ID, Maps, App Store, Siri watch face, HomeCourt, keyboard) and introduce a set of HIG-documented interface patterns for ML outputs (multiple options, attributions, confidence, limitations) and inputs (calibration, implicit feedback, explicit feedback, corrections). The central argument is that ML is a design decision at every stage, not just a technical one.

## Key Topics

### Designing Data
- Data determines model behavior, which determines the experience — it is the most important decision in an ML product.
- **Collect intentionally** — if specific scenarios matter (e.g., outdoor use), oversample for them. Don't just take a uniform sample.
- **Design for the world you want, not the world that exists** — standard datasets reflect historical biases. Portrait mode's training data was deliberately collected across races, ages, and cultures to achieve inclusive face segmentation.
- Update data as the product specification changes; don't assume a one-time collection is sufficient.

### Designing Metrics
- Metrics encode values: they define what "good" means and what is sacrificed.
- **Understand failures as scenarios, not statistics** — Face ID's one-in-a-million false-accept rate still means real people (identical twins, similar siblings) must use a passcode. Communicate these limitations publicly.
- **Metrics are proxies** — App Store recommendation metrics (time-in-app) can over-optimize for engagement and box users into a narrow genre. Editorial content and diversity metrics counteract this.
- Evolve metrics over time; question them constantly.

### Designing Outputs (Interface Patterns)
- **Multiple options** — When a single best answer is insufficient, present diverse, meaningfully different results (Maps route selection with three distinct paths). Learn from selections as implicit feedback.
- **Attributions** — Explain why a result was shown using objective facts, not subjective profiling ("Because you downloaded NY Times Cooking"). Cite data sources for trust (Siri quoting Wolfram Alpha). Avoid language that implies understanding of personality.
- **Confidence** — Translate model confidence into actionable language rather than raw percentages. Hopper says "Wait" or "Buy Now" instead of "65% chance the price will drop." Use percentage ranges for statistically understood domains (weather). When confidence is low, ask the user for help (Photos asking to confirm identity before labeling).
- **Limitations** — When a feature cannot work (Memoji in dark room, Siri on macOS lacking timers), explain the limitation inline and suggest alternatives. Proactively communicate limitations that affect user safety or trust.

### Designing Inputs (Interface Patterns)
- **Calibration** — Collect only essential information; be fast. HomeCourt calibrates hoop and court in one shot without asking for manual annotation. Face ID scans twice, then auto-adapts. Always show progress, provide guidance, confirm completion, allow data removal.
- **Implicit feedback** — Information arising from natural interactions (apps opened, articles read) used to improve models. Keyboard dynamically resizes touch targets without visible changes; Siri personalizes shortcuts from usage patterns. Respect privacy: give users control over which apps appear in Search suggestions. Implicit feedback is slower but more accurate over time.
- **Explicit feedback** — Asking specific questions about results. Prioritize negative feedback over positive (positive is already captured implicitly). Use descriptive language: "Suggest fewer stories from this source" beats "Dislike." Provide granular options. Act immediately and persistently.
- **Corrections** — Let users fix mistakes using familiar controls (retyping autocorrect, dragging a crop handle). Treat corrections as implicit feedback for future training. The keyboard's autocorrect suppression after a re-typed word is the canonical example.

## APIs & Frameworks

This is a design patterns session, not an API walkthrough. Referenced technologies:

- Core ML — on-device model inference and (new in iOS 13) `MLUpdateTask` for updatable models **[NEW]**
- Create ML — model training
- Vision framework — photo/face/body detection (used by Photos, Face ID, HomeCourt)
- Natural Language framework — text understanding (keyboard, Siri)
- SiriKit / Shortcuts — INInteraction-based app shortcuts surfaced via implicit feedback
- `BGTaskScheduler` — Background Task API for scheduling on-device ML training tasks **[NEW]**
- Human Interface Guidelines: Machine Learning — new section (beta at time of WWDC19)

## Code Highlights

No code samples. This is a design and HIG session.

Pattern reference — Output types:
1. Multiple options → diverse, ranked choices with attributions
2. Attributions → objective explanations, source citations
3. Confidence → actionable language, ranges, or low-confidence prompts
4. Limitations → inline coaching, alternative suggestions

Pattern reference — Input types:
1. Calibration → minimal, guided, one-time with adaptation
2. Implicit feedback → natural interactions as training signal
3. Explicit feedback → negative-first, descriptive labels, immediate effect
4. Corrections → familiar controls, suppress for future, treat as implicit signal

## Takeaways

- Data is design: the training set you choose reflects the experience and the values your product will have — don't delegate this decision to whoever collects the data.
- Never surface raw model confidence as a percentage in isolation; translate it into actions, ranges, or human-readable explanations that help users make actual decisions.
- Prioritize collecting negative explicit feedback over positive: positive preference is already inferrable from implicit signals like reading and sharing; explicit "dislike" is uniquely valuable for understanding unwanted outputs.
- Limitations and failures are design opportunities: acknowledge them inline, explain them in plain language, and suggest alternatives that fulfill the same underlying user goal.

---
_Source: WWDC19 Session 803 page (abstract, full transcript, and resource links)._
