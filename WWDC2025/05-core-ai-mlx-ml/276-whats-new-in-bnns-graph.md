# What's new in BNNS Graph
**WWDC25 · Session 276** · [Watch](https://developer.apple.com/videos/play/wwdc2025/276/)

_Platforms:_ iOS 26, iPadOS 26, macOS Tahoe 26, tvOS 26, visionOS 26

## Overview
BNNS (Basic Neural Network Subroutines) is Apple's high-performance CPU-based machine learning library, designed for real-time and low-latency use cases such as audio processing and image pipelines. This session introduces `BNNSGraphBuilder`, a brand-new Swift-native API that lets developers write graphs of operations entirely in Swift — eliminating the need for a separate Python/PyTorch file to generate the underlying graph.

Building on the file-based `BNNSGraph` API introduced at WWDC24, `BNNSGraphBuilder` compiles Swift closures into optimized, reusable execution contexts at app startup. It brings familiar Swift language features — type safety, subscripts, operators, and closures — to low-level ML graph authoring, making it practical for pre-processing, post-processing, and small standalone models.

The session walks through three concrete demonstrations: cropping an image with slicing, thresholding a grayscale image for binary output, applying softmax + topK post-processing, and reimplementing last year's Bitcrusher audio unit in pure Swift (with a switch between FP32 and FP16 via a single type alias).

## Key Topics

### Recapping BNNSGraph
`BNNSGraph` treats an entire model as a single graph object, enabling a suite of automatic optimizations: mathematical transformations (e.g., reordering slice before matmul), layer fusion (convolution + activation), copy elision, weight repacking for cache locality, and memory sharing across tensors. The file-based API ingests a compiled `.mlmodelc` from a CoreML Package and remains the recommended path for existing PyTorch models.

### BNNSGraphBuilder
The new `BNNSGraph.makeContext { builder in ... }` type method accepts a Swift closure that defines the full graph. The closure receives a `builder` parameter used to declare typed input/output arguments. The resulting context is compiled once at startup and executed on demand, benefiting from all BNNSGraph optimizations.

Key benefits:
- **Strong typing**: data-type mismatches are caught at compile time
- **Swift operators**: `*`, `+`, `-`, `/`, `.<`, `.>`, `.==`, logical operators
- **Slicing via subscripts and `SliceRange`**: no-copy window into a tensor
- **Shareable runtime constants**: shape values known before graph init can be baked in for better static-size performance
- **Tensor inspection**: print shape/dtype of intermediate tensors for debugging; Xcode autocomplete works on typed tensors

### Demonstrations
1. **Image cropping** — uses `SliceRange(startIndex:endIndex:)` and `.fillAll` subscript to crop a squirrel photo to 640×640
2. **Thresholding** — computes `mean` over all pixels, then applies `.>` to produce a binary image
3. **Post-processing (softmax + topK)** — creates a small on-the-fly graph; uses `k` captured from the outer Swift scope
4. **Bitcrusher audio unit** — Swift reimplementation mirrors the PyTorch original line-for-line; changing a single `typealias BITCRUSHER_PRECISION = Float16` switches the entire graph to FP16, yielding significantly faster execution

### New vImage integration
`vImage.PixelBuffer` gains a **`withBNNSTensor`** method that creates a zero-copy `BNNSTensor` sharing the pixel buffer's memory, making image-to-graph hand-off trivial.

## APIs & Frameworks

### Accelerate / BNNS
- **`BNNSGraph.makeContext(_:) → BNNSGraph.Context`** **[NEW]** — Swift-closure-based graph compilation
- **`BNNSGraph.Builder`** **[NEW]** — builder object provided inside `makeContext` closure
- `builder.argument(name:dataType:shape:)` **[NEW]** — declares typed input/output tensors
- **`BNNSGraph.Builder.SliceRange(startIndex:endIndex:)`** **[NEW]** — tensor slice range; negative `endIndex` means "dimension size minus n"
- **`BNNSGraph.Builder.SliceRange.fillAll`** **[NEW]** — include all elements along a dimension
- `BNNSGraph.Context.executeFunction(arguments:)` — runs the compiled graph
- `BNNSGraph.Context.argumentNames()` — returns output-first ordered argument names
- `BNNSGraph.Context.tensor(argument:fillKnownDynamicShapes:)` — retrieve a tensor for a given argument name
- `BNNSTensor.allocate(as:count:)` / `allocate(initializingFrom:)` / `deallocate()` / `makeArray(of:)`
- Tensor arithmetic operators: `*`, `+`, `-`, `/` **[NEW via Builder]**
- Tensor comparison operators: `.<`, `.>`, `.==` **[NEW via Builder]**
- Methods: `.mean(axes:keepDimensions:)`, `.tanh()`, `.round()`, `.pow(y:)`, `.cast(to:)`, `.softmax(axis:)`, `.topK(_:axis:findLargest:)` **[NEW]**

### Accelerate / vImage
- **`vImage.PixelBuffer.withBNNSTensor(_:)`** **[NEW]** — zero-copy bridge between a pixel buffer and a `BNNSTensor`
- `vImage.PixelBuffer` (existing), `vImage.InterleavedFx3`, `vImage.Planar16F`
- `vImage_CGImageFormat`

## Code Highlights

```swift
// Basic makeContext usage
let context = try BNNSGraph.makeContext { builder in
    let x = builder.argument(name: "x", dataType: Float.self, shape: [8])
    let y = builder.argument(name: "y", dataType: Float.self, shape: [8])
    let product = x * y
    let mean = product.mean(axes: [0], keepDimensions: true)
    return [product, mean]
}

// Image crop via slicing
let result = src[
    BNNSGraph.Builder.SliceRange(startIndex: verticalMargin, endIndex: -verticalMargin),
    BNNSGraph.Builder.SliceRange(startIndex: horizontalMargin, endIndex: -horizontalMargin),
    BNNSGraph.Builder.SliceRange.fillAll
]

// Bitcrusher — switch FP32↔FP16 with one line
typealias BITCRUSHER_PRECISION = Float16
```

## Takeaways
- Adopt `BNNSGraph.makeContext` for new projects requiring CPU-based pre/post-processing or small ML models; keep the file-based API for existing PyTorch models.
- Use `withBNNSTensor` on `vImage.PixelBuffer` to avoid copying image data into and out of graphs.
- Switch a single `typealias` to `Float16` to benchmark FP16 vs FP32 performance; FP16 can be substantially faster for suitable workloads.
- Check intermediate tensor shapes via `print(tensor)` inside the `makeContext` closure to debug graph construction before shipping.

---
_Source: WWDC25 Session 276 page (abstract, chapter summaries, code samples, and resource links)._
