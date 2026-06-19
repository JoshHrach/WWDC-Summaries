# Accelerate Machine Learning with Metal Performance Shaders Graph
**WWDC21 · Session 10152** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10152/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, tvOS 15

## Overview
Metal Performance Shaders Graph (MPSGraph) is a general-purpose compute graph for the GPU that accelerates linear algebra, machine learning, computer vision, and image processing. In this session, Apple engineers showcase how MPSGraph has been adopted by Core ML and TensorFlow for GPU acceleration, resulting in dramatic speedups — including up to 8x faster training on M1 and 2x faster BERT inference in Core ML.

The session introduces a new Metal Plugin for TensorFlow (built on the TensorFlow 2.5 PluggableDevice interface), which allows any TensorFlow model to run on Apple GPU hardware without code changes. It also covers three major new feature areas: new compute primitives, improved compilation APIs, and control flow capabilities.

A practical image composition demo illustrates these features working together: a Laplacian stencil edge filter combined with an iterative linear solver implemented using the new while-loop control flow API.

## Key Topics

### New Metal Plugin for TensorFlow
- Built on TensorFlow 2.5 PluggableDevice interface
- Install via `pip install tensorflow-macos` then `pip install tensorflow-metal`
- Enables GPU-accelerated training on Apple Silicon without model changes
- ResNet50 trains ~4x faster; other benchmarks up to 8x faster on M1

### Core ML Inference Improvements
- BERT transformer network: 2x speedup via MPSGraph
- ResNet50: additional 16% improvement via new buffer backend
- Convolution2D kernel improved for both NHWC (training) and NCHW (inference) layouts

### New Compute Primitives
**Control Dependency** — explicitly orders operations in the graph to prevent unwanted optimization (e.g., keeping running mean/variance updates alive in batch normalization).

**Stencil Operator** — a generalization of sliding window operators (image convolution, Laplacian filters, finite element methods). Supports argmin/argmax reductions, reflection and clampToZero padding, and benefits from kernel stitching for single-dispatch execution.

**Gather / GatherND Operations** — efficient copying of arbitrary slices from non-contiguous memory. GatherND supports N-dimensional indexing and enables embedding lookup and dynamic matrix copy.

### Compilation APIs (MPSGraphExecutable)
- New `MPSGraphExecutable` lets developers control when graph compilation occurs
- `disableTypeInference()` defers shape inference to runtime, saving tens to hundreds of milliseconds per iteration for variable-shaped inputs (NLP sequences, varying image sizes)

### Control Flow APIs
- **if/else**: dispatches different graph branches based on a Boolean predicate (useful for training vs. inference modes in batch normalization)
- **for loop**: fixed number of iterations (common in RNNs)
- **while loop**: iterates until a predicate is false; supports do-while by reordering blocks
- All execute entirely on GPU timeline — no CPU/GPU synchronization overhead

## APIs & Frameworks

### Metal Performance Shaders (MPS)
- `MPSGraph` — core class for defining compute graphs
- `MPSGraphTensor` — tensor node in a graph
- `MPSGraphTensorData` — wraps MTLBuffer/MTLTexture data for execution
- `MPSGraphShapedType` — describes tensor shape and data type for compilation
- `MPSGraphExecutable` **[NEW]** — compiled executable produced by `graph.compile(...)`
- `MPSGraphCompilationDescriptor` **[NEW]** — configuration for compilation; `disableTypeInference()` **[NEW]**
- `graph.controlDependency(with:dependentBlock:name:)` **[NEW]** — control dependency primitive
- `graph.stencil(with:weights:descriptor:name:)` **[NEW]** — stencil/sliding-window operator
- `graph.gatherND(...)` **[NEW]** — N-dimensional gather operation
- `graph.if(_:then:else:name:)` **[NEW]** — conditional control flow
- `graph.for(numberOfIterations:initialIterationArguments:body:)` **[NEW]** — fixed-count loop
- `graph.while(initialInputs:before:after:name:)` **[NEW]** — predicate-based loop
- `graph.compile(with:feeds:targetTensors:targetOperations:compilationDescriptor:)` **[NEW]**
- `executable.run(with:inputs:results:executionDescriptor:)` **[NEW]**
- `graph.exponent(with:name:)` — element-wise exponent
- `graph.addition(_:_:name:)`, `graph.subtraction(_:_:name:)`, `graph.multiplication(_:_:name:)`
- `graph.lessThan(_:_:name:)` — comparison op
- `graph.placeholder(shape:dataType:name:)`, `graph.constant(_:shape:dataType:)`
- `graph.run(feeds:targetTensors:targetOperations:)` — synchronous execution

## Code Highlights

Control dependency to keep assign operation alive:
```swift
let exp = graph.controlDependency(with: [assign],
                                  dependentBlock: {
                                      return [graph.exponent(with: input, name: nil)]
                                  },
                                  name: nil)
let results = graph.run(feeds: [inputTensor: inputs],
                        targetTensors: [exp],
                        targetOperations: nil)
```

Compile graph and disable type inference for variable-shaped inputs:
```swift
let descriptor = MPSGraphCompilationDescriptor()
descriptor.disableTypeInference()
let executable = graph.compile(with: nil,
                               feeds: /* feeds */,
                               targetTensors: /* target tensors */,
                               targetOperations: nil,
                               compilationDescriptor: descriptor)
```

While loop for iterative linear solver:
```swift
let results = graph.while(initialInputs: [initialValue],
                          before: { (inputs, returnTensors) -> MPSGraphTensor in
                              let predicate = graph.lessThan(inputs[0], threshold, name: nil)
                              returnTensors.add(inputs[0])
                              return predicate
                          },
                          after: { (inputs) -> [MPSGraphTensor] in
                              return [graph.multiplication(inputs[0], multiplier, name: nil)]
                          },
                          name: nil)
```

Stencil for Laplacian edge filter:
```swift
let edges = graph.stencil(with: source,
                          weights: laplacianWeights,
                          descriptor: desc,
                          name: nil)
```

## Takeaways
- MPSGraph now supports TensorFlow via a Metal Plugin, enabling GPU training on Apple hardware with no model changes and up to 8x speedups.
- Three new compute primitives (control dependency, stencil, gatherND) unlock a broader range of ML and image processing operations with optimal kernel stitching.
- `MPSGraphExecutable` and `disableTypeInference()` give developers precise control over compilation timing, dramatically reducing per-iteration latency for variable-input networks.
- Control flow APIs (if/else, for, while) allow complex iterative algorithms to execute entirely on the GPU without CPU synchronization overhead.

---
_Source: WWDC21 Session 10152 page (abstract, chapter summaries, code samples, and resource links)._
