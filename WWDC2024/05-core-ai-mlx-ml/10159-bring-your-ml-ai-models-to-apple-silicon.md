# Bring Your Machine Learning and AI Models to Apple Silicon
**WWDC24 · Session 10159** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10159/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia (Apple Silicon required for best performance; Intel Macs supported with CPU fallback)

## Overview
This session covers how to optimize and deploy machine learning and AI models on Apple Silicon using Core ML Tools, focusing on model compression techniques that dramatically reduce model size while preserving accuracy. The session is essential for developers working with large models—especially transformer-based LLMs and diffusion models—who need to fit them into the memory constraints of mobile or embedded deployments.

Three major compression techniques are covered: palettization (clustering weights into a lookup table), linear quantization (reducing float32 weights to float16/int8/int4), and pruning (zeroing low-magnitude weights to create sparsity). The session also introduces the new **Stateful Core ML models** API for KV-cache and other stateful inference patterns, and the **multi-function Core ML models** feature for sharing weights across multiple variants.

## Key Topics
- **Palettization** — cluster weights into N-bit lookup tables (2/4/6/8 bits); lossless for lookup, lossy for the quantization step; `coremltools.optimize.coreml.palettize_weights`
- **Linear quantization** — per-channel or per-block int8/int4/float16 quantization; `coremltools.optimize.coreml.linear_quantize_weights`
- **Pruning** — magnitude-based zeroing of weights; unstructured (any weight) or structured (whole channels/rows); `coremltools.optimize.coreml.prune_weights`
- **Calibration data** — post-training optimization uses a small calibration dataset to minimize accuracy loss; `coremltools.optimize.coreml.OpPalettizerConfig(mode="kmeans", nbits=4)`
- **[NEW] Stateful Core ML models** — maintain state (e.g., KV-cache) across inference calls; `register_buffer` in PyTorch; `ct.StateType` in coremltools; `MLState` in Core ML Swift API
- **[NEW] Multi-function Core ML models** — single `.mlpackage` with multiple function variants (e.g., prefill + decode); share weights; invoked via `MLModelAsset` and function name selection
- **Combined compression** — stack palettization + quantization + pruning for maximum compression; validate accuracy after each step

## APIs & Frameworks
### Core ML (Swift / Objective-C)
- `MLModel` — load and run Core ML models
- **[NEW] `MLState`** — represents model state for stateful inference; `model.makeState()` creates an instance; passed to `prediction(from:using:)` overload
- **[NEW] `MLModelAsset`** — reference to a compiled model on disk; supports multi-function models; `MLModelAsset(url:)` then `MLModel(asset:configuration:)`
- **[NEW] `MLPredictionOptions.functionName`** — select which function to run in a multi-function model (e.g., `"prefill"`, `"decode"`)
- `MLModelConfiguration` — set `computeUnits` (`.all`, `.cpuAndNeuralEngine`, `.cpuAndGPU`, `.cpuOnly`)
- `MLMultiArray` — data container for model inputs/outputs

### Core ML Tools (Python)
- `coremltools` — Python package for model conversion and optimization
- `ct.convert(model, ...)` — convert PyTorch/TensorFlow models to `.mlpackage`
- **[NEW] `ct.StateType`** — declare a state buffer in the Core ML model spec; paired with `register_buffer` in the PyTorch source model
- `coremltools.optimize.coreml` module:
  - `palettize_weights(model, config)` — cluster weights into lookup tables
  - `linear_quantize_weights(model, config)` — quantize to int8/int4/float16
  - `prune_weights(model, config)` — zero out low-magnitude weights
  - `OpPalettizerConfig(mode, nbits, weight_threshold)` — per-op palettization config
  - `OpLinearQuantizerConfig(mode, dtype, granularity)` — per-op quantization config
  - `OptimizationConfig(global_config, op_type_configs, op_name_configs)` — compose per-op overrides
- `coremltools.optimize.torch` — PyTorch-side quantization-aware training (QAT) and pruning hooks

## Code Highlights
```python
import coremltools as ct
from coremltools.optimize.coreml import (
    palettize_weights, OpPalettizerConfig, OptimizationConfig
)

# Load a converted Core ML model
mlmodel = ct.models.MLModel("MyTransformer.mlpackage")

# Apply 4-bit palettization with K-means clustering
op_config = OpPalettizerConfig(mode="kmeans", nbits=4)
config = OptimizationConfig(global_config=op_config)
compressed_model = palettize_weights(mlmodel, config=config)
compressed_model.save("MyTransformer_4bit.mlpackage")

# Stateful model: register_buffer in PyTorch source
class KVCacheTransformer(nn.Module):
    def __init__(self):
        super().__init__()
        self.register_buffer("kv_cache", torch.zeros(1, 32, 512, 64))

    def forward(self, x, kv_state):
        # kv_state is the Core ML MLState
        ...

# Convert with state
traced = torch.jit.trace(model, example_inputs)
mlmodel = ct.convert(
    traced,
    inputs=[ct.TensorType(shape=(1, 128, 768))],
    states=[ct.StateType(
        wrapped_type=ct.TensorType(shape=(1, 32, 512, 64)),
        name="kv_cache"
    )]
)
```

```swift
// Swift: stateful inference with MLState
let asset = try MLModelAsset(url: modelURL)
let model = try MLModel(asset: asset, configuration: config)

var state = model.makeState()  // NEW

let input = try MLDictionaryFeatureProvider(dictionary: ["tokens": inputArray])
let options = MLPredictionOptions()
options.functionName = "decode"  // multi-function selection

let output = try model.prediction(from: input, using: &state, options: options)
```

## Takeaways
- 4-bit palettization via K-means typically achieves 8x model size reduction with <1% accuracy degradation on most transformer models—it's the first compression technique to try
- Stateful Core ML models eliminate the need to re-pass KV-cache tensors as inputs/outputs on every token; this is essential for efficient LLM autoregressive decoding on-device
- Multi-function models let you pack prefill and decode functions (with shared weights) into a single `.mlpackage`, reducing disk footprint compared to separate model files
- Always validate with a representative calibration dataset after compression; use `coremltools.optimize` metrics to compare pre/post accuracy before shipping

---
_Source: WWDC24 Session 10159 page (abstract, chapter summaries, code samples, and resource links)._
