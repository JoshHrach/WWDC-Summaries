# Discover Machine Learning & AI Frameworks on Apple Platforms
**WWDC25 · Session 360** · [Watch](https://developer.apple.com/videos/play/wwdc2025/360/)

_Platforms:_ iOS 26, iPadOS 26, macOS Tahoe 26, visionOS 26, watchOS 26

## Overview
This survey session maps the entire Apple ML/AI landscape across four layers: platform intelligence (built-in OS features and Apple Intelligence), programmatic ML-powered APIs, model deployment tools (Core ML, Core ML Tools), and frontier exploration (MLX). It is the authoritative starting-point guide for choosing the right tool for a given task — whether you are adding a few lines of code to tap into an on-device model or fine-tuning a large language model on Apple Silicon.

The session highlights the newly introduced Foundation Models framework (iOS 26), the new `SpeechAnalyzer` API, Vision document recognition and lens smudge detection, and new MLX and BNNSGraph capabilities.

## Key Topics

### Platform Intelligence (Built-in)
- Writing Tools, Genmoji, Image Playground — automatic in standard text views; a few lines for custom views.
- `ImageCreator` (ImagePlayground framework, iOS 18.4+) — programmatic image generation.
- **Smart Reply API** (iOS 18.4) — donate conversation context via `UIMessageConversationContext` / `UIMailConversationContext` to a keyboard; handle `insertInputSuggestion` for mail.
- **Foundation Models framework** (iOS 26) — **[NEW]** on-device language model for summarization, extraction, classification, and more.

### Foundation Models Framework Highlights
- Three-line usage: `import FoundationModels`, create a `Session`, call `session.respond(to:)`.
- **Guided Generation** — mark app types with `@Generable` and property-level natural-language guides; the framework customizes the decoding loop to produce correctly-typed structured responses.
- **Tool Calling** — extend the model with live/personal data (weather, calendar events) and real-world actions.
- Entirely on-device, works offline, no API keys required, no per-request cost.
- Tight Xcode integration for iteration and testing.

### ML-Powered APIs
- **Vision** — 30+ image/video analysis APIs; two new additions:
  - Document recognition — groups document structures (paragraphs, headings, tables). **[NEW]**
  - Lens smudge detection. **[NEW]**
- **Natural Language** — language ID, part-of-speech tagging, named entity recognition.
- **Translation** — text translation between multiple languages.
- **Sound Analysis** — recognizes categories of environmental sound.
- **Speech** — new `SpeechAnalyzer` API **[NEW]** replaces `SFSpeechRecognizer` for long-form/distant audio (lectures, meetings); new faster, more flexible on-device model.
- **Create ML** — fine-tune system models with custom data (image classifiers, word taggers, 6-DoF object tracking for Vision Pro).

### Core ML & Model Deployment
- Core ML format: model description (inputs/outputs/architecture) + learned parameters.
- Pre-converted models available at developer.apple.com and the Apple Hugging Face space.
- **Core ML Tools** — convert PyTorch/other formats; automatic optimizations (op fusion, dead-code elimination) plus opt-in compression (quantization, pruning, palettization).
- Xcode model inspector: latency, load time, per-op device placement.
- **[NEW in Xcode 26]** Full model architecture visualization — interactive graph view of all ops.
- Runtime: Core ML automatically distributes work across CPU, GPU, Neural Engine.
- Lower-level access: `MPSGraph`, `Metal`, `Accelerate/BNNS`.
- **BNNSGraph** — **[NEW]** `BNNSGraphBuilder` API lets developers create custom graphs of operations for real-time CPU-based pre/post-processing or small ML models.

### MLX (Exploration & Research)
- Open-source array framework for numerical computing and ML on Apple Silicon.
- Unified memory programming model: arrays not tied to a device; CPU and GPU operations can run in parallel on the same buffer without copies.
- Run state-of-the-art LLMs (Mistral, DeepSeek-R1, etc.) with a single command-line call.
- Supports fine-tuning, distributed training, and inference in Python, Swift, C++, C, and community bindings.
- MLX community on Hugging Face: hundreds of frontier models.
- Metal backend for PyTorch and JAX for researchers who want to stay in standard training frameworks.

## APIs & Frameworks

**Foundation Models (NEW)**
- `FoundationModels` framework
- `Session` — stateful model session
- `Session.respond(to:)` — single-turn prompt
- `@Generable` macro — marks types for structured generation
- Tool calling protocol

**Vision (NEW)**
- Document recognition API
- Lens smudge detection API

**Speech (NEW)**
- `SpeechAnalyzer` — new class replacing `SFSpeechRecognizer` for long-form audio

**Core ML**
- `MLModel`, `MLModelConfiguration`
- Core ML Tools (Python)
- `BNNSGraphBuilder` **[NEW]**

**ImagePlayground**
- `ImageCreator` (iOS 18.4+)
- `.imagePlaygroundSheet()` SwiftUI modifier

**Smart Reply (iOS 18.4)**
- `UIMessageConversationContext`
- `UIMailConversationContext`
- `insertInputSuggestion` delegate method

**MLX**
- Open-source; available via pip / Swift Package Manager

## Code Highlights

```swift
// Foundation Models — 3-line prompt
import FoundationModels
let session = LanguageModelSession()
let response = try await session.respond(to: "Summarize this article: \(text)")
```

## Takeaways
- Use the **Foundation Models framework** for on-device summarization, extraction, and structured generation — it's private, offline, free, and deeply integrated with Swift types.
- Adopt the new **SpeechAnalyzer** for long-form speech transcription; `SFSpeechRecognizer` is now the legacy path.
- Check **Vision**'s new document recognition API for document understanding tasks that previously required third-party solutions.
- Use **MLX** for research, fine-tuning, and running frontier-size models on Apple Silicon Macs; use **Core ML** for production deployment in shipping apps.

---
_Source: WWDC25 Session 360 page (abstract, chapter summaries, and full transcript)._
