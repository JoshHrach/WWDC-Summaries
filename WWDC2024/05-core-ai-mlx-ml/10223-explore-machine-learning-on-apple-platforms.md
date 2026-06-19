# Explore Machine Learning on Apple Platforms
**WWDC24 · Session 10223** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10223/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia 15, tvOS 18, watchOS 11, visionOS 2

## Overview
This session is the annual ML state-of-the-union for Apple platforms. It maps the full ML landscape — from Apple Intelligence (user-facing foundation models) down through Vision, Translation, Create ML, Core ML, MPS Graph, BNNS Graph (new in 2024), and MLX (open-source). The presentation is navigational: it tells developers which layer to use for each class of task, and summarizes what changed at each layer in 2024.

The session serves as an entry point with pointers to deeper dives for each topic, making it essential for any developer planning to integrate on-device ML in 2024 or 2025.

## Key Topics

### Apple Intelligence
Apple Intelligence is a new system-level AI platform in iOS 18/macOS Sequoia. It powers Writing Tools (grammar, rewrite, summary), Image Playground (generative images), and an expanded Siri (conversational, contextual, in-app actions). Apps integrate via existing App Intents and standard text/image APIs — there is no separate "Apple Intelligence SDK."

### Vision Framework — New Swift API
The Vision framework is redesigned with a Swift-first `async`/`await` API, dropping the `VN` prefix. Every request is now a value type; `perform(on:)` returns typed observations directly. New in 2024: `CalculateImageAestheticsScoresRequest` and holistic body pose (hand tracking integrated into `DetectHumanBodyPoseRequest`). See Session 10163 for the full deep dive.

### Translation API
New first-party Translation framework (introduced WWDC23) matures in 2024 with UI integration helpers and expanded language support. Apps can request translation without leaving context.

### Create ML
Three template additions:
- **Object tracking template** — trains models to track physical objects in video for RealityKit anchoring.
- **Time series classification** — classify time-series sensor streams (accelerometer, gyroscope patterns).
- **Time series forecasting** — predict future sensor values from historical windows.

### Core ML
Four significant additions:
- **`MLTensor`** — a new Swift-native tensor type enabling expressive compute graphs, similar to PyTorch tensors, composable with Core ML inference.
- **States (KV-cache)** — models can hold persistent state across inference calls, enabling efficient LLM token generation with key-value caches.
- **Functions** — a model can contain multiple named compute functions (e.g., prefill + decode) dispatched by the caller.
- **Performance Reports** — Xcode 16 generates per-layer latency breakdowns in the Core ML model viewer.

### MPS Graph
Metal Performance Shaders Graph is the recommended GPU-accelerated compute graph for research and custom operators. 2024 additions include improved interoperability with Core ML and PyTorch export paths.

### BNNS Graph (New)
`BNNS` (Basic Neural Network Subroutines) gains a new **Graph API** — a CPU-only ML execution layer optimized for energy efficiency on Apple Silicon. BNNS Graph is positioned for latency-sensitive CPU inference where GPU scheduling overhead is undesirable. Lower-level than Core ML but higher-level than raw BNNS calls.

### MLX
MLX is Apple's open-source array framework for Apple Silicon, targeting researchers and developers who need PyTorch-like ergonomics on Mac. Available on GitHub at `ml-explore/mlx`. Includes a Swift package (`mlx-swift`) and Python bindings. Not a shipping framework inside apps — a research/training tool.

### CoreNet / OpenELM
CoreNet is Apple's open-source neural network training codebase (previously used for on-device model research). OpenELM is a family of open language models released under CoreNet, ranging from 270M to 3B parameters.

## APIs & Frameworks

**Apple Intelligence** — system-level, no dedicated SDK; integrates via App Intents and standard text/image APIs

**Vision**
- New Swift API (no `VN` prefix), `async`/`await` — see Session 10163 **[NEW]**
- `CalculateImageAestheticsScoresRequest` **[NEW]**
- `DetectHumanBodyPoseRequest.detectsHands` **[NEW]**

**Translation**
- `TranslationSession` (existing, maturing)

**Create ML**
- Object Tracking template **[NEW]**
- Time Series Classification template **[NEW]**
- Time Series Forecasting template **[NEW]**

**Core ML**
- `MLTensor` **[NEW]** — Swift-native tensor type
- Model states (KV-cache) **[NEW]** — persistent state across inference calls
- Model functions **[NEW]** — multiple named compute paths in one `.mlpackage`
- Performance Reports in Xcode **[NEW]** — per-layer latency in model viewer

**MPS Graph**
- Improved Core ML / PyTorch interop **[updated]**

**BNNS Graph** **[NEW]**
- CPU-only ML graph execution API
- Energy-efficient inference for latency-sensitive CPU workloads

**MLX (open source)**
- `ml-explore/mlx` on GitHub
- `mlx-swift` Swift package **[NEW]**

**CoreNet / OpenELM (open source)**
- `apple/corenet` on GitHub
- OpenELM model family (270M–3B parameters) **[NEW]**

## Code Highlights

```swift
// MLTensor arithmetic (Core ML)
let a = MLTensor(shape: [2, 3], scalars: [1, 2, 3, 4, 5, 6], scalarType: Float.self)
let b = MLTensor(ones: [2, 3], scalarType: Float.self)
let result = a + b  // element-wise

// Core ML model with state (KV-cache)
let model = try MLModel(contentsOf: modelURL)
var state = model.makeState()
let input = try MLDictionaryFeatureProvider(dictionary: ["tokens": tokenTensor])
let output = try model.prediction(from: input, using: &state)
```

## Takeaways
- Use **Vision** for image/video analysis (faces, text, barcodes, aesthetics, poses) — the new Swift API is the entry point.
- Use **Core ML** for shipping custom models in apps; `MLTensor` and model states unlock LLM inference patterns.
- Use **BNNS Graph** (new) when you need CPU-only inference with minimal scheduling overhead — useful for real-time audio or sensor processing.
- Use **MLX** for on-device fine-tuning or research on Apple Silicon Mac — it is not intended for shipping inside iOS apps.

---
_Source: WWDC24 Session 10223 page (abstract, chapter summaries, code samples, and resource links)._
