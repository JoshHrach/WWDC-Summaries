# Embrace Swift Generics
**WWDC22 · Session 110352** · [Watch](https://developer.apple.com/videos/play/wwdc2022/110352/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, tvOS 16, watchOS 9

## Overview
Presented by Holly from the Swift Compiler team, this session teaches the full workflow for writing generic Swift code: starting with concrete types, identifying duplicated behavior, extracting a protocol, and finally writing parametric code using the new `some` and `any` keywords. It frames generics as an abstraction tool for managing complexity as code evolves, and explains the conceptual distinction between opaque types (`some`) and existential types (`any`).

The session uses a farm simulation as a running example to make the abstract concepts grounded. A key insight: class inheritance is the wrong tool for protocol-like abstractions — it forces reference semantics, requires runtime checks for type safety, and produces awkward generics. Protocols with associated types, combined with `some` and `any`, are the right tool.

## Key Topics

### Polymorphism Strategies
Three forms of polymorphism in Swift:
1. **Ad-hoc (overloading)**: Same function name, different parameter types — leads to repetitive implementations.
2. **Subtype (class hierarchy)**: Superclass polymorphism — forces reference semantics, runtime type checks, awkward associated-type handling.
3. **Parametric (generics)**: Write one piece of code, use concrete types as arguments — the preferred approach.

### Protocols with Associated Types
- `protocol Animal` — defines the capability interface without implementation details.
- `associatedtype Feed: AnimalFeed` — placeholder type that depends on the specific conforming type; guaranteed consistent per instance.
- Conforming types implement requirements; the compiler infers associated types from method parameter/return types.
- `typealias Feed = Hay` — explicit associated type declaration (optional; usually inferred).
- Protocols decouple "what a type does" from "how it does it."

### Opaque Types (`some`) **[NEW in Swift 5.7 for parameter position]**
- `some Animal` — declares an opaque type: a specific, fixed underlying type that satisfies the constraint.
- Equivalent to a named type parameter `<A: Animal>` in a `where` clause, but without the syntactic noise when the type only appears once.
- **Underlying type is fixed** for the scope of the value — guaranteed to be the same concrete type across all accesses.
- For parameters: the caller provides the value; the caller determines the underlying type; the implementation sees only the abstract type.
- For results: the implementation determines the underlying type; all return statements must return the same concrete type.
- Enables full access to associated types and protocol relationships.
- `some` in parameter position is **new in Swift 5.7** **[NEW]**.

### Existential Types (`any`) **[NEW for protocols with associated types in Swift 5.7]**
- `any Animal` — declares an existential type: a single static type that can store any concrete animal type dynamically.
- Underlying type **can vary at runtime** — type erasure at the static level.
- Memory representation: a "box" — small values stored inline; large values heap-allocated with a pointer.
- Using `any` with protocols that have associated types is **new in Swift 5.7** **[NEW]**.
- Enables heterogeneous collections: `[any Animal]`.
- Calling protocol methods on `any Animal` directly: blocked by the compiler when associated types are involved — the associated type is erased.
- **Fix**: pass the `any Animal` value to a function accepting `some Animal` — the compiler **unboxes** the existential, fixing the underlying type for the function scope **[NEW behavior in Swift 5.7]**.

### When to Use `some` vs `any`
- **Default: use `some`** — preserves type relationships, allows access to associated types, zero overhead.
- **Use `any` when**: storing heterogeneous collections, representing absence of underlying type (optional-like), or making the abstraction an implementation detail.
- Analogy: write `let` by default, change to `var` only when mutation is needed.

### Named Type Parameters and `where` Clauses
- Use named type parameters (`<A: Animal>`) when the opaque type needs to appear multiple times in the signature (e.g., return type depends on parameter type).
- `where` clauses specify constraints between type parameters and associated types.
- Example: `func buildHabitat<A: Animal>(for animal: A) -> A.Habitat` — return type depends on the animal's associated `Habitat` type.

### Protocol Associated Type Constraints
- `protocol AnimalFeed { associatedtype CropType: Crop where CropType.Feed == Self }` — constrains the associated type's associated type, forming a reciprocal relationship.
- `protocol Crop { associatedtype Feed: AnimalFeed where Feed.CropType == Self }` — ensures the round-trip: grow a crop, harvest it, get the correct feed type for the animal.
- These same-type requirements allow the compiler to verify type relationships across multi-step method chains.

## APIs & Frameworks

### Swift Language Features
- `protocol` with `associatedtype` — abstract interface for types **[fundamental, enhanced]**
- `some Protocol` — opaque type in parameter position **[NEW in Swift 5.7]**
- `some Protocol` — opaque result type (existed; parameter position is new)
- `any Protocol` — existential type **[NEW for protocols with associated types in Swift 5.7]**
- Implicit unboxing of `any` to `some` at call sites **[NEW in Swift 5.7]**
- `associatedtype Name: Protocol where Name.AssocType == Self` — constrained associated type
- `typealias` — explicit associated type implementation in conformance
- `@ViewBuilder` — result builder for multi-branch opaque result types (SwiftUI example)
- Named type parameters `<T: Protocol>` — for signatures referencing the opaque type multiple times
- Trailing `where` clause — complex type constraints

## Code Highlights

```swift
// Protocol with associated types forming a bidirectional constraint
protocol AnimalFeed {
    associatedtype CropType: Crop where CropType.Feed == Self
    static func grow() -> CropType
}

protocol Crop {
    associatedtype Feed: AnimalFeed where Feed.CropType == Self
    func harvest() -> Feed
}

protocol Animal {
    associatedtype Feed: AnimalFeed
    func eat(_ food: Feed)
}

// Concrete types
struct Hay: AnimalFeed {
    static func grow() -> Alfalfa { Alfalfa() }
}
struct Alfalfa: Crop {
    func harvest() -> Hay { Hay() }
}
struct Cow: Animal {
    func eat(_ food: Hay) {}
}

// Generic function using `some` — opaque type, fixed underlying type
struct Farm {
    // 'some Animal' equivalent to <A: Animal>(animal: A) -> Void
    func feed(_ animal: some Animal) {
        let crop = type(of: animal).Feed.grow()   // access associated type
        let produce = crop.harvest()
        animal.eat(produce)                        // type-safe: wrong food is a compile error
    }

    // Heterogeneous collection requires `any` existential
    func feedAll(_ animals: [any Animal]) {
        for animal in animals {
            feed(animal)   // compiler unboxes any Animal → some Animal
        }
    }
}

// Named type parameter needed when return type depends on input type
extension Farm {
    func buildHabitat<A: Animal>(for animal: A) -> A.Habitat { ... }
}

// Opaque result type — return type fixed by implementation
struct MyView: View {
    var body: some View {      // underlying type: Text (always the same)
        Text("Hello, farm!")
    }
}
```

## Takeaways
- Reach for protocols with associated types instead of class hierarchies — protocols work with value types, enforce type safety at compile time, and express "what a type can do" without implementation noise.
- `some Protocol` is the default choice for generic parameters: it's a simpler syntax for a named type parameter, preserves associated-type access, and costs nothing at runtime.
- `any Protocol` is for when you need to store values of different underlying types in the same variable or collection; pay the type-erasure cost only when you need the storage flexibility.
- Swift 5.7's implicit unboxing of `any` to `some` at call sites eliminates the old need for `AnyAnimal` wrapper types in most cases.
- Build protocols that encode reciprocal type relationships (`where CropType.Feed == Self`) to let the compiler verify multi-step transformations end-to-end.

---
_Source: WWDC22 Session 110352 page (transcript, code samples, and resource links)._
