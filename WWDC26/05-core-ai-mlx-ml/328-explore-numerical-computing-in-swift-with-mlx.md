# Explore Numerical Computing in Swift with MLX
**WWDC26 · Session 328** · [Watch](https://developer.apple.com/videos/play/wwdc2026/328/)

_Platforms:_ macOS, iOS, iPadOS

## Overview
MLX Swift brings NumPy-style numerical computing natively to Swift, enabling array-level GPU-accelerated computation without leaving the Swift toolchain. This session introduces the core concepts of MLX Swift — lazy evaluation, n-dimensional arrays, and automatic differentiation — and demonstrates them through three progressively complex examples: fractal rendering (Mandelbrot set), physical simulation (heat distribution via Jacobi iteration and SOR), and machine learning (quadratic curve fitting via gradient descent).

The session positions MLX Swift within Apple's broader numerical computing ecosystem alongside Accelerate, BNNS, MPS, and Swift Numerics, and explains when MLX Swift is the right choice: when you want to write code that closely mirrors mathematical notation, get automatic GPU execution, and use automatic differentiation for optimization — all in a type-safe Swift environment.

## Key Topics

### MLX Swift and the Apple Ecosystem
MLX Swift complements existing frameworks. Choose it when the primary goal is writing mathematical code that runs on the GPU with automatic differentiation — as opposed to Accelerate (CPU-optimized BLAS/LAPACK), MPS (production inference primitives), or Swift Numerics (scalar mathematics).

### Core Concepts: Arrays, Lazy Evaluation, and eval()
The central abstraction is `MLXArray` — an n-dimensional array similar to NumPy's `ndarray`. MLX uses lazy evaluation: operations build a compute graph that is not executed until `eval()` is called. This enables automatic fusion and optimization before dispatch to the GPU.

### Mandelbrot Set Demo
Demonstrates the power of array computing: a scalar-at-a-time Swift loop computing z = z² + c for each pixel is replaced by a few lines of MLX Swift that apply the iteration across the entire grid at once on the GPU, achieving up to 10x speedup in fewer lines of code. Uses complex number support via `.asImaginary()`, `linspace`, `reshaped`, and elementwise comparison.

### Heat Distribution: Jacobi Iteration and SOR
Models steady-state heat in a 2D room. Jacobi iteration is implemented as a single `conv2d` call with a four-neighbor kernel, with boundary conditions applied via the `which` ternary function. Successive Over-Relaxation (SOR) uses a red/black checkerboard pattern and an omega relaxation parameter to converge in O(N) iterations instead of O(N²), demonstrated with a `checkerboard` helper and alternating masked updates.

### Curve Fitting with Automatic Differentiation
Given data points and a parametric function (quadratic), `grad(loss)` automatically derives the gradient function from the forward pass. A simple gradient descent loop then fits the curve with no manually written derivatives — a minimal demonstration of the same primitive used in neural network training.

### The Full MLX Toolkit
MLX includes: linear algebra (`matmul`, `norm`, `inv`, `svd`, `eigh`), FFTs, n-dimensional convolutions (`conv2d`), reductions (`sum`, `max`, `mean`), scans, advanced indexing, random number generation (`MLXRandom.normal`, `MLXRandom.uniform`), and optimizers (SGD, Adam, RMSprop). The Swift ecosystem packages are `mlx-swift`, `mlx-swift-lm` (LLM inference), and `mlx-swift-examples`.

## APIs & Frameworks

**MLX Swift** (`import MLX`)
- `MLXArray` — n-dimensional array; central type for all operations
- `MLXArray.zeros(like:)`, `MLXArray.zeros(_:dtype:)` — zero-filled arrays
- `linspace(_:_:count:)` — linearly spaced values
- `matmul(_:_:)` — matrix multiplication
- `norm(_:)` — vector/matrix norm
- `eval(_:)` — forces lazy evaluation / materializes computation graph
- `conv2d(_:_:padding:)` — 2D convolution **[new usage pattern]**
- `which(_:_:_:)` — elementwise ternary / masked selection
- `abs(_:)` — elementwise absolute value
- `exp(_:)` — elementwise exponential
- `mean(_:)`, `sum(_:)` — reductions
- `zeros(_:)` — zero-filled array by shape
- `grad(_:)` — returns gradient function of a scalar-valued function **[NEW for Swift]**
- `MLXArray.T` — transpose property
- `MLXArray.reshaped(_:_:)` — reshape
- `MLXArray.asImaginary()` — treat real array as imaginary component of complex numbers
- `.slice`, advanced indexing via subscript

**MLXRandom**
- `MLXRandom.normal(_:)` — Gaussian random array
- `MLXRandom.uniform(_:_:_:)` — uniform random array

**MLX Swift LM** (`mlx-swift-lm` package)
- High-level LLM inference helpers built on MLX Swift

**Optimizers (mlx-swift)**
- `SGD`, `Adam`, `RMSprop` — gradient-based parameter optimizers

**Related Resources**
- [MLX Swift on GitHub](https://github.com/ml-explore/mlx-swift)
- [MLX Swift LM on GitHub](https://github.com/ml-explore/mlx-swift-lm)
- [MLX Swift Examples](https://github.com/ml-explore/mlx-swift-examples)
- [MLX Python API](https://github.com/ml-explore/mlx)
- [MLX Framework site](https://mlx-framework.org)
- [MLX documentation](https://ml-explore.github.io/mlx/)

## Code Highlights

Mandelbrot set — whole-grid iteration in MLX Swift:
```swift
let x = linspace(-2.0, 0.5, count: w)
let y = linspace(-1.25, 1.25, count: h).reshaped(h, 1)
let c = x + y.asImaginary()
var z = MLXArray.zeros(like: c)
var counts = MLXArray.zeros(c.shape, dtype: .int16)
for _ in 0 ..< maxIterations {
    z = z * z + c
    counts = counts + (abs(z) .< 2)
}
```

Automatic differentiation for curve fitting:
```swift
func loss(_ θ: MLXArray) -> MLXArray { mean((f(θ) - y) ** 2) }
let gradLoss = grad(loss)
for _ in 0 ..< steps {
    let g = gradLoss(θ)
    θ = θ - learningRate * g
    eval(θ)
}
```

## Takeaways
- MLX Swift is the right choice when you need GPU-accelerated array math written in Swift — it eliminates cross-language friction compared to wrapping Python or C++ numerical libraries.
- Always call `eval()` after computations that need to be materialized; MLX's lazy graph enables automatic kernel fusion that improves GPU utilization.
- The `grad()` function makes gradient descent trivial to implement — no manual derivative math required; use it for any optimization or fine-tuning task.
- Start with `mlx-swift-examples` on GitHub for working examples including LLM inference, Stable Diffusion, and model training that you can run immediately.

---
_Source: WWDC26 Session 328 page (abstract, chapter summaries, code samples, and resource links)._
