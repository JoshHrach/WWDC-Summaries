# Modern Swift API Design
**WWDC19 · Session 415** · [Watch](https://developer.apple.com/videos/play/wwdc2019/415/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15, watchOS 6, tvOS 13

## Overview
This session distills the hard-won lessons from designing Swift-first frameworks (SwiftUI, RealityKit, Combine) into practical API design guidance. Two engineers — Ben Cohen and Doug Gregor — cover four interconnected topics: when to use value types versus reference types, how to use protocols and generics without over-engineering, a new key-path-based dynamic member lookup feature for forwarding computed properties, and the new property wrappers feature for abstracting data-access policy.

A running example is a geometry API backed by SIMD types, which illustrates when to use protocol conformance versus generic struct composition, how to expose forwarded properties cleanly with `@dynamicMemberLookup`, and how copy-on-write can be implemented transparently using a property wrapper. SwiftUI's `@State`, `@Binding`, `@Environment`, and `@EnvironmentObject` are revealed as property wrappers, and the way `$slide.title` resolves through `Binding`'s dynamic member subscript is explained step by step.

A key naming policy change: new Swift-only Apple frameworks (SwiftUI, RealityKit, Combine) drop the two- or three-letter prefix — rely on Swift's module system for disambiguation instead.

## Key Topics
- **No prefixes in Swift-only APIs** — SwiftUI/RealityKit/Combine types are unprefixed; Objective-C-bridged types retain prefixes for compatibility
- **Value vs. reference semantics** — prefer structs unless the type manages resources, is fundamentally shared, or has identity; accidental mixed semantics (struct containing a mutable class property) produce surprising behavior
- **Copy-on-write** — use `isKnownUniquelyReferenced(_:)` in a private setter to implement full value semantics over a reference-type backing store
- **Protocols vs. generic types** — don't build protocol hierarchies for classification; extend existing protocols or use generic structs for code reuse; unnecessary protocol witness tables inflate binary size and slow compilation
- **Composing via `has-a` rather than `is-a`** — wrap a SIMD value in a generic struct rather than conforming to SIMD; exposes only the intended API surface
- **`@dynamicMemberLookup` with key paths** — a single `subscript(dynamicMember:)` accepting a `KeyPath` exposes every property of a contained type as a computed property on the wrapper, with full type safety and Xcode autocompletion **[NEW Swift 5.1]**
- **Property wrappers** — `@propertyWrapper` types encapsulate storage + access policy; compiler synthesizes backing `$property` stored property and computed `property` forwarding through `value`; SwiftUI's data-binding attributes (`@State`, `@Binding`, `@Environment`, `@EnvironmentObject`) are all property wrappers **[NEW Swift 5.1]**
- **`Binding` + key path member lookup** — `$slide.title` desugars to `$slide[dynamicMember: \.title]`, returning a `Binding<String>` focused on the nested property

## APIs & Frameworks

### Swift 5.1 Language Features
- `@dynamicMemberLookup` attribute on a type **[NEW]**
- `subscript(dynamicMember keyPath: KeyPath<Root, Value>) -> Value` — get-only forwarding **[NEW]**
- `subscript(dynamicMember keyPath: WritableKeyPath<Root, Value>) -> Value` — get+set forwarding with copy-on-write **[NEW]**
- `@propertyWrapper` attribute **[NEW]**
  - Required: `var value: T { get set }` — the policy-enforcing computed property
  - Optional: `init()` — triggers implicit initialization on property declaration
  - Optional: `init(initialValue:)` — enables default-value syntax on the property declaration
  - Backing storage: compiler synthesizes `$propertyName` as the stored wrapper instance
  - Custom attribute initialization: `@DefensiveCopying(withoutCopying:)` syntax calls a custom initializer on the wrapper type
- `isKnownUniquelyReferenced(_:)` — test for unique reference before mutating, implementing copy-on-write

### SwiftUI Property Wrappers (all NEW)
- `@State` — view-local mutable state; storage managed by SwiftUI
- `@Binding` — first-class reference to state owned elsewhere; `Binding<T>` generic over wrapped value type
- `@Environment` — reads a value from the SwiftUI environment by key path
- `@EnvironmentObject` — reads an `ObservableObject` injected into the environment

### RealityKit (design examples)
- `Entity` — reference type (identity-bearing, centrally stored in the engine)
- `Material` — value type (copying gives an independent copy; avoids accidental scene-wide mutations)
- `Transform` (position, orientation) — value types as attributes of `Entity`

### Standard Library
- `SIMD` protocol — homogeneous tuple-like numeric storage; constrainable via `Scalar: FloatingPoint`
- `SIMD2<Scalar>`, `SIMD3<Scalar>`, `SIMD4<Scalar>` — concrete generic structs, not separate protocols
- `isKnownUniquelyReferenced(_:)` — foundation for copy-on-write types

## Code Highlights

```swift
// Key path member lookup: expose all SIMD properties on a wrapper struct
@dynamicMemberLookup
struct GeometricVector<Storage: SIMD> where Storage.Scalar: FloatingPoint {
    var value: Storage

    // Single subscript exposes every property of Storage
    subscript<T>(dynamicMember keyPath: KeyPath<Storage, T>) -> T {
        value[keyPath: keyPath]
    }
}
// Usage: vector.x, vector.y, vector.z — all work, all type-safe
```

```swift
// Property wrapper: copy-on-write for any NSCopying type
@propertyWrapper
struct DefensiveCopying<T: NSCopying> {
    private var storage: T

    init(initialValue: T) {
        storage = initialValue.copy() as! T
    }

    var value: T {
        get { storage }
        set { storage = newValue.copy() as! T }
    }
}

class MyView {
    @DefensiveCopying var path: UIBezierPath = UIBezierPath()
    // Any assignment to `path` triggers a defensive copy automatically
}
```

```swift
// Late-initialized property wrapper (set-before-read enforcement)
@propertyWrapper
struct LateInitialized<T> {
    private var storage: T?

    var value: T {
        get {
            guard let v = storage else { fatalError("Not yet initialized") }
            return v
        }
        set { storage = newValue }
    }
}

class Controller {
    @LateInitialized var model: DataModel
    // Compile-time-safe; fatalError if read before first set
}
```

```swift
// Binding + dynamic member lookup: $slide.title returns Binding<String>
struct ContentView: View {
    @Binding var slide: Slide

    var body: some View {
        TextField("Title", text: $slide.title)
        // $slide       → Binding<Slide>  (compiler-generated backing property)
        // $slide.title → Binding<String> (via Binding's dynamicMember subscript)
    }
}
```

## Takeaways
- Default to structs; reach for classes only when the type manages resources, has meaningful identity, or is inherently shared — and document that choice explicitly.
- Before creating a new protocol, try writing a constrained extension on an existing protocol or using a generic struct; this reduces binary size, improves compile times, and avoids type-hierarchy maintenance.
- `@dynamicMemberLookup` with `KeyPath` subscripts is the cleanest way to forward a contained type's entire property surface with full type safety — use it to implement delegation or to expose copy-on-write wrappers without writing dozens of forwarding properties.
- Property wrappers are the right tool for any recurring data-access policy (lazy init, defensive copy, user defaults, thread-local storage, command line arguments); SwiftUI's entire data-flow model (`@State`, `@Binding`, `@Environment`) is built from them.
- Drop prefixes from new Swift-only frameworks; use `ModuleName.TypeName` disambiguation only when true conflicts arise.

---
_Source: WWDC19 Session 415 page (full transcript, abstract, and resource links)._
