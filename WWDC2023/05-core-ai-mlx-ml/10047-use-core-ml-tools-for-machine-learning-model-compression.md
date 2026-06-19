# Use Core ML Tools for Machine Learning Model Compression
**WWDC23 · Session 10047** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10047/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, tvOS 17, watchOS 10

## Overview
This session covers three weight compression techniques — pruning, quantization, and palettization — and shows how to apply them using the redesigned `coremltools.optimize` API in Core ML Tools 7. The goal is to reduce model size (and in iOS 17, also improve inference latency) while maintaining acceptable accuracy.

Two distinct compression workflows are presented: post-training compression, which operates on an already-trained and converted Core ML model with no retraining required; and training-time compression, which fine-tunes a PyTorch model with differentiable compression layers inserted, recovering accuracy that post-training compression loses at high compression rates. The session uses an SSD object detection model as a running demo, showing how 2-bit post-training palettization breaks the model while 2-bit training-time palettization restores it.

A key iOS 17 runtime improvement enables just-in-time weight decompression on the Neural Engine, meaning compressed models can run 5–30% faster than their Float16 counterparts for memory-bound workloads, without any extra developer work beyond using the `coremltools.optimize` APIs.

## Key Topics

### Compression Techniques

**Pruning**
- Sets the smallest-magnitude weights to zero; only non-zero values and their indices are stored.
- Model size decreases linearly with sparsity; a 50% sparse ResNet50 drops from 50 MB to ~28 MB.
- Controlled by target sparsity (0–1).

**Quantization**
- Stores weights as INT8 (8-bit); reduces model size approximately by half vs Float16.
- Scale and optional bias allow de-quantization back to original range.

**Palettization (Weight Clustering)**
- Groups weights into 2^n clusters; stores a look-up table of centroids and an index table.
- n-bit palettization yields up to 16x/8x/4x compression for 1/2/4 bits respectively vs Float16.
- 2-bit → 8x compression; 4-bit → 4x; 6-bit → ~2.7x.

### Post-Training Compression Workflow
- Input: a trained, converted Core ML model (`.mlpackage`).
- Apply `prune_weights`, `palettize_weights`, or `linear_quantize_weights` from `coremltools.optimize.coreml`.
- Create an `OptimizationConfig` with an `OpPalettizerConfig` / `OpPrunerConfig` / `OpLinearQuantizerConfig`.
- One-step API call; no training data needed.
- Works on any starting point: pre-trained PyTorch, ONNX, or existing Core ML model.

### Training-Time Compression Workflow
- Input: PyTorch model with pre-trained weights + fine-tuning dataset.
- Use `coremltools.optimize.torch` APIs: `MagnitudePruner`, `DKMPalettizer`, etc.
- Steps: create config → create compressor → call `prepare()` to insert compression layers → fine-tune → call `finalize()` to fold compressed weights → convert to Core ML.
- DKM (Differentiable K-Means) algorithm learns weight clusters via attention-based differentiable k-means.
- Fine-tuning can be done locally on a Mac using the MPS PyTorch backend.
- Pass `PassPipeline.DEFAULT_PALETTIZATION` to `coremltools.convert` to preserve palettized representation.

### iOS 17 Runtime Improvements
- iOS 16: compressed weights were decompressed ahead-of-time to Float16 in memory before inference; no latency benefit.
- iOS 17: just-in-time decompression on the Neural Engine for memory-bound models; smaller weights loaded from memory, decompressed per inference call.
- 4-bit palettized models: 5–30% inference speedup on iPhone 14 Pro Max.
- Sparse models: up to 75% faster than Float16 variants.
- 8-bit activation quantized models can now be executed (previously only weight-only quantized models were supported).
- Same improvements in macOS Sonoma, tvOS 17, watchOS 10.

## APIs & Frameworks

- `coremltools.optimize` **[NEW]** — unified module consolidating all compression APIs in Core ML Tools 7
- `coremltools.optimize.coreml` **[NEW]** — post-training compression APIs operating on Core ML models
- `coremltools.optimize.coreml.prune_weights(model, config)` **[NEW]** — applies magnitude-based weight pruning
- `coremltools.optimize.coreml.palettize_weights(model, config)` **[NEW]** — applies k-means weight palettization
- `coremltools.optimize.coreml.linear_quantize_weights(model, config)` **[NEW]** — applies INT8 weight quantization
- `coremltools.optimize.coreml.OptimizationConfig` **[NEW]** — container for global and per-op compression configs
- `coremltools.optimize.coreml.OpPalettizerConfig` **[NEW]** — per-op palettization config (mode, nbits)
- `coremltools.optimize.coreml.OpPrunerConfig` **[NEW]** — per-op pruning config (target sparsity)
- `coremltools.optimize.coreml.OpLinearQuantizerConfig` **[NEW]** — per-op quantization config
- `coremltools.optimize.torch` **[NEW]** — training-time compression APIs for PyTorch models
- `coremltools.optimize.torch.MagnitudePruner` **[NEW]** — magnitude-based training-time pruner
- `coremltools.optimize.torch.MagnitudePrunerConfig` **[NEW]** — config for training-time magnitude pruning
- `coremltools.optimize.torch.DKMPalettizer` **[NEW]** — differentiable k-means training-time palettizer
- `coremltools.optimize.torch.DKMPalettizerConfig` **[NEW]** — config for DKM palettization (n_bits, global_config)
- `MagnitudePruner.prepare(model)` **[NEW]** — inserts pruning layers into PyTorch model
- `MagnitudePruner.step()` **[NEW]** — updates pruner state during training step
- `MagnitudePruner.finalize(model)` **[NEW]** — folds pruning masks into weights
- `DKMPalettizer.prepare(model)` **[NEW]** — inserts palettization layers into PyTorch model
- `DKMPalettizer.finalize(model)` **[NEW]** — restores palettized weights
- `coremltools.convert(model, pass_pipeline=PassPipeline.DEFAULT_PALETTIZATION)` **[NEW param]** — converts with palettized representation
- `PassPipeline.DEFAULT_PALETTIZATION` **[NEW]** — conversion pass pipeline preserving weight clusters
- `coremltools.convert` — existing Core ML model conversion API
- `Core ML` framework — on-device inference runtime with iOS 17 just-in-time decompression
- `compression_utils` submodule (Core ML Tools 6) — predecessor, now superseded by `optimize.coreml`

## Code Highlights

```python
# Post-training 6-bit palettization
from coremltools.optimize.coreml import OpPalettizerConfig, OptimizationConfig, palettize_weights

op_config = OpPalettizerConfig(mode="kmeans", nbits=6)
config = OptimizationConfig(global_config=op_config)
palettized_model = palettize_weights(coreml_model, config)

# Training-time 2-bit palettization with DKM
from coremltools.optimize.torch import DKMPalettizer, DKMPalettizerConfig

config = DKMPalettizerConfig(global_config={"n_bits": 2})
palettizer = DKMPalettizer(model, config)
model = palettizer.prepare()
# ... fine-tune for several epochs ...
model = palettizer.finalize()

# Convert with palettized representation
import coremltools as ct
traced = torch.jit.trace(model, example_input)
coreml_model = ct.convert(traced, pass_pipeline=ct.PassPipeline.DEFAULT_PALETTIZATION)

# Training-time pruning
from coremltools.optimize.torch import MagnitudePruner, MagnitudePrunerConfig

config = MagnitudePrunerConfig(target_sparsity=0.75)
pruner = MagnitudePruner(model, config)
model = pruner.prepare()
for epoch in range(num_epochs):
    train_epoch(model, data)
    pruner.step()
model = pruner.finalize()
```

## Takeaways
- The new `coremltools.optimize` module unifies post-training (`optimize.coreml`) and training-time (`optimize.torch`) compression under a single consistent API.
- Post-training palettization is fast and requires no data, but degrades sharply at 2–4 bits; training-time palettization (DKM) recovers accuracy at the same compression levels with minimal fine-tuning epochs.
- iOS 17's just-in-time decompression on the Neural Engine makes compressed models faster at runtime — not just smaller — for memory-bound models, with no additional developer work.
- Use Core ML performance reports in Xcode to profile compressed variants on device before committing to a configuration.

---
_Source: WWDC23 Session 10047 page (abstract, chapter summaries, code samples, and resource links)._
