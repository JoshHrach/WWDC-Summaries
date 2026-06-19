# Deploy Machine Learning and AI Models On-Device with Core ML
**WWDC24 · Session 10161** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10161/)

_Platforms:_ iOS 18, iPadOS 18, macOS 15, tvOS 18, watchOS 11, visionOS 2

## Overview
Core ML in iOS 18 gains a substantially expanded compute backend and a new model deployment workflow oriented around modern large-scale AI models. Two major themes dominate the session: hardware acceleration improvements (including new Neural Engine scheduling and GPU compute graph execution paths) and the new `MLModelCollection` and stateful model APIs that make deploying and updating on-device models practical for production apps.

The session also introduces `MLComputePlan` — a new API for introspecting which layers of a model run on which compute unit before actually loading the model — enabling developers to make informed deployment decisions without trial-and-error profiling.

## Key Topics
- **`MLComputePlan`** — inspect the planned compute unit assignment for every layer in a Core ML model without running it; essential for verifying Neural Engine utilization of quantized or multi-modal models.
- **Stateful models** — `MLState` wraps persistent state across multiple predictions (e.g., KV-cache for language models); `MLModel.prediction(from:using:)` threads state through successive calls without copying tensors.
- **`MLModelCollection`** — manage a set of related models with a shared identifier; load, swap, and update models in the collection at runtime from disk or over the air via CloudKit or a custom CDN.
- **Activation memory reduction** — new automatic graph optimizations in the Core ML compiler reduce peak activation memory; large models that previously required splitting across multiple `.mlpackage` files may now fit in a single model.
- **Quantization improvements** — 4-bit weight compression (`MLModelConfiguration.ComputeUnits.cpuAndNeuralEngine` combined with quantized weights) now achieves state-of-the-art throughput on the Neural Engine.

## APIs & Frameworks

**Core ML**
- `MLModel` — primary inference type; unchanged API surface
- `MLModelConfiguration` — configure compute units, function name, etc.
  - `MLModelConfiguration.computeUnits` — `.all`, `.cpuAndNeuralEngine`, `.cpuAndGPU`, `.cpuOnly`
- **[NEW]** `MLComputePlan` — pre-flight compute unit assignment analysis
  - `MLComputePlan(contentsOf:configuration:)` — async initializer from a `.mlpackage` URL
  - `MLComputePlan.computeDeviceUsage(for:)` — returns `MLComputeDeviceUsage` for a given layer
  - `MLComputeDeviceUsage` — `.preferred`, `.supported` compute devices for a layer
  - `MLComputePlan.deviceUsage` — dictionary mapping layer name to `MLComputeDeviceUsage`
- **[NEW]** `MLState` — opaque container for stateful model persistent values (e.g., KV-cache)
  - `MLModel.makeState() -> MLState` — create a fresh state instance
  - `MLModel.prediction(from:using:) async throws -> MLFeatureProvider` — stateful prediction; pass an `MLState` to thread state across calls
- **[NEW]** `MLModelCollection` — manage a named group of related `.mlpackage` files
  - `MLModelCollection(identifier:)` — initialize by identifier
  - `MLModelCollection.entries` — dictionary of `MLModelCollectionEntry` values (name → model URL)
  - `MLModelCollectionEntry.model` — load the entry's `MLModel`
- `MLFeatureProvider` — unchanged; carries input/output feature dictionaries
- `MLMultiArray` — unchanged; primary tensor type for numerical inputs/outputs
- `MLShapedArray<Scalar>` — Swift-native tensor type; unchanged

**Create ML / coremltools (Python)**
- `coremltools.optimize.coreml.linear_quantize_weights` — 4-bit weight quantization for Neural Engine deployment
- `MLProgram` model format — required for stateful models; authored via coremltools 7+

## Code Highlights
Pre-flight compute plan inspection:

```swift
let plan = try await MLComputePlan(contentsOf: modelURL, configuration: config)
for (layerName, usage) in plan.deviceUsage {
    print("\(layerName): preferred=\(usage.preferred)")
}
```

Stateful inference with KV-cache:

```swift
let model = try MLModel(contentsOf: modelURL, configuration: config)
let state = model.makeState()
for token in inputTokens {
    let input = try MLDictionaryFeatureProvider(dictionary: ["input_ids": token])
    let output = try await model.prediction(from: input, using: state)
    // state carries KV-cache forward automatically
}
```

## Takeaways
- Use `MLComputePlan` before shipping to confirm that quantized model layers land on the Neural Engine — if critical layers fall back to CPU, adjust quantization or split the model.
- `MLState` is the correct pattern for any generative model with iterative decoding (language models, diffusion samplers); it avoids copying the entire KV-cache on every token prediction.
- `MLModelCollection` enables server-side model updates without a new App Store release — combine with CloudKit or a signed CDN to push improved model checkpoints to users.
- 4-bit quantized models on iOS 18's Neural Engine offer a strong quality/performance/memory trade-off for large models; benchmark with `MLComputePlan` + Instruments to confirm Neural Engine utilization.

---
_Source: WWDC24 Session 10161 page (abstract, chapter summaries, code samples, and resource links)._
