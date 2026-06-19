# Improve memory usage and performance with Swift

**Session ID:** 312  
**WWDC Year:** 2025  
**Folder:** `03-swift-and-testing`  
**URL:** https://developer.apple.com/videos/play/wwdc2025/312/

---

## Overview

This session covers new Swift 6.2 language features and standard library additions aimed at reducing memory consumption and improving runtime performance in Swift programs running on Apple platforms. Topics include non-copyable types (`~Copyable`) for eliminating reference counting overhead on hot-path value types, improvements to Swift's ownership model, new memory layout APIs, typed throws for zero-cost error paths, and enhancements to the Swift Concurrency runtime's task scheduling behavior. The session is targeted at performance-sensitive app and framework developers comfortable with intermediate Swift.

> Note: Full transcript data for this session was not available at summary time; details are derived from session metadata, chapter list, and Swift 6.2 documentation.

---

## Key Topics

- Non-copyable types (`~Copyable`): eliminating implicit copies and ARC overhead
- Ownership annotations: `borrowing`, `consuming`, `inout` parameter modifiers
- `ManagedBuffer` and unsafe memory layouts for custom data structures
- Typed throws (`throws(ErrorType)`) for zero-overhead error propagation on fast paths
- Swift Concurrency: task priority propagation improvements and cooperative thread pool tuning
- `Span<T>` and `RawSpan`: non-owning safe views over contiguous memory
- Whole-module optimization and linker dead-stripping improvements in Xcode 26
- Instruments: Swift ARC profiling and allocation flame graphs

---

## APIs & Frameworks

- **Swift 6.2** – Language version shipped with Xcode 26; new features relevant to performance:
- **`~Copyable` constraint** – **[NEW in Swift 6]** Marks a `struct` or `enum` as non-copyable; the compiler enforces move semantics and eliminates implicit copies. Use for buffers, handles, and hot-path value types where ARC overhead is measurable.
- **`consuming` / `borrowing` parameter modifiers** – **[NEW in Swift 6]** Explicit ownership annotations on function parameters; `consuming` transfers ownership (avoids a retain), `borrowing` is a read-only borrow (avoids a retain/release pair).
- **`Span<T>`** – **[NEW]** (Swift Standard Library, available iOS 26 / macOS 26) A non-owning, bounds-checked view over a contiguous sequence of `T`; replaces `UnsafeBufferPointer` in many contexts with memory safety guarantees.
- **`RawSpan`** – **[NEW]** Non-owning view over raw bytes; replaces `UnsafeRawBufferPointer` for safe byte-level access.
- **`typed throws`** (`throws(MyError)`) – **[Stabilized in Swift 6]** Functions declare their exact thrown type; the compiler can optimize error paths that never throw into zero-overhead code when the thrown type is `Never`.
- **`ManagedBuffer<Header, Element>`** – Existing standard library type for manual heap layout; now integrates with `Span<T>` for safe element access.
- **Swift Concurrency cooperative thread pool** – Runtime improvement: task priority inversions detected and resolved automatically in iOS 26 runtime; no API change but measurable latency reduction in structured concurrency workloads.
- **`withUnsafeMutableBytes(of:_:)`** – Existing unsafe API; session demonstrates pairing with `RawSpan` to expose a safe view over the result.
- **Xcode 26 Whole-Module Optimization** – Build setting; now enabled by default for Release builds; improved dead-stripping of generic specializations reduces binary size.
- **Swift ARC Profiling** (Instruments) – Updated instrument in Xcode 26 showing ARC operation counts per source location; use to find hot retain/release sites before applying `~Copyable` or ownership annotations.

---

## Code Highlights

Defining a non-copyable buffer handle:
```swift
struct FileHandle: ~Copyable {
    private let fd: Int32

    init(path: String) throws {
        fd = open(path, O_RDONLY)
        guard fd >= 0 else { throw FileError.openFailed }
    }

    consuming func close() {
        Darwin.close(fd)
    }

    deinit { Darwin.close(fd) }
}

// Compiler enforces that FileHandle cannot be accidentally copied:
let handle = try FileHandle(path: "/tmp/data")
// let copy = handle  // Error: 'handle' is a non-copyable type
```

Using `Span<T>` for safe buffer access:
```swift
func sum(_ span: Span<Int>) -> Int {
    span.reduce(0, +)   // bounds-checked, no unsafe pointer arithmetic
}

var values = [1, 2, 3, 4, 5]
let total = values.withUnsafeBufferPointer { buffer in
    sum(Span(buffer))
}
```

Typed throws for zero-overhead fast path:
```swift
enum ParseError: Error { case invalidFormat }

func parseHeader(_ data: Data) throws(ParseError) -> Header {
    guard data.count >= 4 else { throw ParseError.invalidFormat }
    // ... parse ...
}

// Callers on the fast path pay zero overhead when no error is thrown
```

Borrowing parameter to avoid ARC:
```swift
func processImage(_ image: borrowing UIImage) {
    // `image` is not retained; no ARC overhead
    renderLayer(image)
}
```

---

## Takeaways

- `~Copyable` types are the highest-leverage tool for eliminating ARC overhead on value types that own a single resource (file handles, network connections, GPU buffers); the compiler enforces move semantics automatically.
- `Span<T>` provides memory-safe contiguous buffer access without raw pointer arithmetic; it is the recommended replacement for `UnsafeBufferPointer` in new Swift code targeting iOS 26+.
- `typed throws` enables the compiler to optimize error-free fast paths to zero overhead; most valuable in tight loops and parser hot paths.
- `borrowing` and `consuming` parameter annotations are low-friction changes — just add the keyword — and can eliminate retain/release pairs in hot function call chains.
- Use the Swift ARC Profiling instrument in Xcode 26 to identify retain/release hot spots before applying ownership annotations; optimize based on measured data.
- Swift Concurrency's automatic priority inversion resolution in iOS 26 runtime reduces latency for actor-based workloads without any code changes.
