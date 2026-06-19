# Using Accelerate and simd
**WWDC18 · Session 701** · [Watch](https://developer.apple.com/videos/play/wwdc2018/701/)

_Platforms:_ iOS 12, macOS Mojave 10.14, watchOS 5, tvOS 12

## Overview
A hands-on walkthrough of the Accelerate framework and its component libraries — vDSP, vImage, simd, BNNS, vForce, and the linear algebra libraries — with live interactive demos showing real-world signal processing, image processing, and 3D rotation use cases. The session is split between two engineers: Matthew covers vDSP (FFT/DCT for signal and image artifact removal) and simd (quaternion-based 3D rotations with Slerp/Spline interpolation); Luke covers vImage (color conversion, geometry, convolution, transform, morphology, dithering, and quantization for live camera effects).

The core value proposition: Accelerate provides thousands of hand-tuned math primitives that map directly to CPU vector hardware, yielding both higher performance and lower energy consumption without any code changes across platform and microarchitecture generations. The same binary achieves best-in-class performance from iPhone 5s through iPhone X — 68 gigaflops (single precision) on iPhone X on the LINPACK benchmark.

## Key Topics

### Accelerate Overview
- Thousands of low-level math primitives, hand-tuned to each CPU microarchitecture
- Available on iOS, macOS, watchOS, and tvOS — portable across all Apple platforms
- Performance and energy savings are automatic: same code benefits from each new microarchitecture without recompilation
- Key sub-libraries: **vDSP** (signal processing), **vImage** (image processing), **vForce** (vector transcendentals), **BNNS** (Basic Neural Network Subroutines), **Sparse/Dense LAPACK** (linear algebra), **simd** (vector/matrix programming aid), **Compression** (lossless compression)

### vDSP — Signal and Image Processing
- **Discrete Cosine Transform (DCT)**: analyze a signal in the frequency domain, threshold frequency components below noise floor to zero, reconstruct — removes broadband noise from audio without affecting signal-frequency spikes
- **2D FFT for image artifact removal**: transform image and a sample of the artifact (halftone screen) to frequency domain; create a mask from artifact frequencies; multiply out the artifact components (set to 0), keep content (multiply by 1); inverse FFT reconstructs a clean image
- DCT setup pattern: `vDSP_DCT_CreateSetup` → `vDSP_DCT_Execute` (type II forward); `vDSP_DCT_CreateSetup` → `vDSP_DCT_Execute` (type III inverse for reconstruction)
- 2D FFT: dimensions must be powers of 2; use `vDSP_fft2d_zrop` (out-of-place, split complex: real and imaginary in separate arrays); specify forward or inverse direction

### simd — Vector and Matrix Programming
- Declares vector and matrix types (`simd_float4`, `simd_float4x4`, etc.) that map directly to SIMD hardware registers; arithmetic operations (`+`, `-`, `*`, `/`) map to single instructions
- Far faster than scalar loops: averaging two `simd_float4` arrays takes one instruction vs. iterating element by element
- Available on all Apple platforms; consistent API across CPU architectures
- **Extensions**: dot product, cross product, clamp, normalize, length, mix
- **Transcendentals via vForce**: `simd_sin`, `simd_cos`, etc.
- **Quaternions** (`simd_quatf` / `simd_quatd`): represent 3D rotations with axis + angle; avoid gimbal lock; concatenate with multiplication (non-commutative like rotation matrices)
  - `simd_quaternion(angle:axis:)` — create quaternion from angle and axis
  - `simd_act(quaternion, vector)` — apply quaternion rotation to a vector
  - `simd_mul(q1, q2)` — combine rotations (order matters)
  - **Slerp** (`simd_slerp`, `simd_slerp_longest`) — spherical linear interpolation between two quaternions; shortest or longest arc
  - **Spline** (`simd_spline`) — smooth interpolation through more than two rotations using previous and next as context; produces smooth (non-sharp-cornered) paths

### vImage — Image Processing
- **Conversion functions**: move images between pixel formats (RGB ↔ YCbCr); camera captures YCbCr; display uses RGB; conversion enables optimal processing in the right color space for each task
- **Geometry**: `vImageScale_*` (enlarge/shrink with Lanczos high-quality resampling); `vImageRotate_*` (clockwise/counterclockwise)
- **Convolution**: blur effects; kernel size controls blur amount; `vImageTentConvolve_*`
- **Transform** (`vImageMatrixMultiply_*`): matrix multiply on per-pixel channel data; used for color saturation (remove bias, scale CbCr channels, restore bias — one function call)
- **Morphology**: `vImageErode_*` / `vImageDilate_*` — shrink/expand objects in the image; custom kernel shapes supported
- **Dithering**: convert 8-bit grayscale to 1-bit using Atkinson algorithm (`vImageConvert_Planar8toPlanar1`) — gray represented by dot density
- **Quantization / LUT**: `vImageTableLookUp_ARGB8888` — limit color palette for retro quantization effect

### vImage Integration with Camera (AVFoundation)
- Receive `CVImageBuffer` from camera delegate; lock base address for CPU access
- Wrap in `vImage_Buffer` struct (height, width, rowBytes, data pointer)
- Use `vImage_Buffer_Init` to allocate output buffers
- After processing: `vImageCreateCGImageFromBuffer` — creates `CGImage` from vImage buffer without copying pixel data; just wraps the existing memory
- Unlock pixel buffer base address after processing so camera can reuse it

## APIs & Frameworks

**Accelerate — vDSP**
- `vDSP_DCT_CreateSetup(_:_:_:)` — create DCT setup object; specifies length and type (II or III)
- `vDSP_DCT_Execute(_:_:_:)` — execute DCT forward or inverse transform
- `vDSP_fft2d_zrop(_:_:_:_:_:_:_:_:_:_:_:)` — 2D real FFT, out-of-place, split complex
- `vDSP_zvmags(_:_:_:_:_:)` — compute magnitudes of complex vector (for frequency thresholding)
- `vDSP_vclip(_:_:_:_:_:_:_:)` — clip values to threshold range (noise removal)

**Accelerate — simd**
- `simd_float2`, `simd_float3`, `simd_float4` — 2/3/4-component float vectors
- `simd_float4x4` — 4×4 float matrix
- `simd_quatf` / `simd_quatd` — single/double precision quaternion
- `simd_quaternion(_:_:)` — construct quaternion from angle and axis
- `simd_act(_:_:)` — apply quaternion rotation to vector
- `simd_mul(_:_:)` — multiply (compose) two quaternions
- `simd_slerp(_:_:_:)` — spherical linear interpolation (shortest arc)
- `simd_slerp_longest(_:_:_:)` — spherical linear interpolation (longest arc)
- `simd_spline(_:_:_:_:_:)` — smooth spline interpolation through sequence of rotations

**Accelerate — vImage**
- `vImage_Buffer` — struct wrapping image data (data, height, width, rowBytes)
- `vImage_Buffer_Init(_:_:_:_:_:)` — allocate and initialize output buffer
- `vImageScale_ARGB8888(_:_:_:_:)` — high-quality image scaling (Lanczos)
- `vImageRotate_ARGB8888(_:_:_:_:_:_:)` — image rotation
- `vImageTentConvolve_ARGB8888(_:_:_:_:_:_:_:_:_:)` — tent/blur convolution
- `vImageMatrixMultiply_ARGB8888(_:_:_:_:_:_:_:_:)` — per-pixel channel matrix multiply; color saturation, white balance
- `vImageErode_ARGB8888` / `vImageDilate_ARGB8888` — morphological operations
- `vImageConvert_ARGB8888toRGB888(_:_:_:)` — strip alpha channel
- `vImageTableLookUp_ARGB8888(_:_:_:_:_:_:_:_:)` — apply LUT to each channel
- `vImageCreateCGImageFromBuffer(_:_:_:_:_:_:)` — wrap vImage buffer as CGImage (no pixel copy)

**Accelerate — BNNS**
- Basic Neural Network Subroutines — inference support for neural networks; referenced but not demoed in this session

## Code Highlights

DCT-based noise removal:
```swift
import Accelerate

// Forward DCT (analysis)
let setupForward = vDSP_DCT_CreateSetup(nil, vDSP_Length(signal.count), .II)
var spectrum = [Float](repeating: 0, count: signal.count)
vDSP_DCT_Execute(setupForward!, signal, &spectrum)

// Zero out frequency components below threshold
vDSP_vthres(&spectrum, 1, &threshold, &spectrum, 1, vDSP_Length(spectrum.count))

// Inverse DCT (reconstruction)
let setupInverse = vDSP_DCT_CreateSetup(nil, vDSP_Length(signal.count), .III)
var reconstructed = [Float](repeating: 0, count: signal.count)
vDSP_DCT_Execute(setupInverse!, spectrum, &reconstructed)
```

simd vector average (vs. scalar loop):
```swift
import simd

// Scalar — slow
var result = [Float](repeating: 0, count: a.count)
for i in 0..<a.count { result[i] = (a[i] + b[i]) / 2 }

// simd — fast, single instruction
let va = simd_float4(a[0], a[1], a[2], a[3])
let vb = simd_float4(b[0], b[1], b[2], b[3])
let vr = (va + vb) / 2
```

Quaternion rotation and Slerp:
```swift
let axis = simd_float3(1, 0, 0)  // x-axis
let q1 = simd_quaternion(Float.pi / 3, axis)
let rotated = simd_act(q1, simd_float3(0, 0, 1))

// Interpolate between two rotations
let q2 = simd_quaternion(Float.pi / 3, simd_float3(0, 1, 0))
let combined = simd_mul(q1, q2)
let interpolated = simd_slerp(q1, combined, 0.5)
```

## Takeaways
- Use Accelerate instead of hand-rolled loops for any math-heavy operation — it is automatically tuned to each Apple CPU generation at link time, with no code changes required across architectures or platforms.
- vDSP DCT/FFT pairs are the right tool for any frequency-domain filtering task (noise removal, artifact removal, equalization) — the API is concise and the performance is competitive with hand-optimized assembly.
- `simd_quatf` quaternion composition and Slerp/Spline interpolation is the correct approach for smooth 3D rotation in games and graphics — avoid Euler angles and rotation matrices for interpolation tasks.
- vImage integrates directly with AVFoundation camera pipelines via `CVImageBuffer` and produces `CGImage` output without copying pixel data, making real-time video effects practical on-device.

---
_Source: WWDC18 Session 701 page (abstract, full transcript, and resource links)._
