# Design Protocol Interfaces in Swift
**WWDC22 · Session 110353** · [Watch](https://developer.apple.com/videos/play/wwdc2022/110353/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, watchOS 9, tvOS 16

## Overview
This session is a deep dive into advanced protocol design in Swift 5.7, building directly on the concepts introduced in "Embrace Swift Generics" (session 110352). It covers three major topics: how protocols with associated types interact with existential `any` types through type erasure, how opaque result types with primary associated type constraints separate interface from implementation, and how same-type requirements express and enforce relationships between multiple related abstract types.

The session uses a farm/animal feeding model as a running example to make the abstract concepts concrete. By the end, viewers understand when to use `any` vs `some`, how to constrain opaque return types to expose just enough interface, and how `where` clauses with same-type requirements collapse infinite chains of nested associated types into precise, verifiable relationships.

## Key Topics

### Existential Types and Associated-Type Erasure
- Protocols with associated types can now be used with `any` keyword in Swift 5.7 **[NEW]**.
- When calling a protocol method with an associated type in **producing position** (return type), the associated type is type-erased to its upper bound (another existential type).
- When an associated type is in **consuming position** (parameter), type erasure cannot be performed safely — must unbox with `some` instead.
- `any Animal` stores any concrete Animal dynamically via a box representation; calling `produce()` on `any Animal` returns `any Food` (the upper bound of `Commodity`).
- Self-returning protocol methods (e.g., `clone() -> Self`) follow the same erasure rule: result is erased to `any Cloneable`.

### Opaque Result Types and Primary Associated Types
- Opaque result types (`some Protocol`) hide implementation details while preserving static type information for callers.
- **Primary associated types** (new in Swift 5.7): declared with angle brackets after the protocol name — `protocol Collection<Element>` **[NEW]**.
- **Constrained opaque result types** (new in Swift 5.7): `some Collection<any Animal>` — exposes the Element type constraint without revealing the concrete collection type **[NEW]**.
- **Constrained existential types** (new in Swift 5.7): `any Collection<any Animal>` — used when the function may return different concrete types across calls **[NEW]**.
- Primary associated types should represent caller-provided concepts (like Element) not implementation details (like Iterator).
- Standard library protocols `Collection`, `Sequence`, `AsyncSequence` now declare primary associated types.

### Same-Type Requirements and Protocol Relationships
- Same-type requirements (in `where` clauses) express that two different associated types must be the same concrete type.
- Without same-type requirements, protocols can model infinite chains of nested associated types.
- Adding `where Self.CropType.FeedType == Self` to `AnimalFeed` collapses the infinite chain into a precise pair of related types.
- Same-type requirements prevent accidentally conforming concrete types that violate the intended invariant.
- Generic code can rely on same-type requirements to chain protocol method calls correctly (e.g., grow then harvest returns the exact FeedType expected by `eat()`).

## APIs & Frameworks

### Swift 5.7 Language Features **[NEW]**
- `any Protocol` — existential type syntax for protocols, including protocols with associated types **[NEW]**
- `some Protocol` — opaque type syntax (existing, expanded usage)
- `protocol MyProtocol<PrimaryAssocType>` — primary associated type declaration syntax **[NEW]**
- `some Collection<Element>` — constrained opaque result type **[NEW]**
- `any Collection<Element>` — constrained existential type **[NEW]**
- `where AssocType1 == AssocType2` — same-type requirement in `where` clause
- Associated type **producing position** (return type) — type erasure applies
- Associated type **consuming position** (parameter) — type erasure does not apply
- Upper bound of an associated type — the existential type carrying the associated type's constraints

### Standard Library
- `Collection` protocol — now has `Element` as a primary associated type **[NEW]**
- `Sequence` protocol — primary associated type **[NEW]**
- `LazyFilterSequence` — concrete type hidden behind opaque result type
- `Array.filter()` vs `Array.lazy.filter()` — example for hiding implementation details with opaque types

## Code Highlights

```swift
// Existential type with associated type (Swift 5.7)
let animals: [any Animal] = [Cow(), Chicken()]

// Associated type erasure: produce() returns 'any Food' (upper bound)
let food: any Food = animals[0].produce()

// Primary associated type declaration (Swift 5.7)
protocol Feed<AnimalType> {
    associatedtype AnimalType: Animal
}

// Constrained opaque result type (Swift 5.7)
var hungryAnimals: some Collection<any Animal> {
    animals.lazy.filter(\.isHungry)
}

// Constrained existential type (Swift 5.7)
func hungryAnimals() -> any Collection<any Animal> { ... }

// Same-type requirement collapsing infinite chain
protocol AnimalFeed {
    associatedtype CropType: Crop where CropType.FeedType == Self
}
protocol Crop {
    associatedtype FeedType: AnimalFeed where FeedType.CropType == Self
}
```

## Takeaways
- Use `any` for existential types where heterogeneous storage is needed; be aware that associated types in consuming position cannot be erased and require switching to `some` (unboxing).
- Declare primary associated types on your protocols to enable constrained opaque and existential types, giving callers enough type information without exposing implementation details.
- Use same-type requirements in `where` clauses to precisely model invariants between related protocols, preventing accidental misuse by concrete conformers and enabling correct generic code.
- The companion session "Embrace Swift Generics" (110352) is a prerequisite for this content.

---
_Source: WWDC22 Session 110353 page (abstract, chapter summaries, code samples, and resource links)._
