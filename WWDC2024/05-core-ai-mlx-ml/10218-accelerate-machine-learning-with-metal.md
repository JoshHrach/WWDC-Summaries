# Accelerate Machine Learning with Metal
**WWDC24 · Session 10218** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10218/)

_Platforms:_ iOS, iPadOS, macOS (Apple Silicon and Intel with discrete GPU)

## Overview
This session covers how to use Metal Performance Shaders Graph (MPSGraph) to accelerate machine learning workloads directly on Apple GPUs. MPSGraph is Apple's framework for defining and running computational graphs—equivalent in role to frameworks like TensorFlow or PyTorch's computation graph backends, but deeply integrated with Metal for maximum performance on Apple hardware.

The session introduces significant new MPSGraph capabilities for WWDC24: transformer model support with fast attention kernels, FFT operations, and the MPSGraph Viewer tool in Xcode 16. These additions make it practical to run large language model inference and other complex ML architectures fully on-device using Metal.

A practical workflow is demonstrated showing how to convert and optimize models for MPSGraph, profile them, and debug execution graphs using the new Xcode tooling. The session pairs naturally with the Core ML Tools compression session (10159) for a full picture of on-device ML optimization.

## Key Topics
- **MPSGraph fundamentals** — graph-based computation, lazy evaluation, GPU scheduling
- **Transformer support** — scaled dot-product attention, KV-cache integration, causal masking
- **FFT operations** — new real and complex Fast Fourier Transform ops for signal processing and diffusion models
- **MPSGraph Viewer (Xcode 16)** — visual graph inspection and profiling in the Instruments-adjacent Xcode debug workflow
- **Performance best practices** — memory layout (NHWC vs NCHW), operator fusion, avoiding unnecessary copies
- **Integration with Core ML** — MPSGraph as a Metal-level back end; complementary to Core ML model pipeline

## APIs & Frameworks
### Metal Performance Shaders Graph (MPSGraph)
- `MPSGraph` — main computation graph object
- `MPSGraphTensor` — symbolic tensor node in the graph
- `MPSGraphOperation` — nodes representing ops (matmul, conv, attention, etc.)
- `MPSGraphExecutable` — compiled, runnable form of a graph; cache for repeated execution
- `MPSGraphTensorData` — wraps `MTLBuffer` / `MTLTexture` for input/output binding
- **[NEW] Scaled dot-product attention** — `scaledDotProductAttentionWithQueryTensor(_:keyTensor:valueTensor:maskTensor:scale:name:)` — fused attention kernel optimized for transformer inference
- **[NEW] FFT ops** — `realFFT`, `complexFFT`, `inverseRealFFT`, `inverseComplexFFT` — enable diffusion model noise schedules and signal processing workloads on GPU
- `MPSGraphCompilationDescriptor` — controls optimization level, memory modes
- `MPSGraphExecutionDescriptor` — controls per-run scheduling, completion handlers

### Xcode 16
- **[NEW] MPSGraph Viewer** — visualize MPSGraph operation graphs, inspect tensor shapes, profile execution time per op; accessed via Instruments or the Xcode debug workflow

### Metal (underlying)
- `MTLCommandBuffer` — used to schedule MPSGraph execution
- `MTLBuffer` — backing store for `MPSGraphTensorData`
- Metal Performance Shaders (MPS) — individual shader kernels that MPSGraph composes

## Code Highlights
```swift
// Build a simple MPSGraph operation
let graph = MPSGraph()
let inputTensor = graph.placeholder(shape: [1, 512, 768], dataType: .float16, name: "input")
let weightTensor = graph.constant(weightData, shape: [768, 768], dataType: .float16)
let outputTensor = graph.matrixMultiplication(primary: inputTensor,
                                               secondary: weightTensor, name: "linear")

// Scaled dot-product attention (new in WWDC24)
let attentionOut = graph.scaledDotProductAttention(
    withQueryTensor: queryTensor,
    keyTensor: keyTensor,
    valueTensor: valueTensor,
    maskTensor: nil,
    scale: Float(1.0 / sqrt(64.0)),
    name: "attention"
)

// Compile and run
let executable = try graph.compile(with: device,
                                    feeds: [:],
                                    targetTensors: [outputTensor],
                                    targetOperations: nil,
                                    compilationDescriptor: nil)
let results = executable.run(with: commandQueue,
                              inputs: [inputData],
                              results: nil,
                              executionDescriptor: nil)
```

## Takeaways
- MPSGraph is the high-performance, Metal-native path for running custom ML models on Apple GPUs; it complements Core ML for cases requiring fine-grained GPU control
- New transformer attention kernels and FFT ops in WWDC24 make it practical to run LLM inference and diffusion models entirely via MPSGraph
- The MPSGraph Viewer in Xcode 16 dramatically reduces the time to diagnose graph structure and performance bottlenecks
- Pair MPSGraph with Core ML Tools palettization/quantization (session 10159) for the best combination of model size reduction and GPU-accelerated inference

---
_Source: WWDC24 Session 10218 page (abstract, chapter summaries, code samples, and resource links)._
