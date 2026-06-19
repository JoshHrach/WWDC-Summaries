# What's New in Swift
**WWDC19 · Session 402** · [Watch](https://developer.apple.com/videos/play/wwdc2019/402/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15, watchOS 6, tvOS 13

## Overview
Swift 5 (released March 2019) and Swift 5.1 (released with Xcode 11) together deliver the most impactful set of language and runtime changes since Swift 1.0. The session covers ABI stability and module stability (which enable binary frameworks), runtime and code-size performance improvements, and a set of powerful new language features in Swift 5.1 that directly enable frameworks like SwiftUI, RealityKit, and Create ML.

The central theme is that Swift is now a platform for building APIs — with ABI stability allowing a shared Swift runtime in the OS, Swift Package Manager integration in Xcode, and new language features (opaque result types, property wrappers, function builders) that make expressive Swift-native APIs possible.

## Key Topics

### ABI Stability (Swift 5) **[NEW]**
- The Application Binary Interface (ABI) is now stable: code compiled with Swift 5 or later is binary compatible regardless of which Swift 5.x compiler produced it.
- A shared Swift runtime now ships in the OS (iOS 12.2+, macOS 10.14.4+, tvOS 12.2+, watchOS 5.2+, and all subsequent versions).
- Apps built with Swift 5 or later use the in-OS runtime; the bundled runtime in the app binary becomes inert. The App Store strips the bundled runtime when delivering to devices that have it, reducing download size.
- Launch time overhead from Swift runtime initialization drops to zero on supported OS versions (previously ~5% overhead).

### Module Stability and Swift Module Interface (Swift 5.1) **[NEW]**
- Module stability is a compile-time concept: Swift 5.1 introduces `.swiftinterface` (Swift module interface) files — a stable, source-like manifest of a framework's public API.
- Frameworks can now be distributed as binary frameworks consumable by any Swift 5+ compiler (not just the same compiler that built the framework).
- Together with ABI stability, this enables proper binary framework distribution (covered in depth in Session 416).

### Performance Improvements
- **Launch time**: Swift runtime initialization overhead eliminated when running on OS with shared runtime.
- **Code size**: Swift 5.1 compiler delivers up to 10% code size reduction overall; up to 15% for size-optimized builds.
- **Bridging performance**: NSDictionary ↔ Dictionary bridging now 1.6x faster; NSString operations on a bridged Swift string up to 15x faster.
- **String representation**: Changed from UTF-16 to UTF-8 internally.
  - Passing Swift strings to C APIs requires no allocation, copy, or transcoding (null-terminated UTF-8 is passed directly).
  - Small string optimization expanded to cover all Unicode characters (not just ASCII), packing strings of ≤15 UTF-8 code units directly into the string value.
  - SwiftNIO benchmark: 20% throughput increase for a web server doing text processing.

### Swift Package Manager in Xcode **[NEW]**
- Swift Package Manager is now integrated directly into Xcode's core app-development workflow.
- See Sessions 408 (Adopting Swift Packages in Xcode) and 410 (Creating Swift Packages) for full details.

### Open Source and Tooling
- Official Docker images for Swift hosted on Docker Hub (community-driven).
- SourceKit stress tester: new tool that fuzzes SourceKit with IDE queries to find crashes and regressions.
- Language Server Protocol (LSP) adoption underway for SourceKit: enables Swift code completion, jump-to-definition, and refactoring in any LSP-compatible editor.

### Language: Swift 5 Features
- **Implicit returns from single-expression functions** **[NEW]**: `return` keyword optional in single-expression functions, methods, and subscripts (previously only in closures).
- **Synthesized memberwise initializer defaults** (SE-0242) **[NEW]**: calling a synthesized memberwise initializer with a subset of arguments (using default values) now works.
- **SIMD types** **[NEW]**: `SIMD2`, `SIMD3`, `SIMD4`, `SIMD8`, `SIMD16`, `SIMD32`, `SIMD64` with standard integer/floating-point element types; pointwise operators (`.==`, `.>`, etc.); `SIMDMask` type.
- **String interpolation redesign** (SE-0228) **[NEW]**: `ExpressibleByStringInterpolation` protocol redesigned; up to 1.7x faster; allows custom types to define their own string interpolation behavior (e.g., `LocalizedStringKey` in SwiftUI for automatic localization of interpolated strings).

### Language: Swift 5.1 Features
- **Opaque result types** (SE-0244) **[NEW]**: `some Protocol` return type.
  - Guarantees that every call returns the same concrete type (unlike a bare protocol type).
  - Enables stronger type checking, associated-type requirements, and `Self` constraints.
  - Required by SwiftUI's `body` property (`var body: some View`).
  - Note: requires newer Swift runtime; guarded by availability checks for back-deployment.
- **Property wrappers** (SE-0258) **[NEW]**: custom `@propertyWrapper` attribute allows defining reusable property access patterns (lazy evaluation, UserDefaults storage, thread-local storage, etc.) applied to properties via custom attributes.
- **Function builders** (result builders, pre-SE-0289) **[NEW]**: `@_functionBuilder` attribute (internal/unofficial at time of WWDC19) enables embedded DSLs by transforming closure bodies containing sequences of expressions into structured values. Powers SwiftUI's `@ViewBuilder` and similar APIs.

## APIs & Frameworks

### Swift Standard Library
- `SIMD2<T>`, `SIMD3<T>`, `SIMD4<T>`, `SIMD8<T>`, `SIMD16<T>`, `SIMD32<T>`, `SIMD64<T>` **[NEW]** — fixed-size SIMD vector types
- `SIMDMask` **[NEW]** — result of pointwise comparisons on SIMD types
- `SIMD` protocol **[NEW]** — base protocol for all SIMD types
- `ExpressibleByStringInterpolation` **[REDESIGNED]** — protocol for custom string interpolation; `StringInterpolation` associated type with `appendLiteral(_:)` and `appendInterpolation` methods
- `DefaultStringInterpolation` **[NEW]** — the default string interpolation builder

### Swift Language Features (compiler)
- `some Protocol` — opaque result type **[NEW]** (Swift 5.1)
- `@propertyWrapper` — property wrapper attribute **[NEW]** (Swift 5.1)
- `wrappedValue` — required computed property in a property wrapper type
- `@_functionBuilder` / `@resultBuilder` — function builder attribute **[NEW]** (Swift 5.1, formal SE in later versions)
- `buildBlock(_:)` — required static method in a result builder type

### Binary Distribution
- `.swiftinterface` — Swift module interface file **[NEW]** (Swift 5.1)
- `XCFramework` — binary framework format for distribution across platforms **[NEW]** (Xcode 11)

## Code Highlights

Opaque result types in SwiftUI's body pattern:
```swift
protocol Shape { /* ... */ }

// Opaque return type: compiler knows the exact type, callers don't
func makeEightPointedStar() -> some Shape {
    Union(Square(), Rotated(Square(), by: .degrees(45)))
}
```

Property wrapper for UserDefaults:
```swift
@propertyWrapper
struct UserDefault<T> {
    let key: String
    let defaultValue: T
    var wrappedValue: T {
        get { UserDefaults.standard.object(forKey: key) as? T ?? defaultValue }
        set { UserDefaults.standard.set(newValue, forKey: key) }
    }
}

struct Settings {
    @UserDefault(key: "dark_mode", defaultValue: false) var darkMode: Bool
    @UserDefault(key: "font_size", defaultValue: 14) var fontSize: Int
}
```

SIMD pointwise operations:
```swift
let x = SIMD4<Float>(1, 2, 3, 4)
let y = SIMD4<Float>(4, 3, 2, 1)
let mask = x .> y  // SIMDMask: false, false, true, true
let negated = .!mask  // SIMDMask: true, true, false, false
```

Custom string interpolation (SwiftUI LocalizedStringKey pattern):
```swift
// Text("You have \(count) items") — SwiftUI uses this conformance:
extension LocalizedStringKey.StringInterpolation {
    mutating func appendInterpolation(_ count: Int) {
        appendLiteral("%d")
        arguments.append(count)
    }
}
```

## Takeaways
- ABI stability means a shared Swift runtime lives in the OS: apps are smaller to download and launch faster on supported OS versions — this is automatic with no code changes.
- Module stability (`.swiftinterface`) and XCFramework together unlock first-class binary Swift framework distribution — binary frameworks no longer require the same compiler version.
- Opaque result types (`some`) and property wrappers are the language foundations that make SwiftUI's declarative API design possible; function builders power `@ViewBuilder` and similar result-builder patterns.
- The UTF-8 string representation is a fundamental performance upgrade, especially for server-side Swift and any code that passes strings to C APIs.

---
_Source: WWDC19 Session 402 page (abstract, chapter summaries, code samples, and resource links)._
