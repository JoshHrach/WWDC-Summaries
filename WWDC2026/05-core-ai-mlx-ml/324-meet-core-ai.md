# Meet Core AI
**WWDC26 · Session 324** · [Watch](https://developer.apple.com/videos/play/wwdc2026/324/)

_Platforms:_ iOS, iPadOS, macOS, tvOS, visionOS, watchOS

## Overview
Core AI is Apple's new on-device AI inference framework — the same engine that powers Apple Intelligence — now available to all developers. It covers the full model deployment lifecycle: Python tooling for converting and optimizing models, a modern Swift API for running inference, deep Xcode integration including a built-in model viewer and instruments template, and a Core AI Models repository with ready-to-run examples.

The framework is designed to leverage all of Apple Silicon simultaneously — CPU, GPU, and the Apple Neural Engine — with automatic hardware routing handled by the runtime. A dedicated specialization step compiles a model for the exact target device, delivering maximum performance with no extra developer effort.

This introductory session walks through the complete workflow end-to-end using a Snake game AI as a running example: convert a PyTorch transformer, integrate it in a Swift app, profile with Instruments, add a key-value cache for performance, and manage model specialization for distribution.

## Key Topics

### What is Core AI
Core AI is Apple's inference framework for on-device AI, designed for the full model deployment lifecycle. It utilizes all Apple Silicon compute resources (CPU, GPU, ANE) and ships with Python tooling, a Swift API, Xcode integration, and a Core AI Models GitHub repository.

### Model Conversion
PyTorch models are converted using the `coreai-torch` Python package. The workflow uses `torch.export.export` with dynamic shape annotations, runs `coreai_torch.TorchConverter`, and saves a `.aimodel` asset. Numerical correctness is verified by comparing PyTorch and Core AI outputs on sample inputs.

### App Integration
In Swift, load a model with `AIModel(contentsOf:)`, call `model.loadFunction(named:)` to get an `InferenceFunction`, construct `NDArray` inputs, and call `mainFunction.run(inputs:)`. The Xcode model viewer shows model size, platform targets, function signatures, and tensor shapes.

### Profiling with Instruments
The new Core AI Instruments template visualizes per-inference latency over time. The demo identified a quadratic slowdown in a transformer caused by reprocessing all prior tokens on every inference step.

### Optimizing Performance with KV Cache
The solution is adding a key-value cache as model state. The PyTorch module uses `register_buffer` for the cache, and the conversion call adds `state_names=["keyCache", "valueCache"]`. In Swift, `NDArray` cache buffers are stored on the player struct and passed as `InferenceFunction.MutableViews` via `states:` on each `run()` call.

### Specialization
Core AI specializes (compiles) a model for the specific target device during first run. `AIModelCache.default` lets apps check if a model is already compiled before loading. `AIModel.specialize(contentsOf:)` triggers specialization explicitly. `SpecializationOptions` control behavior. Ahead-of-time (AOT) compilation via `xcrun coreai-build` can pre-compile models on a development machine, eliminating on-device wait time.

### Additional Features
The Python authoring environment supports rich model construction beyond conversion. The Core AI Debugger provides numeric debugging for converted models. The Core AI debug gauge in Xcode provides streaming activity monitoring.

## APIs & Frameworks

**CoreAI (Swift)** **[NEW]**
- `AIModel` — loads a `.aimodel` file; `init(contentsOf:)`, `loadFunction(named:)`
- `InferenceFunction` — represents a named function in a model; `run(inputs:)`, `run(inputs:states:)`
- `InferenceFunction.MutableViews` — container for stateful cache buffers; `insert(_:for:)`
- `NDArray` — n-dimensional typed tensor; `init(shape:scalarType:)`, `.view()`, `.mutableView()`
- `NDArray.View<Scalar>` — read-only typed view over array data
- `NDArray.MutableView<Scalar>` — mutable typed view
- `AIModelCache` — `.default` singleton; `model(for:options:)` to check compiled cache
- `AIModel.specialize(contentsOf:)` — explicitly trigger on-device specialization **[NEW]**
- `SpecializationOptions` — options controlling specialization behavior

**coreai-torch (Python)** **[NEW]**
- `coreai_torch.TorchConverter` — converts exported PyTorch programs to Core AI format
- `TorchConverter.add_exported_program(exported, input_names:, output_names:, state_names:)`
- `coreai_torch.get_decomp_table()` — decomposition table for `run_decompositions`
- `coreai.runtime.AIModel` — Python runtime for loading and verifying converted models
- `coreai.runtime.NDArray` — Python-side n-dimensional array type

**Xcode Integration** **[NEW]**
- `.aimodel` model viewer — inspect size, platforms, function signatures, tensor shapes
- Core AI Instruments template — latency profiling per inference call
- Core AI debug gauge — streaming activity monitoring
- `xcrun coreai-build compile <model> --platform <platform>` — AOT compilation CLI

**Related Resources**
- [Core AI documentation](https://developer.apple.com/documentation/CoreAI)
- [Core AI PyTorch Extensions](https://apple.github.io/coreai-torch)
- [Core AI Python](https://apple.github.io/coreai-torch/main/coreai-core)
- [Core AI Optimization](https://apple.github.io/coreai-optimization)
- [Compiling Core AI models ahead of time](https://developer.apple.com/documentation/CoreAI/compiling-core-ai-models-ahead-of-time)
- [Managing model specialization and caching](https://developer.apple.com/documentation/CoreAI/managing-model-specialization-and-caching)

## Code Highlights

Convert a PyTorch model:
```python
exported = torch.export.export(pt_model, args=(example,), dynamic_shapes={...})
exported = exported.run_decompositions(coreai_torch.get_decomp_table())
ai_program = coreai_torch.TorchConverter().add_exported_program(
    exported, input_names=["features"], output_names=["logits"],
).to_coreai()
ai_program.save_asset("SnakeTransformer.aimodel")
```

Run inference in Swift:
```swift
let model = try await AIModel(contentsOf: modelURL)
let mainFunction = try model.loadFunction(named: "main")!
var outputs = try await mainFunction.run(inputs: ["features": inputNDArray])
```

Pass stateful KV cache:
```swift
var stateViews = InferenceFunction.MutableViews()
stateViews.insert(&keyCache, for: "keyCache")
stateViews.insert(&valueCache, for: "valueCache")
var outputs = try await nextActionFunction.run(inputs: [...], states: stateViews)
```

Check specialization cache before loading:
```swift
guard let model = try cache.model(for: modelURL, options: .default) else {
    informUser("Preparing AI features…")
}
```

## Takeaways
- Core AI is now the recommended path for on-device custom model deployment on all Apple platforms; prefer it over lower-level alternatives for new projects.
- Use the Core AI Instruments template early to catch latency regressions — transformer KV caches are the standard fix for quadratic inference slowdown.
- Plan a deliberate specialization strategy: use `AIModelCache` to check first-run status and `xcrun coreai-build` for AOT compilation to avoid blocking users on first launch.
- Explore the Core AI Models GitHub repository for ready-made export recipes covering popular architectures (LLMs, segmentation models, and more).

---
_Source: WWDC26 Session 324 page (abstract, chapter summaries, code samples, and resource links)._
