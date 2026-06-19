# What's New in Swift
**WWDC24 · Session 10136** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10136/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia 15, visionOS 2, watchOS 11, tvOS 18, Linux, Windows, Embedded

## Overview
Swift turns 10 and ships Swift 6 — a new compiler and language mode that achieves data-race safety by default, turning all data-race issues into compile-time errors. The session surveys a decade of Swift milestones, then dives into four key 2024 advances: the Swift 6 language mode (complete concurrency checking by default), noncopyable types in generic contexts, Embedded Swift for highly constrained systems, and typed throws for precise error propagation.

On the ecosystem side: a fully static Linux SDK for cross-compilation from macOS to Linux, Foundation rewritten in Swift (open-source, cross-platform, now with Predicate and Regex support), a new Swift Testing framework, explicitly built modules for faster and more predictable Xcode builds, and Swift moving to the `github.com/swiftlang` organization.

## Key Topics

### Swift 6 Language Mode — Data-Race Safety
The Swift 6 language mode turns data-race safety checking — previously opt-in with the complete concurrency flag in Swift 5.10 — into the default, converting all data-race issues to compile errors. Swift 6 also refines what is considered a data race: non-Sendable values can now cross actor boundaries when the compiler can prove they can no longer be referenced from the sending side, eliminating false positives present in Swift 5.10's complete concurrency checking.

Adoption is incremental: the same compiler supports Swift 5 and Swift 6 language modes simultaneously, so modules can migrate independently. Only data-race safety is gated on the Swift 6 language mode; all other Swift 6 language features are available without enabling it.

### Synchronization Module — Atomics and Mutex
New low-level synchronization primitives in the `Synchronization` module:
- `Atomic<T>` — generic atomic, lock-free on supported platforms; stored in `let` properties; operations use explicit memory orderings (`.relaxed`, `.sequentiallyConsistent`, etc.)
- `Mutex` — stored in a `let` property; all accesses to protected state are via `withLock(_:)` closure for mutually exclusive access

### Noncopyable Types in Generic Contexts
Swift 6 removes the main restriction on noncopyable types: they can now be used with generics, protocols, and standard library types like `Optional`. A failable initializer `init?() -> Optional<T>` where `T: ~Copyable` is now expressible, enabling safe resource-owning types (e.g., a `File` struct that opens and closes a file descriptor) with ergonomic, safe APIs. The compiler inserts deinit calls at the right points and catches unintended copies.

### Embedded Swift
A new language subset and compilation model for highly constrained systems (microcontrollers, bare-metal, the Apple Secure Enclave Processor). Embedded Swift turns off features requiring runtime support (reflection, `any` types) and uses full generics specialization and static linking to produce standalone, minimal binaries. The result feels very close to standard Swift and supports ARM and RISC-V targets. Binary sizes as small as a few kilobytes.

### C++ Interoperability Improvements
Swift 6 expands bidirectional C++ interoperability: virtual methods, default arguments, move-only types, and key C++ standard library types can now be directly imported into Swift. C++ move-only types map to `~Copyable` in Swift; the Swift compiler auto-inserts move constructor calls and diagnoses accidental copies.

### Typed Throws
Functions can now declare their thrown error type: `throws(ErrorType)`. The error appears in catch blocks with its concrete type — no type-erasure boxing/unboxing and no dynamic cast required. Untyped `throws` equals `throws(any Error)`; a non-throwing function equals `throws(Never)`. Generic code can abstract over both throwing and non-throwing cases via the Failure type parameter.

### Foundation Open-Source and Cross-Platform
Foundation has been rewritten from legacy C/Objective-C into modern Swift and ships as the unified implementation across all Apple platforms and Linux. `Predicate` (introduced last year) is now available on all platforms in Swift 6 via swift-foundation, with new support for regular expressions.

### Swift Testing Framework
New `@Test` attribute, `#expect` macro, tag-based organization, and parameterized tests with arguments. Open-source, cross-platform, integrates with Xcode and VS Code. Designed to become the ecosystem's default testing solution.

### Explicitly Built Modules
Xcode can now build Swift module dependencies as explicit, parallel build steps rather than implicit sequential ones. Module builds appear in the build log, build in parallel, and are shared between the compiler and debugger — eliminating first-launch debugger pauses from rebuilding module files.

### Fully Static Linux SDK
`swift sdk install` installs a fully static Linux SDK for cross-compilation. `swift build --swift-sdk aarch64-swift-linux-musl` produces a statically linked binary that runs on any Linux machine without a Swift runtime installed.

## APIs & Frameworks

**Swift Language (Swift 6)**
- Swift 6 language mode **[NEW]** — data-race safety by default (compile-time errors for data races)
- Improved data-race checking — non-Sendable crossing actor boundaries with no residual references no longer errors **[NEW]**
- Noncopyable types (`~Copyable`) in generics and `Optional` **[NEW]**
- `throws(ErrorType)` — typed throws **[NEW]**
- C++ virtual methods import **[NEW]**
- C++ default arguments import **[NEW]**
- C++ move-only types → `~Copyable` mapping **[NEW]**
- C++ standard library type imports **[NEW]**
- Pack iteration (value parameter packs) **[NEW]**

**Synchronization Module (new)**
- `Atomic<T>` **[NEW]** — generic, lock-free atomic; explicit memory ordering (`wrappingAdd`, `load`, `store`, etc.)
- `Mutex` **[NEW]** — `withLock(_:)` for mutually exclusive access to protected state

**Embedded Swift**
- Embedded Swift compilation mode **[NEW]** — no Swift runtime; ARM and RISC-V targets
- Full generics specialization + static linking **[NEW]**

**Foundation (swift-foundation)**
- Unified Swift implementation shipping on all Apple platforms and Linux **[NEW]**
- `Predicate` — available on all platforms in Swift 6 **[NEW]**
- `Predicate` with regular expression support **[NEW]**

**Swift Testing**
- `@Test` attribute **[NEW]** — declare test functions with optional display name
- `#expect(_:)` macro **[NEW]** — assertion for arbitrary Swift expressions
- `@Test(.tags(...))` **[NEW]** — tag-based test organization
- `@Test(arguments:)` **[NEW]** — parameterized tests over multiple inputs

**Build System**
- Explicitly built modules in Xcode **[NEW]** — parallel module builds, shared debugger modules

**Swift SDK / Tooling**
- `swift sdk install` — install SDKs for cross-compilation **[NEW]**
- `swift build --swift-sdk aarch64-swift-linux-musl` — cross-compile to static Linux binary **[NEW]**
- Swift moves to `github.com/swiftlang` organization **[NEW]**
- SourceKit-LSP (VS Code, Neovim, Emacs integration) — extended support
- Fedora and Debian added as officially supported Linux platforms **[NEW]**

## Code Highlights

Noncopyable type with failable initializer (Swift 6):
```swift
struct File: ~Copyable {
    private let fd: CInt
    init?(name: String) {
        guard let fd = open(name) else { return nil }
        self.fd = fd
    }
    func write(buffer: [UInt8]) { /* ... */ }
    deinit { close(fd) }
}
```

Typed throws:
```swift
func parse(string: String) throws(IntegerParseError) -> Int {
    for index in string.indices {
        throw IntegerParseError.nonDigitCharacter(string, index: index)
    }
    return 0
}
do {
    let value = try parse(string: "1+234")
} catch {
    // error is IntegerParseError — no cast needed
}
```

Atomic counter:
```swift
import Synchronization
let counter = Atomic<Int>(0)
DispatchQueue.concurrentPerform(iterations: 10) { _ in
    for _ in 0 ..< 1_000_000 {
        counter.wrappingAdd(1, ordering: .relaxed)
    }
}
```

Mutex for protected state:
```swift
import Synchronization
final class LockingResourceManager: Sendable {
    let cache = Mutex<[String: Resource]>([:])
    func save(_ resource: Resource, as key: String) {
        cache.withLock { $0[key] = resource }
    }
}
```

Cross-compile to Linux:
```bash
swift sdk install ~/preview-static-swift-linux-0.0.1.tar.gz
swift build --swift-sdk aarch64-swift-linux-musl
```

## Takeaways
- Enable the Swift 6 language mode module-by-module — start with complete concurrency warnings in Swift 5 mode to surface migration work before switching modes; the compiler supports both simultaneously.
- Use typed throws for internal functions and error-propagation wrappers in constrained environments; keep untyped `throws` for public APIs where the error type may evolve.
- The new `Synchronization` module (`Atomic`, `Mutex`) provides safe, high-performance low-level synchronization without reaching for `DispatchSemaphore` or `os_unfair_lock`.
- Adopt Embedded Swift for bare-metal and microcontroller targets — the subset is close to full Swift, so existing Swift skills transfer with minimal adjustment.

---
_Source: WWDC24 Session 10136 page (abstract, chapter summaries, code samples, and resource links)._
