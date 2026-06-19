# Optimize your Core ML usage
**WWDC22 · Session 10027** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10027/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, watchOS 9, tvOS 16

## Overview
This session covers the latest tools and API enhancements for understanding and optimizing Core ML performance in apps. It introduces two major new tooling additions in Xcode 14: **Performance Reports**, which generate pre-integration benchmarks directly from the Xcode model viewer without writing any code, and the new **Core ML Instrument** in Instruments, which profiles model load, prediction, data transformation, and compute unit dispatch events live while an app runs.

On the API side, Core ML gains native end-to-end **Float16 support** (both images and MultiArrays), an **output backing API** that lets apps provide preallocated output buffers, a new **`cpuAndNeuralEngine` compute unit option**, **in-memory model loading** for custom encryption, and **Swift package support** for bundling Core ML models. Weight compression for ML Programs is also extended with quantization, palettization, and sparsification.

## Key Topics

### Performance Reports (Xcode 14)
- New **Performance tab** in the Xcode model viewer (between Predictions and Utilities tabs)
- Generates on-device benchmarks: compile time, load time, median prediction time
- Shows per-layer compute unit assignment (Neural Engine, GPU, CPU) with filled/unfilled checkmarks
- Works across multiple hardware and OS combinations without writing any code

### Core ML Instrument (Instruments / Xcode 14)
- New **Core ML template** in Instruments includes Core ML Instrument, GPU Instrument, and new Neural Engine Instrument
- Three grouped lanes: **Activity** (load/prediction API calls), **Data** (data checks and transformations), **Compute** (per-compute-unit requests)
- **Model Activity Aggregation** view shows aggregate statistics (count, mean, total duration) sortable by duration
- Supports per-model breakdown (subtrack per model)
- Demonstrated catching a bug: model re-loaded on every prediction due to a computed property instead of a lazy variable

### Float16 Native Support (NEW — iOS 16 / macOS Ventura)
- Core ML now natively supports `OneComponent16Half` grayscale images and `Float16` MultiArrays
- Eliminates CPU-side data conversion overhead (casting 8-bit ↔ Float16)
- Specify new color layouts and data types during `coremltools.convert()` call
- IOSurface-backed buffers allow zero-copy data transfer between compute units via unified memory
- Requires minimum deployment target iOS 16 / macOS Ventura

### Output Backing API (NEW)
- Pre-allocate output buffers and set them on `MLPredictionOptions` via `outputBackings` property
- Core ML fills the preallocated buffer instead of allocating a new one each prediction
- Improves buffer management control and reduces allocation overhead for high-throughput workloads

### Weight Compression for ML Programs (Extended)
- 16-bit and 8-bit quantization support extended to ML Program model type (previously neural-network-only)
- New: **sparse weight representation** (sparsification) for ML Programs
- `coremltools` utilities: `quantize()`, `palettize()`, `sparsify()` for ML Program models

### New `cpuAndNeuralEngine` Compute Unit Option
- Prevents Core ML from dispatching to the GPU
- Useful when the app is concurrently using the GPU for rendering/other computation

### In-Memory Model Loading (NEW)
- Compile and load a Core ML model specification from memory without requiring a compiled `.mlmodelc` on disk
- Enables custom encryption and decryption of model data before loading

### Swift Package Support (NEW in Xcode 14)
- Core ML models can be included in Swift packages
- Xcode automatically compiles and bundles `.mlmodel` files in imported packages
- Code generation interface works the same as in app targets

## APIs & Frameworks

**CoreML**
- `MLModelConfiguration` — model configuration
  - `.computeUnits: MLComputeUnits` — set CPU/GPU/Neural Engine preferences
  - `.computeUnits = .cpuAndNeuralEngine` — **[NEW]** exclude GPU from dispatch
- `MLPredictionOptions` — prediction-time options
  - `.outputBackings: [String: Any]` — **[NEW]** preallocated output buffer dictionary
- `MLModel.init(contentsOf:configuration:)` — standard load from disk
- `MLModel.compileModel(at:)` — compile model from URL
- `MLModel.init(modelStructure:configuration:)` — **[NEW]** in-memory model loading from `MLModelStructure`
- `MLFeatureValue` — model input/output value wrapper
- `MLMultiArray` — multi-dimensional array input/output
  - `MLMultiArrayDataType.float16` — **[NEW]**
- `CVPixelBuffer` with `kCVPixelFormatType_OneComponent16Half` — **[NEW]** supported as model I/O
- `IOSurface`-backed `CVPixelBuffer` — recommended for zero-copy unified memory transfer

**Instruments (Xcode 14)**
- Core ML Instrument — **[NEW]** Activity / Data / Compute lanes, aggregation view, per-model subtrack
- Neural Engine Instrument — **[NEW]**
- GPU Instrument — existing, can be combined with Core ML and Neural Engine instruments

**coremltools (Python)**
- `ct.convert()` — model conversion from PyTorch/TensorFlow; now accepts Float16 image color layouts and MultiArray data types
- `ct.optimize.coreml.quantize_weights()` — extended to ML Program type
- `ct.optimize.coreml.palettize_weights()` — extended to ML Program type
- `ct.optimize.coreml.sparsify_weights()` — **[NEW]** sparse weight representation for ML Programs

## Code Highlights

Providing a Float16 pixel buffer directly as model input (no manual casting):
```swift
// Input: OneComponent16Half CVPixelBuffer — passed directly, no conversion needed
let inputBuffer: CVPixelBuffer = myFloat16PixelBuffer // IOSurface-backed

// Output: preallocate and set as backing buffer
let outputBacking = outputBackingBuffer() // returns OneComponent16Half CVPixelBuffer
let options = MLPredictionOptions()
options.outputBackings = ["output": outputBacking]

let prediction = try model.prediction(from: input, options: options)
```

Lazy model property to avoid repeated loads:
```swift
// Wrong: reloads model on every access
var styleTransferModel: StyleTransfer { try! StyleTransfer() }

// Correct: load once
lazy var styleTransferModel: StyleTransfer = try! StyleTransfer()
```

## Takeaways
- Use Xcode 14's Performance Reports to benchmark model load, compile, and prediction time on real hardware before writing integration code; layer-level compute unit assignment shows where each layer runs.
- The Core ML Instrument reveals hidden costs (repeated loads, unnecessary data transformations) in live app profiling — always combine with the GPU and Neural Engine instruments for full visibility.
- Float16 native support (iOS 16+) eliminates round-trip data casting; pair with IOSurface-backed buffers and the output backing API for maximum throughput in image-processing pipelines.
- Core ML models can now be distributed in Swift packages with automatic compilation and code generation — no extra setup needed by package consumers.

---
_Source: WWDC22 Session 10027 page (abstract, chapter summaries, and resource links)._
