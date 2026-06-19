# Accelerate Machine Learning with Metal
**WWDC22 · Session 10063** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10063/)

_Platforms:_ macOS Ventura 13, iOS 16, iPadOS 16

## Overview
This session covers the major new GPU-accelerated machine learning capabilities introduced in Metal for 2022. Apple engineers walk through PyTorch GPU acceleration via the new MPS backend, enhancements to TensorFlow Metal plug-in, and new operations and features in the MPSGraph framework.

PyTorch support arrives as the most requested feature in the PyTorch community — enabling GPU acceleration on Apple silicon through a new MPS backend merged into the official PyTorch 1.12 release. TensorFlow Metal gains custom operations, distributed training via Horovod, and improved batch size performance due to unified memory architecture.

MPSGraph receives a new shared events API for multi-queue synchronization, plus LSTM/RNN/GRU layers, improved Max Pooling with return indices, a Philox random number generator, Hamming distance, and a suite of new tensor manipulation operations.

## Key Topics

### PyTorch MPS Backend
- New `mps` device backend in PyTorch 1.12 implementing PyTorch operation kernels via MPS Graph and MPS primitives
- Uses Metal Command queues, Command buffers, and synchronization primitives under the hood
- Three-step adoption: install with pip, create MPS device, convert model/tensors with `.to(device=mpsDevice)`
- Demonstrated up to 20x speedup on M1 Ultra across popular benchmarks, 8.3x average

### TensorFlow Metal Plug-in Enhancements
- Larger batch sizes enabled by Apple Silicon unified memory architecture
- New GPU-accelerated ops: `argMin`, `all`, `pack`, `adaDelta`, and more
- Custom operation support via the `TF_MetalStream` protocol: register with `REGISTER_OP`, implement using `MTLCommandBuffer`, import via `tf.load_op_library`
- Distributed training via Horovod ring all-reduce: demonstrated near-linear scaling across 4 Mac Studios connected via Thunderbolt (200 → 800 images/sec for ResNet training)

### MPSGraph New Features
- Shared Events API for cross-command-queue synchronization
- New RNN/LSTM/GRU operations for natural language processing
- Max Pooling with return indices (up to 6x faster gradient pass for PyTorch/TensorFlow)
- Philox parallel random number generator (compatible with TensorFlow for a given seed)
- Hamming distance operation with batch broadcasting
- Tensor manipulation: `expandDims`, `squeeze`, `split`, `stack`, `coordinateAlongAxis`

## APIs & Frameworks

**Metal / MPS**
- `Metal` (framework) **[updated]**
- `MPSImageCanny` — up to 8x faster on 4K images
- `MetalPerformanceShaders` (framework)

**MPSGraph** **[NEW ops]**
- `MPSGraph` (class)
- `MPSGraphExecutionDescriptor` **[NEW]** — `.signal(_:atExecutionEvent:value:)`, `.wait(for:value:)` for shared event synchronization
- `MTLSharedEvent` — created via `MTLDevice.makeSharedEvent()`
- `MPSGraph.LSTM(_:recurrentWeight:inputWeight:bias:initState:initCell:descriptor:name:)` **[NEW]**
- `MPSGraphLSTMDescriptor` **[NEW]** — properties: `inputGateActivation`, `forgetGateActivation`, `cellGateActivation`, `outputGateActivation`, `activation`, `bidirectional`, `training`
- `MPSGraph.GRU(...)` **[NEW]**
- `MPSGraph.RNN(...)` **[NEW]**
- `MPSGraph.maxPooling4DReturnIndices(_:descriptor:name:)` **[NEW]**
- `MPSGraph.maxPooling4DGradient(gradient:indices:outputShape:descriptor:name:)` **[NEW]**
- `MPSGraphPooling4DOpDescriptor` — `.returnIndicesMode` property (e.g., `.globalFlatten4D`) **[NEW]**
- `MPSGraph.randomPhiloxStateTensor(seed:name:)` **[NEW]**
- `MPSGraphRandomOpDescriptor` **[NEW]** — `.distribution` (`.truncatedNormal`, `.normal`, `.uniform`), `.dataType`, `.mean`, `.standardDeviation`, `.min`, `.max`
- `MPSGraph.randomTensor(shapeTensor:descriptor:stateTensor:name:)` **[NEW]**
- `MPSGraph.HammingDistance(primary:secondary:resultDataType:name:)` **[NEW]**
- `MPSGraph.expandDims(_:axis:name:)` **[NEW]**
- `MPSGraph.squeeze(_:axis:name:)` **[NEW]**
- `MPSGraph.split(_:numSplits:axis:name:)` **[NEW]**
- `MPSGraph.stack(_:axis:name:)` **[NEW]**
- `MPSGraph.coordinateAlongAxis(axis:shape:name:)` / `coordinate(alongAxis:withShape:name:)` **[NEW]**

**TensorFlow Metal Plug-in**
- `TF_MetalStream` protocol — `currentCommandBuffer`, `queue`, `commit()`, `commitAndWait()`
- `REGISTER_OP` macro for custom op registration
- `TF_GetInput`, `TF_GetStream`, `TF_DeleteTensor` C APIs
- `tf.load_op_library(_:)` Python API

**PyTorch (MPS backend)**
- `torch.device("mps")` **[NEW]**
- `torch.backends.mps.is_available()` **[NEW]**
- `model.to(device:)` — converts model to MPS device
- `torch.randn(..., device: mpsDevice)` — allocates tensor on GPU

## Code Highlights

Install and use PyTorch MPS backend:
```python
import torch
mpsDevice = torch.device("mps" if torch.backends.mps.is_available() else "cpu")
model = torchvision.models.resnet50().to(device=mpsDevice)
sample_input = torch.randn((32, 3, 254, 254), device=mpsDevice)
prediction = model(sample_input)
```

MPSGraph shared events across command queues:
```swift
let event = MTLCreateSystemDefaultDevice()!.makeSharedEvent()!
let desc1 = MPSGraphExecutionDescriptor()
desc1.signal(event, atExecutionEvent: .completed, value: 1)
let desc2 = MPSGraphExecutionDescriptor()
desc2.wait(for: event, value: 1)
```

MPSGraph LSTM:
```swift
let descriptor = MPSGraphLSTMDescriptor()
descriptor.training = true
let lstm = graph.LSTM(inputTensor, recurrentWeight: recurrentWeightsTensor,
                      inputWeight: weightsTensor, bias: nil,
                      initState: nil, initCell: nil, descriptor: descriptor, name: nil)
```

## Takeaways
- PyTorch 1.12 includes an official MPS backend, enabling GPU-accelerated training on Apple silicon Macs with minimal code changes.
- TensorFlow Metal plug-in now supports custom GPU operations via `TF_MetalStream`, enabling developers to accelerate any operation not in the standard API.
- Distributed training across multiple Macs connected via Thunderbolt achieves near-linear scaling using Horovod.
- MPSGraph gains critical new ops (LSTM, max-pooling indices, Philox RNG, Hamming distance) and a shared events API for fine-grained multi-queue synchronization.

---
_Source: WWDC22 Session 10063 page (abstract, chapter summaries, code samples, and resource links)._
