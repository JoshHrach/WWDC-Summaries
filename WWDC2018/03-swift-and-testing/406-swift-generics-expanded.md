# Swift Generics (Expanded)
**WWDC18 · Session 406** · [Watch](https://developer.apple.com/videos/play/wwdc2018/406/)

_Platforms:_ iOS 12, macOS Mojave 10.14, watchOS 5, tvOS 12 (Swift 4.2)

## Overview
This is an expanded version of the WWDC 2018 Swift generics session, including a brand-new section on recursive constraints not covered in the original presentation. The session explains the design philosophy behind Swift's generics system — providing flexible, reusable abstractions while preserving static type information — and walks through the key language features that make this possible.

The session proceeds from first principles, building up the generics design space from parameterized types and protocol constraints through protocol inheritance, conditional conformances, and finally to recursive constraints. The running example throughout is a `Collection`-like buffer type, mirroring how the Swift standard library itself was designed.

## Key Topics

### Parameterized Types and Type Constraints
- Generic functions and types use type parameters (`<T>`) to work over any type
- `where` clauses and `: Protocol` constraints restrict which types can be used
- The compiler uses constraint information to determine what operations are available on a type parameter
- A constrained generic is both more usable (more operations allowed) and more expressive than an unconstrained one

### Designing Protocols
- Protocols define a set of requirements (methods, properties, associated types) that conforming types must satisfy
- Protocol-oriented design: start with a minimal protocol that captures the core abstraction, then specialize via inheritance
- `associatedtype` — a placeholder type within a protocol that each conforming type specifies concretely
- Using associated types enables generic algorithms that work across all conforming types without losing type safety

### Protocol Inheritance
- Protocols can inherit from other protocols, adding requirements
- Enables a hierarchy of capabilities: base protocol for basic functionality, derived protocols for richer capabilities
- A generic function can accept the base protocol and still work, while a function that needs extra capabilities can require the derived protocol
- Example pattern: `Collection` inherits from `Sequence`, adding index-based access

### Conditional Conformances (Swift 4.2) **[NEW]**
- A generic type can conditionally conform to a protocol only when its type parameter also conforms to that protocol
- Syntax: `extension Array: Equatable where Element: Equatable { ... }`
- Enables composable conformances: if each piece is `Equatable`, the whole is `Equatable`
- Works for standard library types: `Array`, `Optional`, `Dictionary`, slices all gain conditional conformances in Swift 4.2
- Conditional conformances are checked at compile time using the full constraint system

### Class Inheritance and Generics
- Generic code can work with class hierarchies, but the interaction requires care
- Covariance: a `Container<Subclass>` is not a subtype of `Container<Superclass>` in Swift (unlike Java/C# arrays) — this is intentional to preserve type safety
- Use `AnyObject` constraints or existentials (`some Protocol`) when you need to mix class types
- Protocol conformances on classes can be inherited by subclasses if the conformance is on the class, not an extension

### Recursive Constraints **[NEW — expanded section]**
- A protocol can use itself recursively in its associated type constraints
- Classic example: `Comparable`'s `Indices` type in a `Collection` is itself a `Collection`
- Enables deeply nested generic algorithms that work with the full recursive structure of types
- The compiler resolves recursive constraints through a constraint solving algorithm, not infinite loops
- Practical use: defining protocols where the `SubSequence` of a `Collection` is also a `Collection` conforming to the same protocol

### Designing Generic APIs
- Start with concrete types, extract the shared interface into a protocol, then parameterize
- Avoid over-constraining: only add requirements that the algorithm actually needs
- Use protocol composition (`Protocol1 & Protocol2`) when multiple orthogonal requirements are needed
- Type erasure (`AnyCollection`, `AnySequence`) when you need to erase the concrete type but still use a protocol dynamically

## APIs & Frameworks

**Swift Language Features**
- **Generic type parameters** — `func f<T>(_ x: T)`, `struct Box<T> { ... }`
- **Protocol `associatedtype`** — `associatedtype Element`, `associatedtype Index`
- **`where` clauses** — `func f<T: Collection>(_ c: T) where T.Element: Equatable`
- **Protocol inheritance** — `protocol BidirectionalCollection: Collection { ... }`
- **Conditional conformances** **[NEW in Swift 4.2]** — `extension Array: Equatable where Element: Equatable`
- **Protocol composition** — `T: Hashable & Comparable`
- **`some` keyword** (opaque return types — previewed concept, formally arriving in Swift 5.1)
- **Recursive constraints** — `associatedtype SubSequence: Collection where SubSequence.Element == Element`

**Standard Library (benefiting from conditional conformances in Swift 4.2)**
- `Array: Equatable where Element: Equatable` **[NEW]**
- `Array: Hashable where Element: Hashable` **[NEW]**
- `Optional: Equatable where Wrapped: Equatable` **[NEW]**
- `Optional: Hashable where Wrapped: Hashable` **[NEW]**
- `Dictionary: Equatable where Value: Equatable` **[NEW]**
- Slice types: `Slice`, `ArraySlice`, `Substring` — all gain conditional conformances

## Code Highlights

Defining a protocol with an associated type:
```swift
protocol Container {
    associatedtype Element
    var count: Int { get }
    func element(at index: Int) -> Element
}
```

Protocol inheritance for richer capability:
```swift
protocol RandomAccessContainer: Container {
    func element(at index: Int) -> Element  // O(1) guarantee
}
```

Conditional conformance (Swift 4.2):
```swift
extension Array: Equatable where Element: Equatable {
    static func == (lhs: [Element], rhs: [Element]) -> Bool {
        guard lhs.count == rhs.count else { return false }
        return zip(lhs, rhs).allSatisfy { $0 == $1 }
    }
}
// Now: [1, 2, 3] == [1, 2, 3]  // works!
// And: [[1,2],[3]] == [[1,2],[3]]  // nested arrays also work
```

Recursive constraint on `SubSequence`:
```swift
protocol Collection: Sequence {
    associatedtype SubSequence: Collection
        where SubSequence.Element == Element,
              SubSequence.SubSequence == SubSequence
    subscript(bounds: Range<Index>) -> SubSequence { get }
}
```

## Takeaways
- Conditional conformances are the single most impactful Swift 4.2 generics addition — they allow conformances to compose automatically through type parameters, eliminating massive amounts of manual boilerplate.
- Design protocols by starting minimal and adding capabilities through inheritance rather than cramming all requirements into one protocol.
- Recursive constraints are the mechanism that makes deeply nested generic types like `Collection.SubSequence` type-safe without special-casing.
- Over-constraining generic parameters is the most common mistake — only add the protocol requirements that the specific algorithm actually needs to function correctly.

---
_Source: WWDC18 Session 406 page (abstract, related session links, and resource links)._
