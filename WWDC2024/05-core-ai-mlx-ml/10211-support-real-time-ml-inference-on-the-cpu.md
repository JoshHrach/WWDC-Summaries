# Support Real-Time ML Inference on the CPU
**WWDC24 · Session 10211** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10211/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia 15, visionOS 2, watchOS 11

## Overview
BNNS Graph is a new API in the Accelerate framework's BNNS library that lets the system consume an entire ML model graph as a single object, rather than requiring developers to wire up individual layer primitives. This enables automatic optimizations—mathematical transformations, layer fusion, copy elision, shared-memory tensor layouts, and weight repacking—that on average deliver at least 2x faster inference compared to the previous BNNS primitives API.

The key advantage of BNNS Graph for real-time workloads is explicit control over memory and threading. Developers pre-allocate a page-aligned workspace buffer and optionally pin execution to a single thread, ensuring the execute phase has zero runtime memory allocations and no context switches into kernel code—critical requirements for audio processing, signal processing, and any other low-latency, real-time use case.

The session demonstrates building a real-time bitcrusher AudioUnit that uses BNNS Graph from both C++ (for the real-time DSP kernel) and Swift (for SwiftUI chart visualization using the same model), showing the consistent API surface across both languages.

## Key Topics

### Introducing BNNS Graph
`BNNSGraphCompileFromFile` compiles an `.mlmodelc` file into a `bnns_graph_t` object. The graph is immutable and captures the full computation graph, enabling cross-layer optimizations not possible when working with individual BNNS primitives. Optimizations include: slice movement (mathematical transformation), convolution+activation layer fusion, copy elision for slice operations, shared memory between compatible tensors, and weight repacking for cache locality.

### Real-Time Processing Requirements
Real-time audio processing (e.g., in an AudioUnit) demands no memory allocation and no multithreading during the execute phase to avoid kernel context switches that would miss real-time deadlines. BNNS Graph addresses this through: `BNNSGraphCompileOptionsSetTargetSingleThread` to pin execution to one thread, a pre-allocated page-aligned workspace via `aligned_alloc` / `UnsafeMutableRawBufferPointer.allocate`, and `BNNSGraphArgumentTypePointer` mode to pass raw pointers directly rather than tensor descriptor structures.

### Adopting BNNS Graph
The workflow is: (1) drag `.mlpackage` into Xcode—it compiles to `.mlmodelc`; (2) call `BNNSGraphCompileFromFile` to build the graph; (3) call `BNNSGraphContextMake` to create a mutable context; (4) set argument type, dynamic shapes, and batch size; (5) query `BNNSGraphContextGetWorkspaceSize` and pre-allocate workspace; (6) use `BNNSGraphGetArgumentPosition` to get stable argument indices; (7) call `BNNSGraphContextExecute` on each inference.

### BNNS Graph in Swift
The API is consistent between C/Objective-C and Swift. Swift types like `UnsafeMutableBufferPointer<Float>` serve as argument buffers, and `bnns_graph_argument_t` structs are populated with `data_ptr` and `data_ptr_size` fields. Separate contexts are required per thread (context can only execute on one thread at a time), so a SwiftUI visualization component must use its own context distinct from the real-time audio context.

## APIs & Frameworks

- `Accelerate` framework — parent framework containing BNNS
- `BNNS` (Basic Neural Network Subroutines) — ML inference/training library on CPU
- `BNNSGraph` **[NEW]** — new graph-level API for whole-model inference
- `bnns_graph_t` **[NEW]** — immutable compiled graph object
- `bnns_graph_context_t` **[NEW]** — mutable execution context wrapping a graph
- `bnns_graph_compile_options_t` **[NEW]** — compilation configuration structure
- `BNNSGraphCompileOptionsMakeDefault()` **[NEW]** — creates default compile options
- `BNNSGraphCompileOptionsSetTargetSingleThread(_:_:)` **[NEW]** — pins execution to a single thread
- `BNNSGraphCompileFromFile(_:_:_:)` **[NEW]** — compiles `.mlmodelc` file into a `bnns_graph_t`
- `BNNSGraphCompileOptionsDestroy(_:)` **[NEW]** — deallocates compile options after use
- `BNNSGraphContextMake(_:)` **[NEW]** — wraps graph in a mutable execution context
- `BNNSGraphContextSetArgumentType(_:_:)` **[NEW]** — sets argument passing mode
- `BNNSGraphArgumentTypePointer` **[NEW]** — direct pointer argument mode (no tensor structs)
- `BNNSGraphContextSetDynamicShapes(_:_:_:_:)` **[NEW]** — configures max input/output shapes for dynamic batching
- `BNNSGraphContextGetWorkspaceSize(_:_:)` **[NEW]** — queries required workspace size in bytes
- `BNNSGraphContextSetBatchSize(_:_:_:)` **[NEW]** — updates first-dimension size before each execute
- `BNNSGraphGetArgumentPosition(_:_:_:)` **[NEW]** — returns stable index for a named argument
- `BNNSGraphContextExecute(_:_:_:_:_:_:)` **[NEW]** — runs inference; zero allocations when workspace is pre-provided
- `bnns_graph_argument_t` **[NEW]** — struct with `data_ptr` and `data_ptr_size` fields
- `bnns_graph_shape_t` **[NEW]** — struct with `rank` and `shape` fields
- `BNNSGraphCompileOptionsSetOptimizationPreference` — choose performance vs. size (performance is default)
- `BNNSGraphContextEnableNaNAndInfinityChecks` — debug-only NaN/Inf detection in tensors
- `aligned_alloc` — C stdlib function for page-aligned workspace allocation
- `NSPageSize()` — used for alignment value in workspace allocation
- `AudioUnit` / Audio Unit Extension — real-time audio processing plugin architecture
- `Core ML` (`.mlpackage`, `.mlmodelc`) — upstream model format consumed by BNNS Graph

## Code Highlights

Compile graph with single-thread execution (C++):
```objc
bnns_graph_compile_options_t options = BNNSGraphCompileOptionsMakeDefault();
BNNSGraphCompileOptionsSetTargetSingleThread(options, true);
bnns_graph_t graph = BNNSGraphCompileFromFile(mlmodelc_path.UTF8String, NULL, options);
BNNSGraphCompileOptionsDestroy(options);
```

Pre-allocate workspace and get argument positions:
```objc
BNNSGraphContextSetArgumentType(context, BNNSGraphArgumentTypePointer);
uint64_t shape[] = {mMaxFramesToRender, 1, 1};
bnns_graph_shape_t shapes[] = { {.rank=3, .shape=shape}, {.rank=3, .shape=shape} };
BNNSGraphContextSetDynamicShapes(context, NULL, 2, shapes);
workspace_size = BNNSGraphContextGetWorkspaceSize(context, NULL) + NSPageSize();
workspace = (char *)aligned_alloc(NSPageSize(), workspace_size);
dst_index = BNNSGraphGetArgumentPosition(graph, NULL, "dst");
src_index = BNNSGraphGetArgumentPosition(graph, NULL, "src");
```

Execute graph in real-time audio render callback:
```objc
BNNSGraphContextSetBatchSize(context, NULL, frameCount);
// populate arguments[] array at pre-computed indices...
BNNSGraphContextExecute(context, NULL, 5, arguments, workspace_size, workspace);
```

## Takeaways

- BNNS Graph consumes an entire `.mlmodelc` as a single object, unlocking cross-layer optimizations (fusion, copy elision, weight repacking) that deliver on average 2x or greater speedup over individual BNNS primitives.
- Pre-allocating a page-aligned workspace and using `BNNSGraphArgumentTypePointer` mode guarantees zero runtime memory allocation during inference—essential for AudioUnit and other real-time signal processing workloads.
- `BNNSGraphCompileOptionsSetTargetSingleThread` ensures single-threaded execution, preventing context switches that would violate real-time deadlines.
- The Swift and C APIs are consistent; use separate `bnns_graph_context_t` instances for each concurrent execution context (e.g., one for the audio DSP thread, one for the SwiftUI preview thread).

---
_Source: WWDC24 Session 10211 page (abstract, chapter summaries, code samples, and resource links)._
