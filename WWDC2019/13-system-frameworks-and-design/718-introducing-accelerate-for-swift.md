# Introducing Accelerate for Swift
**WWDC19 · Session 718** · [Watch](https://developer.apple.com/videos/play/wwdc2019/718/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15, tvOS 13, watchOS 6

## Overview
The Accelerate framework provides hundreds of highly optimized computational functions that leverage SIMD (Single Instruction Multiple Data) hardware to deliver exceptional performance and energy efficiency across all Apple platforms. Prior to this session, Accelerate's libraries were powerful but their C-style interfaces were unfriendly to Swift developers. This session introduces a new Swift overlay with clean, idiomatic Swift APIs for four of Accelerate's core libraries: vDSP, vForce, Quadrature, and vImage.

The new APIs replace pointer-based function calls with familiar Swift types like arrays and array slices, use Swift enumerations and option sets for flags and configuration, and surface errors as thrown Swift exceptions rather than integer codes. Both in-place (pre-allocated results) and self-allocating variants are provided, giving developers flexibility between maximum performance and code simplicity.

A live Linpack benchmark demonstrated that using Accelerate's hand-tuned BLAS/LAPACK routines yields over 24x faster performance than unoptimized code running the same algorithm on the same iPhone XS hardware, with corresponding energy savings that directly improve battery life.

## Key Topics

**vDSP — Digital Signal Processing**
Covers element-wise arithmetic on large arrays, type conversion, Discrete Fourier Transforms (DFT), and biquadratic (biquad) filtering. The new API exposes functions through a `vDSP` namespace, accepts `Array` and `ArraySlice` instead of raw pointers, and eliminates explicit count parameters. Self-allocating variants use Swift's uninitialized buffer initialization for ergonomic one-liners.

**vForce — Transcendental Math**
Provides vectorized versions of `sqrt`, `exp`, `log`, trig, and other functions not included in vDSP. The new API matches the vDSP style; `vForce.sqrt(_:)` demonstrated up to 10x speedup over scalar `map` with `Foundation.sqrt`.

**Quadrature — Numerical Integration**
Computes definite integrals of functions over finite or infinite intervals. The new API replaces C function pointer integrands with Swift trailing closures (enabling natural capture of outer values), and represents integration algorithms as enumerations with associated values rather than raw option structs.

**vImage — Image Processing**
A rich library for alpha blending, format conversion, histogram operations, convolution, geometry, and morphology. The new API adds a single throwable initializer to create a `vImage_Buffer` from a `CGImage`, a `createCGImage()` method on the buffer, a static `make` factory on the any-to-any converter, and a `CVImageFormat` type wrapping Core Video pixel buffer format descriptions.

**Linpack Benchmark & GEMM**
The session benchmarked SGEMM (single-precision general matrix multiply) — the core of BLAS, which underpins LAPACK, which underpins Linpack. Accelerate's hand-tuned implementation reached ~125 GFLOPS on iPhone XS, nearly 2.5x faster than the Eigen library at ~51 GFLOPS, illustrating that framework-level tuning to micro-architecture consistently wins over generic optimized libraries.

## APIs & Frameworks

**Accelerate / vDSP** **[NEW Swift overlay]**
- `vDSP` namespace
- `vDSP.add(_:_:result:)` / `vDSP.add(_:_:)` (self-allocating)
- `vDSP.subtract(_:_:result:)` / `vDSP.subtract(_:_:)`
- `vDSP.multiply(_:_:result:)` / `vDSP.multiply(_:_:)`
- `vDSP.convert(_:to:rounding:)` — type conversion with `vDSP.RoundingMode` enum **[NEW]**
- `vDSP.DFT` — Discrete Fourier Transform setup object **[NEW]**
  - `vDSP.DFT.init(count:direction:)` **[NEW]**
  - `vDSP.DFT.transform(real:imaginary:)` **[NEW]**
- `vDSP.Biquad` — biquadratic filter **[NEW]**
  - `vDSP.Biquad.init(coefficients:channelCount:sectionCount:)` **[NEW]**
  - `vDSP.Biquad.apply(input:)` **[NEW]**

**Accelerate / vForce** **[NEW Swift overlay]**
- `vForce` namespace **[NEW]**
- `vForce.sqrt(_:result:)` / `vForce.sqrt(_:)` **[NEW]**
- `vForce.exp(_:)`, `vForce.log(_:)`, trig variants **[NEW]**

**Accelerate / Quadrature** **[NEW Swift overlay]**
- `Quadrature` struct **[NEW]**
- `Quadrature.IntegrationRule` enum (e.g., `.gaussKronrod`, `.globallyAdaptive`) **[NEW]**
- `Quadrature.integrate(over:integrand:)` with trailing Swift closure **[NEW]**

**Accelerate / vImage** **[NEW Swift overlay]**
- `vImage_Buffer.init(cgImage:format:)` throwable initializer **[NEW]**
- `vImage_Buffer.createCGImage(format:)` **[NEW]**
- `CGImageFormat.init(cgImage:)` **[NEW]**
- `vImageConverter.make(sourceFormat:destinationFormat:)` static factory **[NEW]**
- `vImageConverter.convert(source:destination:)` **[NEW]**
- `CVImageFormat` — wraps `CVPixelBuffer` format descriptions **[NEW]**
- `CVImageFormat.make(pixelBuffer:)` **[NEW]**
- `CVImageFormat.channelCount` property **[NEW]**
- `vImage.Options` as `OptionSet` (replacing raw flags) **[NEW]**

**Underlying Libraries**
- `vDSP` (classic C API)
- `vForce` (classic C API)
- `Quadrature` (classic C API)
- `vImage` (classic C API)
- `BLAS` / SGEMM (via Accelerate)
- `LAPACK` / matrix factorization and backsolve

## Code Highlights

Self-allocating vDSP arithmetic (new Swift API):
```swift
let result = vDSP.add(a, b)
let product = vDSP.multiply(vDSP.subtract(a, b), vDSP.subtract(c, d))
```

Type conversion with rounding mode:
```swift
let integers: [UInt16] = vDSP.convert(doubles, to: UInt16.self, rounding: .towardZero)
```

Fourier transform with new API:
```swift
let dft = try vDSP.DFT(count: n, direction: .forward)
let (real, imaginary) = dft.transform(real: inputReal, imaginary: inputImaginary)
```

Biquad filter creation:
```swift
let biquad = try vDSP.Biquad(coefficients: [b0, b1, b2, a1, a2],
                               channelCount: 1, sectionCount: 1)
let output = biquad.apply(input: signal)
```

Quadrature with trailing closure integrand:
```swift
let quadrature = Quadrature(integrator: .globallyAdaptive(pointsPerInterval: .fifteenPointKronrod,
                                                            maxIntervals: 10))
let result = try quadrature.integrate(over: 0.0...1.0) { x in sqrt(1 - x * x) }
```

vImage buffer from CGImage:
```swift
let format = try CGImageFormat(cgImage: cgImage)
let buffer = try vImage_Buffer(cgImage: cgImage, format: format)
let outputImage = try buffer.createCGImage(format: format)
```

## Takeaways
- The new Swift overlay for Accelerate replaces opaque C pointer APIs with readable, type-safe Swift interfaces across vDSP, vForce, Quadrature, and vImage — zero performance penalty for using the Swift wrappers.
- SIMD vectorization delivers 3–10x speedups over scalar loops for common operations (arithmetic, type conversion, sqrt) and 24x on real-world workloads like Linpack.
- Both pre-allocated (highest performance, buffer reuse) and self-allocating (convenience) function variants are provided to suit different coding styles.
- Accelerate's performance advantage is not just speed — the energy savings from fewer CPU cycles directly extend battery life on iOS, watchOS, and tvOS devices.

---
_Source: WWDC19 Session 718 page (abstract, chapter summaries, code samples, and resource links)._
