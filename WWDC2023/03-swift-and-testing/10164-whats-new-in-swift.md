# What's new in Swift
**WWDC23 · Session 10164** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10164/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, tvOS 17, watchOS 10, visionOS 1, Linux, Windows

## Overview
Swift 5.9 is one of the most significant language releases to date, introducing macros as a first-class extensibility mechanism, type parameter packs for variadic generics, non-copyable types for fine-grained resource ownership, and first-class C++ interoperability. The session covers the open Swift Evolution process, key language changes, a rewritten Swift-backed Foundation, concurrency improvements (custom actor executors), and a real-world case study using FoundationDB.

The open source Swift community now has a Language Steering Group (formed 2022) and an incoming Ecosystem Steering Group. Vision documents — starting with Swift Macros — provide a new way to coordinate multi-proposal features through the evolution process.

## Key Topics

**Expressivity improvements**
- `if`/`else` and `switch` statements can now be used as expressions — clean variable initialization without ternary chains or immediately-executed closures
- Result builder type checking significantly improved: faster type checking on valid and invalid code, more precise error diagnostics pointing to the actual mistake

**Type Parameter Packs (Variadic Generics)**
- New `each` and `repeat` keywords enable generic abstraction over the count of type parameters
- Eliminates the need for fixed-overload patterns (e.g., 1-through-6-argument overloads) with a single function
- `func evaluate<each Result>(_ :repeat Request<each Result>) -> (repeat each Result)` handles any argument count
- Used internally by Foundation's new `#Predicate` macro

**Swift Macros**
- Freestanding macros: `#macroName(...)` — produce code at call site; use `@freestanding(expression)` or `@freestanding(declaration)` roles
- Attached macros: `@MacroName` custom attribute syntax — augment declarations; roles: `member`, `peer`, `accessor`, `memberAttribute`, `conformance`
- Defined as compiler plugins (external macros), distributed via Swift packages
- Type-checked at use site; expanded code is viewable in Xcode via "Expand Macro"
- `@Observable` macro (multi-role): replaces `ObservableObject` + `@Published` + `ObservedObject` with a single annotation
- `#Predicate` macro: type-safe closure-based predicates used with collections, SwiftUI, SwiftData

**Swift Foundation (rewritten in Swift)**
- New open-source Swift implementation shared across Apple and non-Apple platforms
- New Swift-backed implementations of `Date`, `Calendar`, `Locale`, `AttributedString`, `JSONDecoder`, `JSONEncoder`
- Performance improvements in macOS Sonoma / iOS 17: Calendar date calculation 20%+ faster, `FormatStyle` date formatting 150% faster, JSON decoding 2–5x faster

**Ownership and Non-Copyable Types**
- New `~Copyable` syntax to suppress implicit copying for `struct` and `enum` types
- Non-copyable types can have `deinit` for deterministic resource cleanup without heap allocation
- `consuming` method attribute: calling a consuming method transfers ownership and prevents further use — compile-time safety
- `borrowing` (default): method borrows `self`, ownership returns to caller

**C++ Interoperability**
- Swift compiler maps C++ value types (with copy/move constructors, destructors) to Swift value types automatically
- C++ containers (`std::vector`, `std::map`) usable as Swift collections
- Swift APIs exposed to C++ via generated header — no `@objc` annotation required
- Reference-counted C++ smart pointers annotated for class-like ARC treatment in Swift
- CMake support: Swift + C++ in the same target; Swift CMake Examples repository on GitHub

**Swift Concurrency Updates**
- Custom actor executors: actors can implement `unownedExecutor` property with a type conforming to `SerialExecutor` (e.g., a `DispatchQueue`)
- `SerialExecutor` protocol: `checkIsolated()`, `asUnownedSerialExecutor()`, `enqueue(_:)` **[NEW]**
- `DispatchQueue` now conforms to `SerialExecutor`
- Abstract concurrency model adapts to cooperative single-threaded schedulers for constrained environments

## APIs & Frameworks

**Swift Language**
- `if`/`switch` as expressions **[NEW]**
- Type parameter packs: `each T`, `repeat each T` **[NEW]**
- `~Copyable` constraint on `struct`/`enum` **[NEW]**
- `consuming` / `borrowing` ownership modifiers on methods and parameters **[NEW]**
- `macro` keyword for declaring macros **[NEW]**
- `@freestanding(expression)`, `@freestanding(declaration)` macro roles **[NEW]**
- `@attached(member)`, `@attached(peer)`, `@attached(accessor)`, `@attached(memberAttribute)`, `@attached(conformance)` macro roles **[NEW]**
- `#externalMacro(module:type:)` **[NEW]** — links a macro declaration to its compiler plugin implementation
- `#Predicate<T> { ... }` **[NEW]** — type-safe predicate macro (Foundation)
- `Predicate<T>` type **[NEW]** (Foundation)
- `@Observable` macro **[NEW]** — replaces ObservableObject pattern
- `Observable` protocol **[NEW]**
- `@ObservationTracked` macro **[NEW]** (used internally by `@Observable`)
- `SerialExecutor` protocol **[NEW]** — custom actor executor conformance
- `UnownedSerialExecutor` **[NEW]**
- Actor `unownedExecutor: UnownedSerialExecutor` property for custom executor **[NEW]**
- `DispatchQueue: SerialExecutor` conformance **[NEW]**
- `withCheckedContinuation` / `withUnsafeContinuation` (existing, noted for C++ bridge)

**Foundation**
- `Date`, `Calendar`, `Locale`, `AttributedString` — new Swift implementations **[NEW underlying impl]**
- `JSONDecoder`, `JSONEncoder` — new Swift implementations, 2–5x faster **[NEW underlying impl]**
- `FormatStyle` date/time formatting — 150% performance improvement

## Code Highlights

`if` as expression:
```swift
let bullet =
    if isRoot && (count == 0 || !willExpand) { "" }
    else if count == 0 { "- " }
    else if maxDepth <= 0 { "▹ " }
    else { "▿ " }
```

Parameter packs:
```swift
func evaluate<each Result>(_ : repeat Request<each Result>) -> (repeat each Result)
```

Non-copyable file descriptor with consuming close:
```swift
struct FileDescriptor: ~Copyable {
    private var fd: Int32
    consuming func close() { ... } // last use enforced by compiler
    deinit { /* auto-cleanup */ }
}
```

Custom actor executor with DispatchQueue:
```swift
actor DatabaseConnection {
    let queue = DispatchQueue(label: "db")
    nonisolated var unownedExecutor: UnownedSerialExecutor { queue.asUnownedSerialExecutor() }
}
```

`@Observable` replacing ObservableObject:
```swift
@Observable class Person {
    var name: String
    var favoriteColor: Color
}
```

## Takeaways
- Adopt `@Observable` immediately — it drastically simplifies SwiftUI observation with a single macro replacing three annotations.
- Use `#Predicate` for type-safe filtering with SwiftData and Swift collections.
- Explore parameter packs when you currently ship fixed-arity overloads; the new syntax handles unlimited argument counts cleanly.
- Use `~Copyable` for resource handles (file descriptors, locks) where copying would be semantically wrong — you get compile-time safety instead of runtime crashes.

---
_Source: WWDC23 Session 10164 page (abstract, chapter summaries, code samples, and resource links)._
