# Optimize machine learning for Metal apps
**WWDC23 · Session 10050** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10050/)

_Platforms:_ macOS Sonoma 14, iOS 17

## Overview
This session covers the full breadth of Metal-accelerated machine learning on Apple Silicon in 2023, spanning training frameworks (PyTorch, TensorFlow, and the newly supported JAX), and inference improvements in MPSGraph. PyTorch's MPS backend reaches Beta status with PyTorch 2.0, delivering up to 5x speedup over the previous release. TensorFlow reaches a stable 1.0 Metal plugin release. JAX debuts GPU acceleration through Metal and benchmarks around 10x faster than CPU across common networks on M2 Max.

On the inference side, MPSGraph gains a new binary serialization format (`MPSGraphPackage`) to eliminate application launch-time compilation overhead, a command-line conversion tool (`MPSGraphTool`) to import CoreML and ONNX models, and new 8-bit integer quantization operators. Additional operator coverage includes complex number types, Fast Fourier Transforms, 3D convolutions, grid sampling, Sort/ArgSort, and cumulative operations.

The session provides practical guidance on writing custom PyTorch Metal kernels (to replace CPU fallbacks), enabling Automatic Mixed Precision with float16/bfloat16, using the new MPSGraph quantization APIs, and profiling ML workloads with Metal System Trace in Instruments.

## Key Topics

### PyTorch 2.0 MPS Backend (Beta)
Support for the top 60 most-used Torch operators including grid sampler, triangular solve, and topk. Expanded model coverage includes WhisperAI, YOLOv5, and Stable Diffusion. Up to 5x faster than the previous release on macOS Sonoma.

### PyTorch Profiling with Metal System Trace
New OS signpost-based profiling in nightly builds. `torch.mps.profiler.start()` / `.stop()` enable capture in Instruments Metal System Trace, showing per-op execution times, CPU↔GPU copies, and CPU fallback events.

### Custom PyTorch Metal Kernels
When an operator falls back to CPU (e.g., Softshrink), developers can write a custom Metal/Objective-C kernel, bind it with PYBIND11, compile it as a C++ extension using `torch.utils.cpp_extension.load()`, and import it directly into training scripts. Key MPS backend APIs: `get_command_buffer`, `get_dispatch_queue`, `synchronize`, `commit`.

### Automatic Mixed Precision
Both float16 and bfloat16 are now supported with PyTorch autocast on MPS. bfloat16 is a new data type added to MPSGraph in macOS Sonoma (1 sign bit, 8 exponent bits, 7 mantissa bits), better suited to deep learning than IEEE float16.

### TensorFlow Metal Plugin 1.0 (Stable)
Grappler remapping optimizer pass automatically fuses convolutions, matrix multiplications, optimizer operations, and RNN cells into optimized kernels. Mixed precision support for float16 and bfloat16 via global policy. Simplified installation via standard pip workflow; also available in nightly TensorFlow builds.

### JAX Metal Acceleration (New)
JAX debuts GPU acceleration on Apple Silicon through the Metal backend — ~10x faster than CPU on M2 Max. Supports `grad` (automatic differentiation), `vmap` (vectorization), and `jit` (just-in-time compilation).

### MPSGraph Serialization: MPSGraphPackage
New binary format for pre-compiled `MPSGraphExecutable` objects. Eliminates first-launch compilation overhead for complex graphs. `MPSGraphExecutable.serialize(to:descriptor:)` writes the package; constructor `MPSGraphExecutable(mpsgraphPackageAtURL:descriptor:)` loads it.

### MPSGraphTool (New Command-Line Tool)
Converts CoreML (`.mlpackage`) and ONNX (`.onnx`) models to `MPSGraphPackage` format, enabling integration without manually encoding the inference graph.

### 8-bit Integer Quantization
`MPSGraph.quantizeTensor(_:scale:zeroPoint:dataType:name:)` and `MPSGraph.dequantizeTensor(_:scale:zeroPoint:dataType:name:)` for symmetric and asymmetric int8 quantization. The graph automatically fuses dequantize → convolution → quantize into a single kernel, saving memory bandwidth.

### New MPSGraph Operators
Complex number support (float32 and float16), FFT (complex-to-complex, complex-to-real, real-to-complex, up to 4D), 3D convolutions, grid sampling, Sort/ArgSort, and cumulative sums/products/minima/maxima.

## APIs & Frameworks

- **Metal Performance Shaders (MPS)** — GPU primitives for image processing, linear algebra, ML
- **MetalPerformanceShadersGraph (MPSGraph)** — general-purpose compute graph
  - `MPSGraphExecutable` — compiled executable graph
    - `serialize(to:descriptor:)` — serialize to MPSGraphPackage **[NEW]**
    - `init(mpsgraphPackageAtURL:descriptor:)` — load from MPSGraphPackage **[NEW]**
  - `MPSGraphSerializationDescriptor` — configuration for serialization **[NEW]**
  - `MPSGraphCompilationDescriptor` — configuration for compilation/loading
  - `MPSGraph.quantizeTensor(_:scale:zeroPoint:dataType:name:)` — 8-bit quantization **[NEW]**
  - `MPSGraph.dequantizeTensor(_:scale:zeroPoint:dataType:name:)` — 8-bit dequantization **[NEW]**
  - Complex number data types (float32 and float16 complex) **[NEW]**
  - FFT operators: complex-to-complex, complex-to-real, real-to-complex (up to 4D) **[NEW]**
  - 3D convolution operators **[NEW]**
  - Grid sampling operators **[NEW]**
  - `MPSGraph.sort(_:axis:name:)` / `MPSGraph.argSort(_:axis:name:)` **[NEW]**
  - Cumulative sum, product, minimum, maximum operators **[NEW]**
  - `bfloat16` data type support **[NEW]** (macOS Sonoma)
- **MPSGraphTool** — command-line tool to convert CoreML/ONNX → MPSGraphPackage **[NEW]**
- **PyTorch MPS Backend** (torch.backends.mps)
  - `torch.mps.profiler.start()` / `torch.mps.profiler.stop()` — OS signpost profiling **[NEW]**
  - `torch.autocast("mps", dtype=torch.float16|bfloat16)` — Automatic Mixed Precision **[NEW]**
  - `torch.utils.cpp_extension.load()` — compile custom Metal/C++ extensions
  - MPS Backend APIs: `get_command_buffer`, `get_dispatch_queue`, `synchronize`, `commit`
  - PYBIND11 for Python↔Objective-C bindings
- **TensorFlow Metal Plugin** 1.0 (stable)
  - Grappler remapping optimizer (automatic kernel fusion) **[NEW]**
  - `tf.keras.mixed_precision.set_global_policy("mixed_float16"|"mixed_bfloat16")` **[NEW]**
- **JAX Metal Backend** (GPU acceleration) **[NEW]**
  - `jax.grad` — automatic differentiation
  - `jax.vmap` — vectorization
  - `jax.jit` — just-in-time compilation
- **Instruments / Metal System Trace** — profiling ML workloads with OS signpost lane

## Code Highlights

Enabling PyTorch profiling with Metal System Trace:
```python
import torch.mps.profiler as profiler
profiler.start()
# ... run model ...
profiler.stop()
```

Enabling Automatic Mixed Precision on MPS:
```python
with torch.autocast("mps", dtype=torch.bfloat16):
    output = model(input)
```

Loading a pre-serialized MPSGraphPackage (pseudocode):
```swift
let descriptor = MPSGraphCompilationDescriptor()
let executable = MPSGraphExecutable(mpsgraphPackageAtURL: packageURL, descriptor: descriptor)
```

Converting a CoreML model with MPSGraphTool:
```bash
MPSGraphTool --input-type coreml-package \
             --output path/to/output \
             --model-name MyModel \
             --platform macOS --min-os 14.0 \
             MyModel.mlpackage
```

## Takeaways

- PyTorch 2.0's MPS backend is now Beta-quality (5x faster than the prior release) and supports popular models like YOLOv5, WhisperAI, and Stable Diffusion; use `torch.mps.profiler` to identify CPU-fallback bottlenecks and replace them with custom Metal kernels.
- TensorFlow Metal Plugin 1.0 is stable and now installs via standard pip; JAX gains Metal GPU acceleration (~10x vs CPU on M2 Max) for the first time in 2023.
- Use `MPSGraphPackage` serialization to pre-compile inference graphs and eliminate app launch overhead; `MPSGraphTool` converts existing CoreML and ONNX models without manual graph encoding.
- MPSGraph's new int8 quantization operators automatically fuse into single kernels, reducing memory bandwidth and improving inference efficiency.

---
_Source: WWDC23 Session 10050 page (abstract, chapter summaries, code samples, and resource links)._
