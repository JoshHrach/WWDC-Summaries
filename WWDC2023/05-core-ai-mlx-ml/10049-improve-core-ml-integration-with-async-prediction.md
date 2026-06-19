# Improve Core ML Integration with Async Prediction
**WWDC23 · Session 10049** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10049/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, tvOS 17, watchOS 10, visionOS 1

## Overview
This session covers new APIs and best practices for integrating Core ML models into apps efficiently, with a focus on three main areas: a new API for inspecting compute device availability at runtime, a deep look at the model loading and caching lifecycle, and a new async prediction API that enables concurrent model inference for significantly improved throughput.

iOS 17 ships with an improved Core ML inference engine that speeds up many models automatically — no model recompilation or code changes required. The session uses a live demo of an image colorization app to demonstrate how switching from synchronous, serial predictions to concurrent async predictions using Swift's structured concurrency achieved approximately a 2x throughput improvement.

Key tradeoffs discussed: concurrent predictions increase peak memory usage (more input/output buffers in flight simultaneously), so apps should implement flow control to cap the number of in-flight predictions based on their memory budget and use case.

## Key Topics

### Inference Engine Improvements
- iOS 17 delivers automatic speed improvements for many Core ML models — no model changes required.
- Amount of speedup is model- and hardware-dependent.

### Compute Device Availability
- New `MLComputeDevice` enum with associated values capturing device type and properties.
- `MLModel.availableComputeDevices` property — returns the collection of compute devices Core ML can use.
- Enables runtime decisions about which models or configurations to use based on available hardware (CPU, GPU, Neural Engine).

### Model Lifecycle and Caching
- Two model asset types: source (`.mlmodel`/`.mlpackage`, for authoring) and compiled (`.mlmodelc`, for runtime).
- `MLModel` instantiation triggers either a cached load or an uncached load (device specialization).
- Device specialization: parse → optimization passes → compute segmentation → per-device compilation → cache artifacts.
- Cache is keyed by model path + configuration; persists across app launches and reboots.
- Cache is cleared on low disk space, system updates, or model file modifications.
- Core ML Instrument load event labels: "prepare and cache" (uncached) or "cached."
- Core ML Performance Reports now show both cached and uncached load times.
- Best practices: don't load models on the UI thread at launch; use async loading; keep models loaded for repeated predictions; unload when idle to reduce memory pressure.

### Async Prediction API
- New `MLModel.prediction(from:)` async variant — thread-safe, works with Swift structured concurrency.
- Previous synchronous API is not thread-safe; required actor isolation to serialize predictions.
- Async API enables concurrent predictions: multiple images can be colorized simultaneously.
- `await model.prediction(from: input)` — new async prediction call.
- The API responds to Swift task cancellation; add a manual `Task.checkCancellation()` at call site for best results.
- Concurrent predictions with `MLProgram` and `Pipeline` model types yield the greatest throughput improvements.

### Batch Prediction API (Comparison)
- `MLModel.predictions(fromBatch:)` — runs a batch of inputs through the model with internal concurrency.
- Best when the quantity of work is known upfront; does not support per-item cancellation; less flexible for streaming/dynamic workloads.

### Memory Management with Concurrent Predictions
- Running many predictions concurrently increases peak memory usage due to simultaneous input/output buffers.
- Add flow control: limit the maximum number of in-flight predictions.
- For camera streaming: drop frames rather than deferring them to avoid accumulating stale work.
- Profile with Core ML Instrument + Allocations Instrument combined to identify memory growth from concurrent inference.

### CoreMLTools and Model Development
- New `coremltools.optimize` submodule — unified post-training compression utilities, quantization-aware training extensions for PyTorch.
- New ML Program operations for activation quantization.
- `float16` and `float32` compute precision selectable per-operation at conversion time.

## APIs & Frameworks
- `Core ML` framework — on-device ML inference
- `MLModel` — primary model class
- `MLModel.prediction(from:)` — **[NEW async variant]** async, thread-safe prediction
- `MLModel.availableComputeDevices` **[NEW]** — returns available compute devices at runtime
- `MLComputeDevice` **[NEW]** — enum capturing CPU, GPU, or Neural Engine device and its properties
- `MLModel.predictions(fromBatch:)` — batch prediction API
- `MLModelConfiguration` — model load configuration (compute units, etc.)
- `.mlmodel` / `.mlpackage` — source model file formats
- `.mlmodelc` — compiled model file format
- Core ML Instrument (Xcode) — load event labeling: "prepare and cache" vs. "cached"
- Core ML Performance Reports (Xcode) **[updated]** — now shows uncached load times in addition to cached
- `coremltools` Python package — model conversion and compression
- `coremltools.optimize` **[NEW]** — unified compression submodule with quantization-aware training
- `Accelerate` framework — CPU compute backend for Core ML
- `Metal` framework — GPU compute backend for Core ML
- Neural Engine — dedicated on-device ML accelerator leveraged by Core ML
- Allocations Instrument — memory profiling; combine with Core ML Instrument for concurrent prediction analysis

## Code Highlights

Checking for Neural Engine availability:
```swift
let hasNeuralEngine = MLModel.availableComputeDevices.contains {
    if case .neuralEngine = $0 { return true }
    return false
}
```

Converting from synchronous serial prediction (actor-isolated) to async concurrent prediction:
```swift
// Before: actor-isolated, serialized
actor ColorizingService {
    let model = try! Colorizer()
    func colorize(image: CGImage) throws -> CGImage {
        let input = ColorizerInput(image: image)
        let output = try model.prediction(input: input)
        return output.colorizedImage
    }
}

// After: class-based, concurrent via async prediction
class ColorizingService {
    let model = try! Colorizer()
    func colorize(image: CGImage) async throws -> CGImage {
        try Task.checkCancellation()
        let input = ColorizerInput(image: image)
        let output = try await model.prediction(input: input) // NEW async call
        return output.colorizedImage
    }
}
```

## Takeaways
- iOS 17's updated inference engine provides free speedups for many Core ML models without any code or model changes.
- The new async `prediction(from:)` API is thread-safe and enables concurrent predictions, delivering up to 2x throughput improvement for image processing workloads with `MLProgram` and `Pipeline` model types.
- Understand the caching lifecycle: device specialization only runs once (subsequent loads are fast), but the cache can be evicted — profile both cached and uncached load times.
- Concurrent predictions increase memory usage; implement in-flight concurrency limits and profile with the Allocations Instrument to avoid excessive peak memory consumption.

---
_Source: WWDC23 Session 10049 page (abstract, chapter summaries, code samples, and resource links)._
