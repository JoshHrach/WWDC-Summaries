# Dive into Core AI Model Authoring and Optimization
**WWDC26 · Session 325** · [Watch](https://developer.apple.com/videos/play/wwdc2026/325/)

_Platforms:_ iOS, iPadOS, macOS, tvOS, visionOS, watchOS

## Overview
This session is a deep dive into the complete custom model deployment workflow for Apple Silicon using the Core AI framework. Where the introductory "Meet Core AI" session covers the end-to-end workflow at a high level, this session goes further into advanced authoring techniques — including writing custom Metal kernels that plug directly into Core AI models, platform-aware compression and quantization strategies, and AI-assisted workflows that guide development from concept to optimized on-device execution.

The session covers the new Core AI Debugger, which provides intrinsic analysis for numerically debugging converted models layer by layer. It also explores compression strategies that are aware of the target platform's capabilities, enabling developers to trade model size against quality in a controlled way.

Note: At the time of this summary, the full video content was marked "Available soon" on the Apple Developer website, so this summary is based on the session's abstract and related video context. The resource links and related sessions provide the best supplementary material.

## Key Topics

### Custom Metal Kernel Authoring
Core AI supports plugging custom Metal operations directly into a model graph. This enables developers to implement operations not covered by built-in primitives — for example, novel activation functions, custom attention variants, or domain-specific preprocessing steps — while still benefiting from Core AI's automatic hardware routing and specialization.

### Platform-Aware Compression
The Core AI Python tooling includes compression strategies that account for the specific capabilities of the target platform. Rather than applying a single quantization setting, developers can specify target hardware (e.g., devices with Apple Neural Engine support) and the optimizer selects appropriate bit-widths and formats.

### Core AI Debugger
The Core AI Debugger provides deep intrinsic analysis of converted models. It enables layer-by-layer numeric inspection, helping developers identify where numerical drift or unexpected outputs originate after conversion from PyTorch or other frameworks.

### AI-Assisted Workflows
The session introduces AI-assisted tooling that helps guide developers from an initial model concept through optimization to on-device execution, reducing the iteration time of the authoring and optimization loop.

## APIs & Frameworks

**CoreAI (Swift)** **[NEW]**
- `AIModel` — on-device inference model container
- `InferenceFunction` — named inference entry point within a model
- `NDArray` — n-dimensional array type for inputs/outputs

**coreai-torch (Python)** **[NEW]**
- `coreai_torch.TorchConverter` — converts PyTorch exported programs to Core AI format
- `coreai_torch.get_decomp_table()` — decomposition table helper

**Core AI Optimization (Python)** **[NEW]**
- Platform-aware compression and quantization APIs (see [Core AI Optimization](https://apple.github.io/coreai-optimization))
- Compression strategies targeting specific Apple Silicon configurations

**Core AI Debugger** **[NEW]**
- Layer-by-layer numeric analysis of converted models
- Intrinsic debugging tooling for identifying conversion artifacts

**Metal / Custom Ops**
- Custom Metal kernel integration into Core AI model graphs **[NEW]**
- See related session: "Optimize custom machine learning operations with Metal tensors" (Session 330)

**Related Resources**
- [Core AI documentation](https://developer.apple.com/documentation/CoreAI)
- [Core AI PyTorch Extensions](https://apple.github.io/coreai-torch)
- [Core AI Python](https://apple.github.io/coreai-torch/main/coreai-core)
- [Core AI Optimization](https://apple.github.io/coreai-optimization)

## Code Highlights
Full code samples were not yet published for this session at time of writing. See Session 324 ("Meet Core AI") for foundational conversion and inference patterns, and Session 330 ("Optimize custom machine learning operations with Metal tensors") for custom Metal kernel integration.

```python
# Representative pattern from related sessions
ai_program = coreai_torch.TorchConverter().add_exported_program(
    exported,
    input_names=["input"],
    output_names=["output"],
).to_coreai()
ai_program.save_asset("MyModel.aimodel")
```

## Takeaways
- Use the Core AI Debugger to validate numeric correctness of converted models layer-by-layer before shipping — critical for production deployments.
- Prefer platform-aware compression from the Core AI Optimization library over manual quantization to get best quality-vs-size tradeoffs for each target device.
- Custom Metal kernels can be integrated directly into Core AI model graphs for operations not covered by built-in primitives; combine with Session 330 guidance.
- The full video content was marked "Available soon" at publish time — revisit the session page for chapter summaries and code samples once released.

---
_Source: WWDC26 Session 325 page (abstract and resource links; detailed chapter summaries and code samples not yet published at time of writing)._
