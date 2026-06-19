# Explore Numerical Computing in Swift
**WWDC20 · Session 10217** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10217/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, watchOS 7, tvOS 14

## Overview
This session introduces the Swift Numerics open-source package and the new `Float16` type added to the Swift standard library. Swift Numerics provides the `Real` protocol — a unified abstraction over all standard floating-point types — enabling generic numeric algorithms that automatically work with `Float`, `Double`, `Float80`, and now `Float16` without source-level duplication.

The `Real` protocol is built from a layered hierarchy: `AdditiveArithmetic`, `SignedNumeric`, `FloatingPoint` (from the standard library), plus new protocols `AlgebraicField`, `ElementaryFunctions`, and `RealFunctions` from the Numerics package. This hierarchy enables clean, decomposed implementations of new numeric types. The session also covers the package's `Complex<T>` type, which has binary-compatible memory layout with C and C++ complex numbers and exposes significant performance advantages over C for multiplication and division.

`Float16` (IEEE 754 half-precision) is now a first-class Swift type on ARM platforms, conforming to all standard protocols including `Real`. It delivers up to 2x throughput in SIMD operations and is already well-supported by Apple CPUs starting with A11 Bionic.

## Key Topics

**The Real Protocol**
- Unifies all standard floating-point types under a single generic constraint
- Enables write-once generic algorithms for `Float`, `Double`, `Float80`, `Float16`
- `log`, `log(onePlus:)`, trig, exponential, and root functions available as static methods on any `Real`-conforming type

**Protocol Hierarchy (Swift Numerics)**
- `AdditiveArithmetic` — addition and subtraction (algebraic group)
- `SignedNumeric` — adds multiplication
- `FloatingPoint` — adds comparison, exponent/significand decomposition, `infinity`, `pi`, etc.
- `AlgebraicField` — adds division (four basic operations)
- `ElementaryFunctions` — core trig, logarithms, exponentials, roots, powers
- `RealFunctions` — gamma, error functions, additional trig variants
- `Real` — combines all of the above

**Complex Number Support**
- `Complex<T: Real>` generic struct with `real` and `imaginary` components
- C/C++-compatible memory layout (directly passable to BLAS/LAPACK via pointer)
- Polar coordinates via `length` (hypot) and `phase` (atan2)
- Swift's multiplication ~1.3x faster than C; division ~4x faster; division by constant ~10x faster

**Float16**
- New IEEE 754 half-precision type in Swift standard library
- Available on ARM-based platforms; simulated on older CPUs (same results, slower)
- Hardware support begins at Apple A11 Bionic
- 2 bytes vs 4 (Float) or 8 (Double) — doubles SIMD lane count and cache efficiency
- Range: ~10^-8 (min) to ~65,000 (max) — limited range requires care
- Conforms to `SIMDScalar`, `Real`, and all standard floating-point protocols
- BNNS convolution benchmark: Float at ~49 GFLOPS vs Float16 at ~119 GFLOPS

## APIs & Frameworks

### Swift Numerics Package (github.com/apple/swift-numerics)
- `Real` protocol **[NEW]** — universal floating-point constraint for generic algorithms
- `AlgebraicField` protocol **[NEW]** — adds division to `SignedNumeric`
- `ElementaryFunctions` protocol **[NEW]** — standard math functions as static members
- `RealFunctions` protocol **[NEW]** — extended math functions (gamma, erf, etc.)
- `Complex<T: Real>` **[NEW]** — fully featured complex number type
  - `init(_ real: T, _ imaginary: T)` — rectangular form constructor
  - `init(length: T, phase: T)` — polar form constructor
  - `var real: T` — real component
  - `var imaginary: T` — imaginary component
  - `var length: T` — Euclidean norm (`.hypot`)
  - `var phase: T` — phase angle (`.atan2`)
  - Conforms to `SignedNumeric` — `+`, `-`, `*` operators
  - `AlgebraicField` conformance — `/` operator
- Static math methods on `Real` types: `.log(_:)`, `.log(onePlus:)`, `.exp(_:)`, `.sin(_:)`, `.cos(_:)`, `.hypot(_:_:)`, `.atan2(y:x:)`, `.pow(_:_:)`, `.sqrt(_:)`, `.gamma(_:)`, `.erf(_:)`

### Swift Standard Library
- `Float16` **[NEW]** — IEEE 754 half-precision floating-point type (ARM platforms)
  - Conforms to: `FloatingPoint`, `Real`, `SIMDScalar`, `BinaryFloatingPoint`, `CustomStringConvertible`, `Hashable`, `Codable`
  - Max value: ~65,504; Min positive normal: ~6.1e-5; Min positive subnormal: ~6e-8

### Accelerate / BLAS Interop
- `cblas_dznrm2` — example of passing `[Complex<Double>]` directly via pointer to C BLAS
- `BNNS` — convolution layers benefit from `Float16` input (~2.4x throughput gain)

## Code Highlights

Generic logit function using `Real`:
```swift
import Numerics
func logit<T: Real>(_ p: T) -> T {
    .log(p) - .log(onePlus: -p)
}
// Works for Float, Double, Float80, Float16 unchanged
```

Creating and using Complex numbers:
```swift
let z = Complex(1.0, 2.0)          // Complex<Double> = 1 + 2i
let w = Complex(length: 1.0, phase: .pi / 4)  // polar form
```

Passing Complex array to BLAS:
```swift
import Numerics, Accelerate
let z = (0 ..< 100).map { Complex(length: 1.0, phase: Double.random(in: -.pi ... .pi)) }
let norm = cblas_dznrm2(z.count, &z, 1)
```

## Takeaways
- Use the `Real` protocol from Swift Numerics to write floating-point algorithms once and have them work across all standard types including the new `Float16`.
- `Complex<T>` is binary-compatible with C/C++ complex types, enabling zero-copy interop with performance-critical libraries like Accelerate.
- `Float16` on Apple Silicon nearly doubles throughput for neural network and SIMD workloads compared to `Float` — ideal for ML inference and image processing pipelines.
- Swift Numerics is an active open-source project on GitHub welcoming community contributions, including planned support for arbitrary-precision integers, shaped arrays, and decimal floating-point.

---
_Source: WWDC20 Session 10217 page (abstract, chapter summaries, code samples, and resource links)._
