# Safely manage pointers in Swift
**WWDC20 · Session 10167** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10167/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, watchOS 7, tvOS 14

## Overview
Swift's unsafe pointer types allow direct memory access when interoperating with C APIs or performing performance-sensitive work, but they come with strict usage rules that must be followed for memory safety. This session covers every Swift pointer type in detail, explains the rules governing their validity, and demonstrates common pitfalls — especially around pointer escaping and type punning — that lead to undefined behavior.

The session introduces the concept of "pointer provenance" and explains when pointers become invalid. It also covers the correct patterns for obtaining raw bytes of typed values, demonstrates `withUnsafeBytes` on `Data` and collections, and explains the `Unmanaged` API for bridging reference types without ARC.

## Key Topics

### Swift Pointer Type Hierarchy
- **`UnsafePointer<T>`** — typed, read-only pointer to `T`
- **`UnsafeMutablePointer<T>`** — typed, read-write pointer to `T`
- **`UnsafeRawPointer`** — untyped, read-only pointer to raw memory
- **`UnsafeMutableRawPointer`** — untyped, read-write pointer to raw memory
- **`UnsafeBufferPointer<T>`** / **`UnsafeMutableBufferPointer<T>`** — typed, bounded pointer (pointer + count)
- **`UnsafeRawBufferPointer`** / **`UnsafeMutableRawBufferPointer`** — untyped, bounded raw pointer
- **`OpaquePointer`** — pointer with no known type (for C structs that can't be expressed in Swift)

### Pointer Validity Rules
- A pointer is only valid for the duration of its source's lifetime
- Pointers must not escape the closure passed to `withUnsafePointer`, `withUnsafeBytes`, `withUnsafeMutableBytes`, etc.
- Storing a pointer from one of these closures into a variable and using it after the closure returns is **undefined behavior**
- Swift compiler does not guarantee that a variable's address remains stable across closures — always obtain a fresh pointer from `withUnsafePointer` for each use

### Typed vs. Raw Pointers
- Typed pointers (`UnsafePointer<T>`) assume memory is initialized with values of type `T`; accessing the wrong type is undefined behavior
- Raw pointers (`UnsafeRawPointer`) allow reading raw bytes; use `.load(as:)` and `.storeBytes(of:as:)` for type-punning safely
- Casting a typed pointer directly to a different typed pointer (pointer aliasing) violates Swift's memory model and produces undefined behavior

### withUnsafeBytes on Data and Collections
- `Data.withUnsafeBytes(_:)` provides an `UnsafeRawBufferPointer` to the bytes in the closure
- Use `.load(fromByteOffset:as:)` to read a value of a specific type at a given byte offset
- `Array.withUnsafeBufferPointer(_:)` / `Array.withUnsafeMutableBufferPointer(_:)` provide typed buffer pointers to array storage

### Common Pitfalls
- **Dangling pointer**: saving a pointer from a `with*` closure and using it later — the memory may have moved or been freed
- **Stale pointer from `inout`**: passing a value with `&` to a C function that retains the pointer for longer than the call — undefined behavior; use `withUnsafePointer(to:)` explicitly
- **Type aliasing**: casting `UnsafePointer<UInt8>` to `UnsafePointer<Float>` — use raw pointer `.load(as:)` instead
- **Implicit bridging pointers**: Swift allows passing a `String`, `Array`, or `Data` directly where a C pointer is expected (implicit bridging), but the pointer is only valid for the duration of that function call

### Unmanaged
- `Unmanaged<T>` bridges reference types across C APIs that use manual retain/release
- `.passRetained(_:)` — +1 retain; recipient must call `.release()` or `.takeRetainedValue()`
- `.passUnretained(_:)` — +0, caller must keep the object alive; recipient calls `.takeUnretainedValue()`
- `.fromOpaque(_:)` / `.toOpaque()` — bridge between `Unmanaged<T>` and `UnsafeRawPointer`

## APIs & Frameworks

- `UnsafePointer<T>` — typed read-only pointer
- `UnsafeMutablePointer<T>` — typed mutable pointer
- `UnsafeRawPointer` — raw read-only pointer; `.load(fromByteOffset:as:)`, `.loadUnaligned(fromByteOffset:as:)`
- `UnsafeMutableRawPointer` — raw mutable pointer; `.storeBytes(of:toByteOffset:as:)`, `.initializeMemory(as:repeating:count:)`, `.moveInitializeMemory(as:from:count:)`, `.deallocate()`
- `UnsafeBufferPointer<T>` — bounded typed read-only pointer; subscript, `baseAddress`, `count`
- `UnsafeMutableBufferPointer<T>` — bounded typed mutable pointer
- `UnsafeRawBufferPointer` — bounded raw read-only pointer; `load(fromByteOffset:as:)`, `bindMemory(to:)`
- `UnsafeMutableRawBufferPointer` — bounded raw mutable pointer
- `OpaquePointer` — forward-declared C pointer
- `withUnsafePointer(to:_:)` — obtain typed pointer to a value
- `withUnsafeMutablePointer(to:_:)` — obtain mutable typed pointer to a value
- `withUnsafeBytes(of:_:)` — obtain raw bytes of a value
- `withUnsafeMutableBytes(of:_:)` — obtain mutable raw bytes of a value
- `Array.withUnsafeBufferPointer(_:)` — typed buffer pointer to array storage
- `Array.withUnsafeMutableBufferPointer(_:)` — mutable typed buffer pointer to array storage
- `Data.withUnsafeBytes(_:)` — raw buffer pointer to Data bytes
- `UnsafeMutablePointer<T>.allocate(capacity:)` — heap allocation
- `UnsafeMutablePointer<T>.initialize(to:)` / `.initialize(repeating:count:)`
- `UnsafeMutablePointer<T>.deinitialize(count:)`
- `UnsafeMutablePointer<T>.deallocate()`
- `Unmanaged<T>` — manual ARC bridge
  - `.passRetained(_:)`, `.passUnretained(_:)`, `.release()`, `.retain()`
  - `.takeRetainedValue()`, `.takeUnretainedValue()`
  - `.fromOpaque(_:)`, `.toOpaque()`
- `UnsafePointer<T>.withMemoryRebound(to:capacity:_:)` — temporary type rebind
- `UnsafeRawPointer.bindMemory(to:capacity:)` — permanent type binding

## Code Highlights

Safe raw byte reading from Data:
```swift
data.withUnsafeBytes { rawBuffer in
    let value = rawBuffer.load(fromByteOffset: 4, as: UInt32.self)
    // use value here — do NOT let rawBuffer escape this closure
}
```

Heap-allocated typed memory:
```swift
let ptr = UnsafeMutablePointer<Int>.allocate(capacity: 10)
ptr.initialize(repeating: 0, count: 10)
// ... use ptr[0] through ptr[9] ...
ptr.deinitialize(count: 10)
ptr.deallocate()
```

Bridging a Swift object through a C callback via Unmanaged:
```swift
let ref = Unmanaged.passRetained(myObject).toOpaque()
// pass ref to C API as context pointer
// in callback:
let obj = Unmanaged<MyClass>.fromOpaque(ref).takeRetainedValue()
```

## Takeaways

- Never let an unsafe pointer escape the closure in which it was obtained — using it afterward is undefined behavior even if the address looks valid.
- Use raw pointers (`UnsafeRawPointer`, `.load(as:)`) for type punning rather than directly casting typed pointers; typed pointer aliasing violates Swift's memory model.
- Implicit pointer bridging (passing `&array`, `&string`, etc. to C functions) is only safe for the duration of that call; if the C API stores the pointer, use `withUnsafeBufferPointer` explicitly and keep the source alive.
- Use `Unmanaged` for bridging Swift objects through C APIs that require manual retain/release; always balance `.passRetained` with `.takeRetainedValue` or `.release`.

---
_Source: WWDC20 Session 10167 page (abstract, chapter summaries, code samples, and resource links)._
