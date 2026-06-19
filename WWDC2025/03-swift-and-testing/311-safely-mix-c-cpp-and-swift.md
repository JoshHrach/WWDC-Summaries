# Safely Mix C, C++, and Swift
**WWDC25 · Session 311** · [Watch](https://developer.apple.com/videos/play/wwdc2025/311/)

_Platforms:_ iOS 26, iPadOS 26, macOS Tahoe 26, tvOS 26, visionOS 26, watchOS 26

## Overview
As apps grow, they often combine Swift with older C and C++ codebases or third-party libraries. While Swift is safe by default, mixing languages can undermine those guarantees — raw pointers crossing the language boundary introduce the risk of buffer overflows and use-after-free bugs. This session shows how to systematically find and eliminate those risks using Swift 6.2's new Strict Memory Safety mode, new C/C++ pointer annotations, and updated Xcode bounds-safety features.

The session also introduces `Span` and `MutableSpan`, Swift 6.2's new safe pointer types that carry bounds and lifetime information, as the preferred replacement for `UnsafePointer` / `UnsafeMutablePointer` when calling annotated C and C++ functions.

## Key Topics

### Strict Memory Safety (Swift 6.2)
A new compiler mode (`SWIFT_STRICT_MEMORY_SAFETY = YES` in build settings) emits warnings wherever Swift code uses unsafe constructs — including hidden uses like `&array` that produce `UnsafeMutablePointer`. This helps surface every boundary with C/C++.

### Annotating Functions That Take Pointers
- `__counted_by(n)` — annotates a C pointer parameter to declare it points to `n` elements, enabling the compiler to import it as a Swift `Span` / `MutableSpan`.
- `__noescape` — declares the pointer parameter does not escape the function call, closing the lifetime gap needed for safe Span conversion.
- C++ Span (`std::span`) with `__noescape` can be treated as a Swift `MutableSpan`, eliminating unsafe boilerplate at the call site.

### Annotating Functions That Return Pointers
- `__lifetimebound` on a parameter tells the compiler that the returned pointer's lifetime is bounded by that parameter. This allows a C++ function returning `std::span` to be imported as a Swift `MutableSpan` with enforced lifetime.

### Importing Custom C++ Types Safely
- `SWIFT_NONESCAPABLE` — marks a C++ struct as a non-escapable type, preventing it from outliving the memory it views (analogous to Swift's `~Escapable` protocol).
- `SWIFT_SHARED_REFERENCE(retain_fn, release_fn)` — marks a C++ struct as reference-counted; Swift manages its lifetime automatically.
- `SWIFT_RETURNS_RETAINED` / `SWIFT_RETURNS_UNRETAINED` — annotate factory and accessor functions so Swift knows whether to retain the returned object.

### Making C/C++ Code Itself Safer
- **C++ Standard Library Hardening** — enables bounds checks on array subscripts in `std::span`, `std::vector`, etc. Set `ENFORCE_BOUNDS_SAFE_BUFFER_USAGE_IN_CPP = YES` in Xcode project settings.
- **Unsafe buffer usage errors** — Xcode can emit errors when raw pointers are used in C++, pushing developers to use `std::span` or standard containers instead.
- **Bounds Safety extension for C** — a Clang extension (enabled per-file in Xcode project settings) that requires `__counted_by` annotations throughout C code and inserts runtime bounds checks.

## APIs & Frameworks

**Swift 6.2**
- `Span<T>` **[NEW]** — safe, bounds-checked, non-escaping view into contiguous memory
- `MutableSpan<T>` **[NEW]** — mutable variant of `Span`
- Strict Memory Safety compiler mode **[NEW]** — `SWIFT_STRICT_MEMORY_SAFETY`

**C/C++ Annotations (Clang)**
- `__counted_by(expr)` **[NEW for Swift interop]** — bounds annotation on pointer parameters/return
- `__noescape` **[NEW for Swift interop]** — lifetime annotation: pointer does not escape function
- `__lifetimebound` **[NEW for Swift interop]** — return value lifetime bound to annotated parameter
- `SWIFT_NONESCAPABLE` **[NEW]** — imports C++ struct as `~Escapable` in Swift
- `SWIFT_SHARED_REFERENCE(retain, release)` **[NEW]** — imports C++ struct as Swift reference-counted type
- `SWIFT_RETURNS_RETAINED` / `SWIFT_RETURNS_UNRETAINED` **[NEW]** — ownership semantics for returned references

**Xcode Build Settings**
- `ENFORCE_BOUNDS_SAFE_BUFFER_USAGE_IN_CPP` **[NEW]** — enables C++ Standard Library Hardening + unsafe buffer errors
- `-fbounds-safety` C language extension **[NEW]** — runtime bounds checking for C

## Code Highlights

```c
// Annotate a C function so Swift can call it via MutableSpan
void invertImage(uint8_t *__counted_by(imageSize) imagePtr __noescape, size_t imageSize);
```

```swift
// Call site becomes safe — no raw pointer needed
var span = imageData.mutableSpan
invertImage(&span)
```

```cpp
// C++ Span parameter with noescape — Swift treats it as MutableSpan
void applyGrayscale(CxxSpanOfByte imageView __noescape);
```

```cpp
// Import custom view type as non-escapable
struct ImageView {
  std::span<uint8_t> pixelBytes;
  int width, height;
} SWIFT_NONESCAPABLE;
```

## Takeaways
- Enable Strict Memory Safety in security-sensitive Swift targets to surface every unsafe C/C++ call site.
- Add `__counted_by`, `__noescape`, and `__lifetimebound` annotations to C/C++ APIs that Swift calls; this allows the compiler to import them as safe `Span`-based functions with no call-site boilerplate.
- Use `SWIFT_NONESCAPABLE` and `SWIFT_SHARED_REFERENCE` to import custom C++ view and reference-counted types safely.
- Enable `ENFORCE_BOUNDS_SAFE_BUFFER_USAGE_IN_CPP` in Xcode to add runtime bounds checks to C++ standard containers and catch raw pointer usage at compile time.

---
_Source: WWDC25 Session 311 page (abstract, chapter summaries, code samples, and resource links)._
