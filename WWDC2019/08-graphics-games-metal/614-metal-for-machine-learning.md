# Metal for Machine Learning
**WWDC19 · Session 614** · [Watch](https://developer.apple.com/videos/play/wwdc2019/614/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15, tvOS 13

## Overview
This session covers significant new features in the Metal Performance Shaders (MPS) framework for machine learning, all targeting iOS 13, macOS Catalina, and tvOS 13. Three major additions land: implicit training graph creation (derive a full training graph from an inference graph automatically), separable forward and gradient loss kernels (support for networks with multiple intermediate losses like style transfer and GANs), and GPU-native random number generation (MTGP32 and Philox generators operating on `MPSMatrix`).

Two fully worked examples illustrate the new capabilities: a style transfer network demonstrating Gram Matrix calculation, separable losses, and implicit graph creation; and a ray-traced image denoising network demonstrating skip connections, encoder-decoder architecture, multi-channel input concatenation via a custom Metal compute shader, and cross-device deployment via MPS secure coding serialization.

Performance improvements include `MPSCommandBuffer` with `commitAndContinue` (near double-buffering GPU utilization with minimal memory overhead) and kernel predication (conditional kernel execution based on GPU-resident predicate values, enabling early exit from iterative loops like LSTM caption generation).

## Key Topics
- **Implicit training graph creation** — call `trainingGraph(withSourceGradient:nodeHandler:)` on a loss node to auto-generate all gradient nodes for the entire inference graph; `stopGradient` property enables partial training (transfer learning)
- **Separable loss kernels** — `MPSNNForwardLoss` and `MPSNNLossGradient` as distinct nodes; `MPSNNInitialGradient` to start back-propagation when losses are combined before differentiation
- **Gram Matrix** — `MPSNNGramMatrix` forward and gradient kernels for style loss computation **[NEW]**
- **Random number generation** — `MPSMatrixRandom` with MTGP32 and Philox generators; operates on `MPSMatrix`/`MPSVector` in GPU memory; eliminates CPU-GPU sync for large random arrays **[NEW]**
- **Predication** — `MPSPredicate` backed by a Metal buffer; attach to `MPSCommandBuffer` to conditionally execute kernels based on GPU-resident values
- **`MPSCommandBuffer`** — MTLCommandBuffer-conforming class; `commitAndContinue()` enables near double-buffering performance with significantly lower memory overhead than full double buffering
- **Serialization** — all MPS kernels and graphs support `NSSecureCoding`; `MPSKeyedUnarchiver` restores networks to a specific Metal device for cross-device deployment

## APIs & Frameworks
- **Metal Performance Shaders (MPS)**
  - `MPSNNGraph` — inference and training graph
    - `trainingGraph(withSourceGradient:nodeHandler:)` — implicit training graph creation **[NEW]**
  - `MPSNNFilterNode.stopGradient: Bool` **[NEW]** — prevent gradient propagation past this node
  - `MPSNNForwardLoss` — forward-only loss node **[NEW]**
  - `MPSNNLossGradient` — gradient-only loss node **[NEW]**
  - `MPSNNInitialGradient` — generates initial gradient image of ones **[NEW]**
  - `MPSCNNLoss` — combined forward+gradient loss (existing)
  - `MPSNNGramMatrix` — Gram Matrix forward node **[NEW]**
  - `MPSNNGramMatrixGradient` — Gram Matrix gradient node **[NEW]**
  - `MPSMatrixRandom` — random number generator class **[NEW]**
    - `MPSMatrixRandomMTGP32` — Mersenne Twister GP32 variant **[NEW]**
    - `MPSMatrixRandomPhilox` — counter-based Philox generator **[NEW]**
    - `MPSMatrixRandomDistributionDescriptor` — configure uniform float distribution **[NEW]**
    - `encode(to:destinationMatrix:)` — encode RNG to command buffer
  - `MPSPredicate` — wraps a MTLBuffer with predicate integer value and offset **[NEW]**
  - `MPSCommandBuffer` — MTLCommandBuffer conforming class **[NEW]**
    - `init(from: MTLCommandQueue)`
    - `commitAndContinue()` — commit encoded work and continue encoding **[NEW]**
    - `predicate: MPSPredicate?` — attach predicate to command buffer **[NEW]**
  - `MPSImage` / `MPSTemporaryImage`
  - `MPSMatrix` / `MPSVector`
  - `MPSCNNConvolutionNode` / `MPSCNNPoolingMaxNode` / `MPSCNNFullyConnectedNode`
  - `MPSCNNNeuronReLUNode` / `MPSCNNNeuronTanhNode`
  - `MPSNNUpsamplingNearestNode`
  - `MPSNNAdditionNode`
  - MPS Secure Coding: `NSSecureCoding` conformance on all kernels and graphs
  - `MPSKeyedUnarchiver` — device-aware unarchiver for cross-device restore **[NEW]**
- **Metal**
  - Custom Metal compute shaders for multi-channel image concatenation
  - `MTLBuffer` — backing store for predicate values
  - `MTLCommandQueue` — source for `MPSCommandBuffer`

## Code Highlights

```swift
// Implicit training graph creation from inference graph
let lossNode = MPSNNForwardLoss(source: inferenceOutputNode, labels: groundTruthNode)
let initialGrad = MPSNNInitialGradient(source: lossNode)
let trainingGraph = MPSNNGraph(device: device,
                               resultImages: [initialGrad.resultImage],
                               resultImageIsNeeded: [false])
// All gradient nodes are created automatically
```

```swift
// Random number generation (GPU-resident)
let descriptor = MPSMatrixRandomDistributionDescriptor.uniformDistributionDescriptor(
    withMinimum: 0.0, maximum: 1.0)
let rng = MPSMatrixRandomPhilox(device: device,
                                destinationDataType: .float32,
                                seed: 42,
                                distributionDescriptor: descriptor)
let randomMatrix = MPSMatrix(buffer: buffer, descriptor: matrixDescriptor)
rng.encode(to: commandBuffer, destinationMatrix: randomMatrix)
```

```swift
// MPSCommandBuffer with commitAndContinue
let mpsCmdBuf = MPSCommandBuffer(from: commandQueue)
kernel1.encode(commandBuffer: mpsCmdBuf, ...)
kernel2.encode(commandBuffer: mpsCmdBuf, ...)
mpsCmdBuf.commitAndContinue()  // commit first two, continue encoding
kernel3.encode(commandBuffer: mpsCmdBuf, ...)
kernel4.encode(commandBuffer: mpsCmdBuf, ...)
mpsCmdBuf.commit()
```

```swift
// Serialization for cross-device deployment
let coder = NSKeyedArchiver(requiringSecureCoding: true)
trainingGraph.encode(with: coder)
let data = coder.encodedData
try data.write(to: archiveURL)

// Restore on a different device (e.g., iPad)
let unarchiver = try MPSKeyedUnarchiver(forReadingFrom: data, device: targetDevice)
let restoredGraph = try unarchiver.decodeTopLevelObject(of: MPSNNGraph.self, forKey: NSKeyedArchiveRootObjectKey)
```

## Takeaways
- Implicit training graph creation reduces training graph setup to a single call, eliminating dozens of manually written gradient node declarations.
- Separable forward/gradient loss kernels unlock network architectures like style transfer and GANs that require multiple independent loss values before back-propagation.
- `MPSCommandBuffer.commitAndContinue()` achieves near double-buffering GPU utilization at a fraction of the memory cost — use it by default for MPS graph execution.
- MPS secure coding enables training on Mac and deploying the serialized network for inference on iPhone/iPad without any weight-management boilerplate.

---
_Source: WWDC19 Session 614 page (abstract, chapter summaries, code samples, and resource links)._
