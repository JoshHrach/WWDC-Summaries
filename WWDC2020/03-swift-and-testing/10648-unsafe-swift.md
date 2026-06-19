# Unsafe Swift
**WWDC20 · Session 10648** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10648/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, tvOS 14, watchOS 7

## Overview
This session defines what "unsafe" means in the context of Swift: an operation is unsafe if it can exhibit undefined behavior for at least some inputs that violate its documented contract. Safe APIs guarantee a defined response (typically a fatal error) for all inputs including invalid ones; unsafe APIs trust the caller to fulfill preconditions. The `unsafe` prefix in Swift is analogous to a hazard symbol—not a prohibition, but a warning that extra discipline is required.

The two primary reasons to reach for unsafe APIs are C/Objective-C interoperability (C functions take pointers; Swift needs a way to generate them) and fine-grained performance control (eliminating otherwise superfluous checks in hot paths). The session systematically walks through Swift's memory model, the unsafe pointer type family, implicit value-to-pointer conversions for calling C APIs, and the new (Swift 5.3) compiler warning for dangling pointers created via `UnsafeMutablePointer(&value)` escaping out of an implicit conversion.

Two new stdlib initializers—`Array.init(unsafeUninitializedCapacity:initializingWith:)` and `String.init(unsafeUninitializedCapacity:initializingUTF8With:)`—allow direct in-place initialization of collection storage, eliminating the intermediate manual-allocation pattern previously required when bridging C output buffers.

## Key Topics
- **Safe vs. unsafe APIs** — safe = fully defined behavior for all inputs; unsafe = undefined behavior for contract-violating inputs
- **`unsafelyUnwrapped`** — `Optional` property that skips the nil check in optimized builds; fatal in debug builds
- **Swift's flat memory model** — linear address space; stacks, heaps, binary, mapped resources share the same space
- **Unsafe pointer types** — raw abstraction over C pointers; no lifetime management, no bounds checking
- **Manual memory management** — `allocate(capacity:)`, `initialize(to:)` / `initialize(from:count:)`, `deinitialize(count:)`, `deallocate()`; dangling pointer after deallocation is UB
- **Unsafe buffer pointer types** — pair of (base address, count); subscript bounds-checked in debug builds
- **C interoperability** — C pointer types map to Swift `UnsafePointer<T>` / `UnsafeMutablePointer<T>` / raw variants
- **Implicit value-to-pointer conversions** — `[T]` → `UnsafePointer<T>`; `&array` → `UnsafeMutablePointer<T>`; `String` → `const char *`; `&value` → `UnsafePointer<T>` — valid only for the duration of the call
- **Dangling pointer warning** — Swift 5.3 compiler warns when `UnsafeMutablePointer(&value)` escapes the implicit conversion **[NEW]**
- **New uninitialized storage initializers** — `Array.init(unsafeUninitializedCapacity:initializingWith:)` and `String.init(unsafeUninitializedCapacity:initializingUTF8With:)` **[NEW in Swift 5.3]**
- **Best practices** — prefer closure-based APIs over escaping pointer values; use buffer pointers over plain pointers for regions; use Address Sanitizer to catch issues

## APIs & Frameworks

**Swift Standard Library — Unsafe Pointer Types**
- `UnsafePointer<Pointee>` — read-only typed pointer
- `UnsafeMutablePointer<Pointee>` — read-write typed pointer
  - `static func allocate(capacity:) -> UnsafeMutablePointer<Pointee>`
  - `func initialize(to:)`, `func initialize(from:count:)`, `func initialize(repeating:count:)`
  - `func deinitialize(count:) -> UnsafeMutableRawPointer`
  - `func deallocate()`
  - `var pointee: Pointee` — dereference
  - Pointer arithmetic via `+`, `-`, subscript
- `UnsafeRawPointer` / `UnsafeMutableRawPointer` — untyped raw byte pointers
- `UnsafeBufferPointer<Element>` / `UnsafeMutableBufferPointer<Element>` — typed buffer with (baseAddress, count)
  - Subscript bounds-checked in debug builds
- `UnsafeRawBufferPointer` / `UnsafeMutableRawBufferPointer` — raw buffer with byte count

**Optional**
- `Optional.unsafelyUnwrapped` — skips nil check in optimized builds; fatal in debug builds

**Implicit Conversions (C interop)**
- `[T]` → `UnsafePointer<T>` / pass `&array` → `UnsafeMutablePointer<T>` (duration of call only)
- `String` → `const char *` (`UnsafePointer<CChar>`) — includes NUL terminator
- `&value` → `UnsafePointer<T>` / `UnsafeMutablePointer<T>` (duration of call only)

**Closure-based Pointer Access**
- `withUnsafePointer(to:_:)` / `withUnsafeMutablePointer(to:_:)`
- `withUnsafeBytes(of:_:)` / `withUnsafeMutableBytes(of:_:)`
- `Array.withUnsafeBufferPointer(_:)` / `.withUnsafeMutableBufferPointer(_:)`
- `Array.withUnsafeBytes(_:)` / `.withUnsafeMutableBytes(_:)`
- `String.withCString(_:)` / `.withUTF8(_:)`
- `Sequence.withContiguousStorageIfAvailable(_:)`
- `MutableCollection.withContiguousMutableStorageIfAvailable(_:)`

**New Uninitialized Storage Initializers (Swift 5.3 / Xcode 12)**
- `Array.init(unsafeUninitializedCapacity:initializingWith:)` **[NEW]** — initialize array storage in-place; closure receives `UnsafeMutableBufferPointer` and inout initialized count
- `String.init(unsafeUninitializedCapacity:initializingUTF8With:)` **[NEW]** — initialize string UTF-8 storage in-place; closure returns number of initialized bytes (excluding NUL)

**Xcode Tools (referenced)**
- Address Sanitizer — catches use-after-free, out-of-bounds reads/writes, heap corruption at runtime

## Code Highlights

Manual allocation and deallocation (avoid this pattern when possible):
```swift
let ptr = UnsafeMutablePointer<Int>.allocate(capacity: 1)
ptr.initialize(to: 42)
print(ptr.pointee) // 42
ptr.deinitialize(count: 1)
ptr.deallocate()
// ptr.pointee = 23  // UNDEFINED BEHAVIOR — dangling pointer
```

Preferred: implicit array-to-pointer conversion for C call:
```swift
let values: [CInt] = [0, 2, 4, 6]
process_integers(values, values.count)  // pointer valid only during this call
```

Query system info using sysctl with implicit inout-to-pointer conversions:
```swift
import Darwin
func cachelineSize() -> Int {
    var query = [CTL_HW, HW_CACHELINE]
    var result: CInt = 0
    var resultSize = MemoryLayout<CInt>.size
    let r = sysctl(&query, CUnsignedInt(query.count), &result, &resultSize, nil, 0)
    precondition(r == 0)
    precondition(resultSize == MemoryLayout<CInt>.size)
    return Int(result)
}
```

New `String` uninitialized initializer to avoid intermediate buffer:
```swift
func kernelVersion() -> String {
    var query = [CTL_KERN, KERN_VERSION]
    var length = 0
    sysctl(&query, 2, nil, &length, nil, 0)
    return String(unsafeUninitializedCapacity: length) { buffer in
        var length = buffer.count
        sysctl(&query, 2, buffer.baseAddress, &length, nil, 0)
        return length - 1  // exclude NUL terminator
    }
}
```

Swift 5.3 dangling pointer warning (avoid this pattern):
```swift
var value = 42
let p = UnsafeMutablePointer(&value)  // BROKEN — Swift 5.3 now warns here
p.pointee += 1  // undefined behavior
```

## Takeaways
- "Unsafe" in Swift means the operation has undefined behavior for contract-violating inputs, not merely that it can crash—unsafe code is valid code when preconditions are met, but debugging violations can be extremely difficult.
- Prefer closure-based APIs (`withUnsafeBufferPointer`, `withUnsafeMutablePointer`, etc.) over extracting pointer values and using them later; the closure scope makes pointer lifetimes explicit and eliminates dangling pointer bugs.
- The new `Array.init(unsafeUninitializedCapacity:initializingWith:)` and `String.init(unsafeUninitializedCapacity:initializingUTF8With:)` initializers (Swift 5.3) eliminate manual allocation patterns when bridging C output buffers into Swift collections.
- Use unsafe buffer pointers (base address + count together) rather than bare pointers whenever working with memory regions; debug-build bounds checking provides a partial safety net.

---
_Source: WWDC20 Session 10648 page (abstract, chapter summaries, code samples, and resource links)._
