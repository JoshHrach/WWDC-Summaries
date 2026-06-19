# Build Customized ML Models with the Metal Performance Shaders Graph
**WWDC20 · Session 10677** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10677/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, tvOS 14

## Overview
This session introduces the Metal Performance Shaders Graph (MPSGraph), a new framework that extends Metal's compute capabilities to multi-dimensional tensors. MPSGraph sits on top of the existing MPS framework and provides a unified DAG-based API for expressing, optimizing, and executing both ML inference and training workloads entirely on the GPU — without requiring kernel extensions or custom Metal shaders.

Three major demos illustrate the breadth of MPSGraph: a GeLU activation function as a custom compute graph, an LSTM-based Shakespearean text generator (inference), and a GAN (Generative Adversarial Network) for handwritten digit generation (training). The session also covers MPS framework improvements: FP32 support for CNN primitives and the new `MPSNDArray` multi-dimensional tensor primitive.

A key architectural feature is the MPSGraph compiler's automatic "stitching" optimization, which fuses adjacent element-wise operations into single optimized Metal shaders — including fusion into hand-tuned MPS kernels like convolution — delivering up to 10–50x speedups with zero code changes.

## Key Topics

**MPSNDArray**
A new primitive supporting up to 16 dimensions with virtually no size restrictions, bridging to/from `MTLTexture`-based `MPSImageBatch` and `MTLBuffer`. Forms the tensor backing for `MPSGraphTensorData`.

**MPSGraph Core Concepts**
- A directed acyclic graph (DAG) of operations (nodes) and tensors (edges).
- `MPSGraphTensor`: symbolic data abstraction with shape, data type, and owning operation.
- Placeholders: symbolic inputs replaced at runtime by `MPSGraphTensorData`.
- Dynamic shapes: specify `-1` for unknown dimensions; `nil` for fully dynamic rank.
- Variables: tensors that retain values across graph runs (for trainable parameters).
- Control dependencies: edges ensuring execution order without data dependency.

**Compiler Optimizations**
- Stitching: adjacent element-wise ops are fused by the Metal compiler into a single shader.
- Fusion into hand-tuned kernels: stitchable regions around MPS primitives (convolution, matmul, reduction) are fused directly into those kernels.
- Dead code elimination and constant folding in the backward pass.
- Graph compilation is cached after the first run for identical input/output types.

**Inference and Training**
- Inference: parameters stored as constants; supports LSTM, convolution, matmul, softmax.
- Training: uses `MPSGraph` variables + automatic differentiation + optimizer nodes + variable assignment in a single graph.
- GAN training: two networks in one graph, differentiated selectively by specifying which variables to compute gradients for; both trained in a single `run` call.

## APIs & Frameworks

### Metal Performance Shaders Graph **[NEW]**
- `MPSGraph` **[NEW]** — the graph object; owns all tensors and operations
- `MPSGraphTensor` **[NEW]** — symbolic tensor: shape, dataType, source operation
- `MPSGraphTensorData` **[NEW]** — runtime data wrapper for `MTLBuffer`, `MPSImageBatch`, `MPSNDArray`, `MPSVector`, `MPSMatrix`
- `MPSGraph.placeholder(shape:dataType:name:)` **[NEW]** — symbolic input
- `MPSGraph.constant(_:dataType:)` / `constant(_:shape:dataType:)` **[NEW]** — constant tensor
- `MPSGraph.variable(with:shape:dataType:name:)` **[NEW]** — persistent variable tensor
- `MPSGraph.readVariable(_:name:)` **[NEW]** — reads variable value (implicit when passing variable directly)
- `MPSGraph.assignVariable(_:newValue:name:)` **[NEW]** — assigns new value to variable
- `MPSGraph.run(feeds:targetTensors:targetOperations:)` **[NEW]** — synchronous execution
- `MPSGraph.runAsync(feeds:targetTensors:targetOperations:executionDescriptor:)` **[NEW]** — asynchronous execution
- `MPSGraph.encode(to:feeds:targetTensors:targetOperations:executionDescriptor:)` **[NEW]** — encode on `MPSCommandBuffer`
- `MPSGraphExecutionDescriptor` **[NEW]** — completion handler and synchronization
- Math operations: `multiply(_:_:)`, `add(_:_:)`, `subtract(_:_:)`, `squareRoot(_:)`, `erf(_:)`, `log(_:)`, `reciprocal(_:)` **[NEW]**
- Shape operations: `reshape(_:shape:)`, `slice(_:dimension:start:length:)`, `concat(_:with:dimension:)`, `transpose(_:dimension:withDimension:)` **[NEW]**
- `MPSGraph.convolution2D(_:weights:descriptor:name:)` **[NEW]**
- `MPSGraph.convolution2DDataGradient(_:weights:outputShape:descriptor:name:)` **[NEW]**
- `MPSGraph.convolution2DWeightsGradient(_:source:outputShape:descriptor:name:)` **[NEW]**
- `MPSGraph.convolutionTranspose2D(_:weights:outputShape:descriptor:name:)` **[NEW]**
- `MPSGraphConvolution2DOpDescriptor` **[NEW]** — padding styles: SAME, VALID, explicit; data layouts: NCHW, NHWC, OIHW, HWIO
- `MPSGraph.softMax(_:axis:name:)` **[NEW]**
- `MPSGraph.softMaxCrossEntropy(_:labels:axis:reductionType:name:)` **[NEW]**
- `MPSGraph.reductionSum(with:axis:name:)` **[NEW]**
- `MPSGraph.mean(of:axes:name:)`, `MPSGraph.variance(of:meanTensor:axes:name:)` **[NEW]**
- `MPSGraph.normalize(_:mean:variance:gamma:beta:epsilon:name:)` **[NEW]** — batch/instance/layer norm
- `MPSGraph.dropout(_:rate:name:)` **[NEW]**
- `MPSGraph.gather(withUpdatesTensor:indicesTensor:axis:batchDimensions:name:)` **[NEW]** — embedding lookup
- `MPSGraph.gradients(of:withRespectTo:name:)` **[NEW]** — automatic differentiation
- `MPSGraph.stochasticGradientDescent(learningRate:values:gradient:name:)` **[NEW]** — SGD optimizer
- `MPSGraph.LSTM(_:recurrentWeight:inputWeight:bias:initState:initCell:descriptor:name:)` **[NEW]**
- `MPSGraphLSTMDescriptor` **[NEW]**

### Metal Performance Shaders (enhancements)
- `MPSNDArray` **[NEW]** — N-dimensional array (up to 16 dims)
- `MPSNDArrayDescriptor` **[NEW]** — shape and dataType specification
- Full FP32 support for `MPSCNNConvolution` and `MPSCNNFullyConnected` **[NEW]** — via `kernelWeightsDataType` returning `.float32`

## Code Highlights

Creating a placeholder with dynamic batch size:
```swift
let input = graph.placeholder(shape: [-1, 28, 28, 1], dataType: .float32, name: "input")
```

Executing the graph synchronously:
```swift
let results = graph.run(
    feeds: [inputTensor: MPSGraphTensorData(buffer, shape: [...], dataType: .float32)],
    targetTensors: [geluOutput],
    targetOperations: nil
)
```

Automatic differentiation for selective parameter updates:
```swift
let grads = graph.gradients(of: loss, withRespectTo: [weightsVar, biasVar], name: "grads")
let weightGrad = grads[weightsVar]!
```

Asynchronous training loop with double-buffering:
```swift
let semaphore = DispatchSemaphore(value: 2)
let desc = MPSGraphExecutionDescriptor()
desc.completionHandler = { _, _ in semaphore.signal() }
for _ in trainingIterations {
    semaphore.wait()
    graph.runAsync(feeds: feeds, targetTensors: [loss],
                   targetOperations: [updateWeights, updateBias], executionDescriptor: desc)
}
```

## Takeaways
- MPSGraph is a new framework for GPU-accelerated tensor computation supporting custom inference and training — including GAN-style multi-network training — with automatic differentiation and optimizer nodes.
- The stitching compiler optimization automatically fuses element-wise operations into single Metal shaders and into hand-tuned MPS kernels, delivering 10–50x speedups with no code changes required.
- Variables persist across graph runs, eliminating the need to pass trainable parameters as feed inputs each iteration; selective gradient computation (by specifying only target variables) allows per-network gradient isolation in multi-network training.
- Graph compilation is cached after the first invocation; use asynchronous run methods with double-buffered semaphores to maximize CPU/GPU overlap during training loops.

---
_Source: WWDC20 Session 10677 page (abstract, chapter summaries, code samples, and resource links)._
