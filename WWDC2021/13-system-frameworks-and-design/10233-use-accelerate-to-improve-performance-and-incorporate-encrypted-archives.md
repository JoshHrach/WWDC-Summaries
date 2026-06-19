# Use Accelerate to Improve Performance and Incorporate Encrypted Archives
**WWDC21 · Session 10233** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10233/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, watchOS 8, tvOS 15

## Overview
The Accelerate framework provides high-performance numerical computation across all Apple platforms, offering access to machine learning accelerators in Apple Silicon Macs and recent iPhone and iPad devices. This session covers the latest additions to Accelerate and its BNNS (Basic Neural Network Subroutines) library, including new layer types, activation functions, and optimizer improvements that expand the range of network architectures that can be accelerated on the CPU.

The session also highlights improvements to `simd.h`, Apple's library for small-vector and matrix computation. The new C++ template support makes it easier to write vectorized code without complicated type boilerplate, and several new mathematical functions have been added across all supported languages. A live demo shows how SIMD vectorization can achieve nearly 3x speedup over scalar loops.

Finally, the session introduces the new Apple Encrypted Archive format (AEA), an extension to Apple Archive that combines lossless compression with authenticated encryption and optional digital signatures. Multiple encryption profiles are available for different use cases, from simple signature-only verification to full public-key encryption, all designed and audited by Apple's security team.

## Key Topics

### BNNS — New Layers and Optimizer Support
BNNS now supports embedding, random fill, and quantization layer types. A new AdamW optimizer has been added, and existing layers have been enhanced with SiLU and HardSwish activation functions plus ternary select, multiply-add, and element-wise min/max arithmetic operations. Layer fusions between convolution/fully-connected and quantization layers, and between arithmetic and normalization layers, allow the output of one layer to feed directly into the next without intermediate memory writes. Gradient clipping improvements and AMSGrad support for Adam-based optimizers round out the ML additions.

### simd.h — C++ Template Improvements
New `simd_traits` and `get_traits` structs allow moving between scalar types, vector lengths, and concrete SIMD types without manual boilerplate. `Vector_t` and `Matrix_t` aliases simplify syntax. Templated versions of `make` and `convert` functions allow use in templated C++ code. New classification functions (`isfinite`, `isinf`, etc.) provide vector equivalents of libm scalar functions, along with gamma function calculations and SIMD matrix trace operations.

### Apple Encrypted Archive (AEA)
The new AEA format builds on Apple Archive to provide compression + authenticated encryption + digital signature in one secure package. Supported encryption profiles include: signature-only (no encryption), symmetric key encryption (with or without signature), password-based encryption, and full public-key encryption. Compression is always optional; data is always authenticated. The stream-based API supports both sequential and random access with multithreaded performance.

## APIs & Frameworks

- **Accelerate** framework (`import Accelerate` / `#include <Accelerate/Accelerate.h>`)
- **AppleArchive** framework (`import AppleArchive` / `#include <AppleArchive/AppleArchive.h>`) **[NEW in macOS 12]**
- **Compression** framework (`import Compression`)
- **simd** (`#include <simd/simd.h>`)
- `BNNS` — Basic Neural Network Subroutines (CPU ML primitives)
  - `BNNSLayerTypeEmbedding` **[NEW]**
  - `BNNSLayerTypeRandomFill` **[NEW]**
  - `BNNSLayerTypeQuantization` **[NEW]**
  - `BNNSOptimizerAdamW` **[NEW]**
  - `BNNSActivationFunctionSiLU` **[NEW]**
  - `BNNSActivationFunctionHardSwish` **[NEW]**
  - `BNNSArithmeticTernarySelect` **[NEW]**
  - `BNNSArithmeticMultiplyAdd` **[NEW]**
  - AMSGrad support for Adam-based optimizers **[NEW]**
  - Gradient clipping as standalone functions **[NEW]**
- `SIMD8<Float>`, `SIMD4<Float>`, etc. — fixed-size SIMD vector types
- `simd.exp(_:)` — vectorized exponential
- `SIMD.replacing(with:where:)` — conditional element replacement (mask-based blend)
- `simd_traits<T, N>` **[NEW]** — C++ struct for type/length → concrete SIMD type
- `get_traits<simd_type>` **[NEW]** — C++ struct for concrete SIMD type → generic traits
- `Vector_t<T, N>` alias **[NEW]** — simplified C++ SIMD vector alias
- `Matrix_t<T, C, R>` alias **[NEW]** — simplified C++ SIMD matrix alias
- `simd_make_*` / `simd_convert_*` templated variants **[NEW]**
- `simd_isfinite`, `simd_isinf` — vector classification functions **[NEW]**
- `simd_tgamma` — vector gamma function **[NEW]**
- `simd_trace` — SIMD matrix trace **[NEW]**
- `ArchiveEncryptionContext` **[NEW]** — configures AEA encryption profile and algorithm
  - `ArchiveEncryptionContext.Profile.hkdf_sha256_aesctr_hmac__symmetric__none` — symmetric profile, no signature
  - `ArchiveEncryptionContext.setSymmetricKey(_:)` **[NEW]**
  - `ArchiveEncryptionContext.setPassword(_:)` **[NEW]**
- `ArchiveByteStream.fileStream(path:mode:options:permissions:)` — file output stream
- `ArchiveByteStream.encryptionStream(writingTo:encryptionContext:)` **[NEW]** — AEA encryption stream
- `ArchiveStream.encodeStream(writingTo:)` — archive encode stream
- `ArchiveStream.writeDirectoryContents(archiveFrom:keySet:)` — recursively encodes directory
- `ArchiveHeader.FieldKeySet` — specifies which file attributes to archive (e.g. `"TYP,PAT,DAT,UID,GID,MOD"`)
- `SymmetricKey(size:)` — generates a secure symmetric key
- `FilePath` — type-safe file path representation
- `compression_tool` CLI — compresses/decompresses Apple Archive files
- `aea` CLI **[NEW]** — encrypts/decrypts Apple Encrypted Archive files
- `aa` CLI — handles full Apple Archive container

## Code Highlights

**Vectorized activation function using SIMD8:**
```swift
func swishharder_elementwise(_ x: SIMD8<Float>) -> SIMD8<Float> {
    let y = 2 * simd.exp(x)
    let a = y.replacing(with: 0, where: x .<= -.pi)
    let b = ((x + .pi) / 2).replacing(with: 1, where: .pi .<= x)
    return a * b
}
```

**Encrypting a directory with Apple Encrypted Archive:**
```swift
let context = ArchiveEncryptionContext(
    profile: .hkdf_sha256_aesctr_hmac__symmetric__none,
    compressionAlgorithm: .lzfse)
try context.setSymmetricKey(encryptionKey)

let fileStream = ArchiveByteStream.fileStream(path: destination, mode: .writeOnly,
    options: [.create, .truncate], permissions: FilePermissions(rawValue: 0o644))
let encryptionStream = ArchiveByteStream.encryptionStream(writingTo: fileStream,
    encryptionContext: context)
let encoderStream = ArchiveStream.encodeStream(writingTo: encryptionStream)

defer {
    try? encoderStream.close()
    try? encryptionStream.close()
    try? fileStream.close()
}

let fields = ArchiveHeader.FieldKeySet("TYP,PAT,DAT,UID,GID,MOD")!
try encoderStream.writeDirectoryContents(archiveFrom: source, keySet: fields)
```

## Takeaways

- BNNS adds embedding, random fill, and quantization layers plus AdamW and AMSGrad optimizer support, broadening the range of CPU-accelerated neural network architectures.
- `simd.h` gains C++ template traits and aliases that drastically reduce boilerplate when writing generic vectorized code; SIMD vectorization can yield ~3x speedup over scalar loops.
- Apple Encrypted Archive (new in macOS 12) provides a single-pass compression + authenticated encryption + digital signature container with multiple profiles, backed by state-of-the-art cryptography audited by Apple's security team.
- The AEA stream-based Swift API makes it straightforward to encrypt entire directory trees with just a few lines of code, with correct stream close ordering ensuring the archive is properly signed and sealed.

---
_Source: WWDC21 Session 10233 page (abstract, chapter summaries, code samples, and resource links)._
