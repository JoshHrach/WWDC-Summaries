# Integrate On-Device AI Models into Your App Using Core AI
**WWDC26 · Session 326** · [Watch](https://developer.apple.com/videos/play/wwdc2026/326/)

_Platforms:_ iOS, iPadOS, macOS

## Overview
This session is a practical walkthrough of integrating popular open-source AI models — including Qwen, Mistral, and SAM3 — into an iOS and macOS app using the Core AI framework. All models run entirely on-device: no server, no cost per token, no cloud latency. The demo app is a camera-based language learning tool that segments real-world objects and generates multilingual vocabulary cards using on-device inference.

The session covers the full workflow from model discovery through deployment: using the Core AI Models GitHub repository to find and export ready-made `.aimodel` files, integrating them with just a few lines of Swift code, diagnosing first-run latency with Instruments, and using ahead-of-time (AOT) compilation to eliminate on-device specialization wait time. It also shows how the same Swift code scales from a small iOS model to an 8B-parameter macOS model with no code changes.

## Key Topics

### Model Discovery
Define requirements by content type, language support, and device constraints. SAM3 is selected for text-prompted image segmentation, and Qwen 0.6B (119 languages) is selected for vocabulary card generation. The Core AI Models repository provides a browsable catalog with ready-made export recipes.

### Getting Models with the Core AI Models Repository
The `coreai-models` GitHub repository provides export scripts for popular architectures. Running the provided export script produces optimized `.aimodel` files ready for app integration.

### Integration
`.aimodel` files can be inspected in Xcode's model viewer (size, platforms, function signatures, tensor shapes). Add the `coreai-models` Swift package and select the `CoreAILM` and `CoreAISegmentation` libraries as dependencies.

### Writing the Swift Integration Code
SAM3 image segmentation uses `ImageSegmenter(resourcesAt:)` and `segment(image:prompt:)`. Language model integration uses `CoreAILanguageModel(resourcesAt:)` with the familiar `LanguageModelSession` from the Foundation Models framework. The `@Generable` macro enables structured output typed directly to Swift types.

### Diagnosing Model Specialization Latency
The Core AI Instruments template identifies first-run latency caused by on-device model specialization. This is expected behavior — the model is being compiled for the specific device hardware — but it needs to be handled gracefully in the UI.

### Deployment Strategy
Keep models out of the app bundle to avoid bloating update size. Use a first-run experience to introduce the feature, then trigger on-demand model download via Background Assets when the user opts in.

### Ahead-of-Time (AOT) Compilation
`xcrun coreai-build compile <model> --platform <platform>` runs compilation on the development machine, generating device-architecture-specific assets that eliminate on-device specialization time at first launch.

### Multiplatform
The same Swift code runs on macOS without changes. On macOS, the app steps up to Qwen3 8B for higher-quality reasoning, uses longer context for curriculum generation, and supports batch processing of photo folders — all with the same API surface.

## APIs & Frameworks

**CoreAI (Swift)** **[NEW]**
- `AIModel` — base model container; `init(contentsOf:)`
- `InferenceFunction` — named inference entry point; `run(inputs:)`
- `NDArray` — tensor input/output type
- `AIModelCache` — specialization cache management

**CoreAIImageSegmenter (Swift)** **[NEW]**
- `ImageSegmenter` — **[NEW]** loads a SAM3-compatible segmentation model; `init(resourcesAt:)`
- `ImageSegmenter.segment(image:prompt:)` — text-prompted segmentation returning `SegmentationResponse`
- `SegmentationResponse.segments` — array of segmented regions, each with a `.mask`

**CoreAILanguageModels (Swift)** **[NEW]**
- `CoreAILanguageModel` — **[NEW]** loads a local LLM `.aimodel`; `init(resourcesAt:)`
- Conforms to the `LanguageModel` protocol from Foundation Models

**FoundationModels (Swift)**
- `LanguageModelSession(model:)` — create a session with a custom `CoreAILanguageModel`
- `session.respond(to:)` — generate a text response
- `session.respond(to:generating:)` — generate structured output conforming to `@Generable`
- `@Generable` macro — marks a Swift struct for structured LLM output generation
- `Prompt` / `Attachment` — multimodal prompt construction

**Command-Line Tools** **[NEW]**
- `xcrun coreai-build compile <model.aimodel> --platform <platform>` — AOT compilation

**Xcode Integration** **[NEW]**
- `.aimodel` model viewer (size, platform targets, signatures, shapes)
- Core AI Instruments template — specialization latency diagnosis

**Swift Package: coreai-models**
- `CoreAILM` library — language model integration helpers
- `CoreAISegmentation` library — segmentation model integration helpers

**Related Resources**
- [Core AI documentation](https://developer.apple.com/documentation/CoreAI)
- [Core AI PyTorch Extensions](https://apple.github.io/coreai-torch)
- [Core AI Optimization](https://apple.github.io/coreai-optimization)
- [Compiling Core AI models ahead of time](https://developer.apple.com/documentation/CoreAI/compiling-core-ai-models-ahead-of-time)

## Code Highlights

SAM3 image segmentation:
```swift
import CoreAIImageSegmenter
let segmenter = try await ImageSegmenter(resourcesAt: sam3ModelURL)
let response = try await segmenter.segment(image: inputImage, prompt: "flower")
let mask = response.segments.first?.mask
```

Language model with structured output:
```swift
import FoundationModels
import CoreAILanguageModels

@Generable
struct VocabCard {
    let chineseWord: String
    let englishMeaning: String
    let exampleSentence: String
}

let model = try await CoreAILanguageModel(resourcesAt: modelURL)
let session = LanguageModelSession(model: model)
let response = try await session.respond(to: "Create a vocab card for flower",
                                          generating: VocabCard.self)
let card: VocabCard = response.content
```

AOT compilation:
```bash
xcrun coreai-build compile MyModel.aimodel --platform iOS
```

## Takeaways
- The Core AI Models repository provides one-command export scripts for SAM3, Qwen, Mistral, and other popular models — start there rather than building export pipelines from scratch.
- Keep model files out of the app bundle; use Background Assets for on-demand download so only users who opt into the feature pay the download cost.
- Run `xcrun coreai-build` AOT compilation in CI before shipping to eliminate first-run specialization latency — the difference is dramatic on first launch.
- The `CoreAILanguageModel` + `LanguageModelSession` + `@Generable` combination provides a clean path to structured LLM output with the same API used for the on-device Apple Foundation Model.

---
_Source: WWDC26 Session 326 page (abstract, chapter summaries, code samples, and resource links)._
