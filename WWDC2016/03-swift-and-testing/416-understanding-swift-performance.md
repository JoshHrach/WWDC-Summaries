# Understanding Swift Performance
**WWDC16 · Session 416** · [Watch](https://developer.apple.com/videos/play/wwdc2016/416/)

_Platforms:_ iOS 10, macOS Sierra 10.12, tvOS 10, watchOS 3

## Overview
This advanced session from Kyle and Arnold on the Swift team provides a systematic, implementation-level understanding of how Swift's core language constructs — structs, classes, protocols, and generics — are compiled and executed. The goal is to equip developers with a mental model for predicting performance impact so they can narrow Swift's broad design space to the most efficient solution for a given problem. Three dimensions are analyzed for each construct: **memory allocation** (stack vs. heap), **reference counting overhead**, and **method dispatch** (static vs. dynamic).

The session is structured in two halves. The first covers value types (structs) versus reference types (classes). The second covers protocol types (using existential containers and witness tables) and generic code (parametric polymorphism with compiler specialization). Both halves include worked examples from a messaging application showing how to refactor towards safer, faster Swift.

## Key Topics

### Dimension 1: Allocation (Stack vs. Heap)
- **Stack**: contiguous memory; allocation/deallocation is a single integer increment/decrement of the stack pointer. Cost: trivial.
- **Heap**: dynamic lifetime; allocation requires searching a free-list data structure and locking for thread safety. Cost: significantly higher.
- **Struct**: stored inline on the stack (properties in-line). No heap allocation unless the struct contains reference-typed properties.
- **Class**: pointer stored on stack, but the object itself is always allocated on the heap. Class instances have two extra hidden words (type pointer + reference count) beyond the declared properties.
- **Practical example**: replacing a `String`-keyed dictionary with a `struct Balloon` key eliminates a heap allocation on every cache lookup, even on hits.

### Dimension 2: Reference Counting
- Swift tracks heap allocations using a reference count stored on the instance itself.
- Reference count operations are **atomic** (thread-safe), making them non-trivial despite being a simple integer increment/decrement.
- Classes: every copy of a reference adds a retain; every end-of-lifetime adds a release.
- Structs: no reference counting unless they contain reference-typed stored properties.
- A struct with two `String` or `class` properties incurs *more* reference counting than a single class because each reference is independently tracked.
- **Practical example**: replacing `String` uuid and mimeType fields in an `Attachment` struct with `UUID` (value type, new in Foundation) and a raw-value `enum` eliminates those two heap allocations and their reference counting overhead.

### Dimension 3: Method Dispatch
- **Static dispatch**: implementation determined at compile time; the compiler can inline, collapse chains, and eliminate call overhead entirely.
- **Dynamic dispatch**: implementation looked up at runtime via a virtual method table (V-Table) for classes; one level of indirection, but blocks compiler optimizations.
- `final` keyword on a class forces static dispatch. Whole Module Optimization lets the compiler opportunistically convert dynamic dispatches to static when it can prove no subclass exists.

### Protocol Types: Existential Container and Witness Tables
When a value of protocol type is stored in a variable or passed as a parameter, Swift uses an **Existential Container**:
- 3-word inline `valueBuffer` — small values (≤3 words) are stored directly in-line; large values are heap-allocated and the container stores a pointer.
- Pointer to **Value Witness Table** (VWT) — one per type; manages allocate, copy, destruct, deallocate operations on values.
- Pointer to **Protocol Witness Table** (PWT) — one per type-protocol conformance; stores pointers to the concrete implementations of each protocol requirement.

Performance implications:
- Small values in protocol types: no heap allocation, no reference counting → fast.
- Large values in protocol types: heap allocation on every copy → expensive.
- Mitigation: implement large-value types using **indirect storage with copy-on-write** (a `class` storage object + `isKnownUniquelyReferenced` check before mutation) — trades heap allocation for a cheaper reference count increment on most copies.

### Generic Code: Parametric Polymorphism and Specialization
- Generic functions receive the type's Value Witness Table and Protocol Witness Table as extra implicit arguments (no existential container needed).
- Local variables of generic type use a 3-word stack-allocated `valueBuffer` (same layout as the existential container's inline buffer).
- **Generic specialization**: the compiler creates type-specific versions of each generic function when it can observe both the concrete type and the function definition. Specialized code has the same performance as hand-written concrete code.
- **Whole Module Optimization** (enabled by default in Xcode 8): allows specialization across file boundaries by compiling all files in a module together.
- Unspecialized generic code: shares one implementation across call sites; no heap allocation for small values; correct but not as fast as specialized code.
- Generic stored properties with a concrete type binding: Swift stores values *inline* in the enclosing struct/class (no extra heap allocation) because the type cannot change at runtime.

## APIs & Frameworks

- **Swift** — core language constructs analyzed; no new stdlib APIs added in this session
- `struct` — value semantics, stack allocation, no implicit reference counting
- `class` — reference semantics, heap allocation, implicit reference counting, V-Table dispatch by default
- `final` keyword — disables dynamic dispatch on class methods; enables static dispatch and inlining
- `protocol` — protocol types use Existential Container + Protocol Witness Table (PWT) + Value Witness Table (VWT)
- Existential Container — 3-word inline valueBuffer + VWT reference + PWT reference (Swift runtime internal)
- Value Witness Table (VWT) — per-type runtime table: `allocate`, `copy`, `destruct`, `deallocate`, `projectBuffer`
- Protocol Witness Table (PWT) — per-conformance runtime table: pointers to concrete protocol requirement implementations
- Generic specialization — compiler transformation producing type-specific copies of generic functions (enabled by Whole Module Optimization across files)
- `isKnownUniquelyReferenced(_:)` — Foundation/Swift stdlib; checks reference count == 1 to implement copy-on-write for indirect storage
- `UUID` — Foundation value type (new in iOS 10 / macOS Sierra) storing 128 bits inline; replaces `String` for UUIDs with no heap allocation
- `enum` with raw `String` value — represents a closed set of string constants as a value type; no heap allocation, type-safe alternative to `String` for fixed domains
- Whole Module Optimization (`-whole-module-optimization`) — enabled by default in Xcode 8; allows cross-file generic specialization and dead-code elimination

## Code Highlights

Struct as dictionary key instead of String (eliminates heap allocation on cache hit):
```swift
// Before: heap-allocated String key
var cache: [String: UIImage] = [:]
let key = "\(color)-\(orientation)-\(tail)"

// After: struct key, stack-allocated, type-safe
struct BalloonAttributes: Hashable {
    var color: Color
    var orientation: Orientation
    var tail: Tail
}
var cache: [BalloonAttributes: UIImage] = [:]
```

Using `UUID` and raw-value `enum` to reduce reference counting in a struct:
```swift
// Before: 3 heap-allocated String properties
struct Attachment {
    let fileURL: URL
    let uuid: String     // heap-allocated
    let mimeType: String // heap-allocated
}

// After: only fileURL is heap-backed
enum MimeType: String {
    case jpeg = "image/jpeg"
    case png  = "image/png"
    case gif  = "image/gif"
}
struct Attachment {
    let fileURL: URL
    let uuid: UUID      // 128-bit value, no heap
    let mimeType: MimeType  // enum, no heap
}
```

Copy-on-write indirect storage for large protocol-type values:
```swift
final class LineStorage { var x1, y1, x2, y2: Double }
struct Line: Drawable {
    private var storage = LineStorage()
    var x1: Double {
        get { storage.x1 }
        set {
            if !isKnownUniquelyReferenced(&storage) { storage = LineStorage(storage) }
            storage.x1 = newValue
        }
    }
    func draw() { /* ... */ }
}
```

## Takeaways
- The three performance dimensions of any Swift abstraction are allocation location (stack vs. heap), reference counting frequency, and dispatch mechanism (static vs. dynamic). Paying for any of these without gaining corresponding benefit is wasted overhead.
- Structs are faster than classes by default on all three dimensions; use a class only when you need heap-based identity, shared mutable state, or Objective-C interoperability.
- A struct with more than one reference-typed stored property incurs more reference-counting overhead than a single class; prefer value-type stored properties (`UUID`, enums, other structs) where possible.
- Protocol types with large values incur heap allocations on every copy; implement indirect storage with copy-on-write to trade allocation cost for a cheaper reference-count increment on non-mutating copies.
- Whole Module Optimization (default in Xcode 8) unlocks generic specialization across file boundaries, enabling the compiler to produce code as fast as hand-written concrete implementations.

---
_Source: WWDC16 Session 416 page (abstract, transcript, and resource links)._
