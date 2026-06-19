# Protocol-Oriented Programming in Swift
**WWDC15 · Session 408** · [Watch](https://developer.apple.com/videos/play/wwdc2015/408/)

_Platforms:_ iOS 9, OS X El Capitan 10.11, watchOS 2

## Overview
This landmark session by Dave Abrahams (Swift Standard Library technical lead) introduces Protocol-Oriented Programming (POP) as the foundational design philosophy of Swift. Through a dialogue with the fictional "Crusty" — a skeptical old-school programmer — the session dismantles three core problems with class-based OOP (implicit sharing of mutable state, monolithic inheritance, poor type relationship modeling) and shows how protocols, protocol extensions, and value types address each one.

The session is the origin of the mantra "Don't start with a class. Start with a protocol." It demonstrates that Swift is the first protocol-oriented programming language — from `for`-in loops and string literals to the standard library's emphasis on generics, protocols are the primary abstraction mechanism. The session builds a complete diagramming app model using only protocols, structs, and enums (no classes), and shows how retroactive conformance allows `CGContext` to conform to a `Renderer` protocol without subclassing.

Protocol extensions — a new Swift 2 feature introduced here — enable default implementations, constrained extensions, and the kind of generics-meets-protocols magic that powers the redesigned Swift Standard Library.

## Key Topics

### Problems with Classes (Crusty's Three Beefs)
1. **Implicit sharing of mutable state** — passing a class instance shares the reference; mutations by one holder surprise other holders; leads to defensive copying, dispatch queues, locks, deadlocks, and bugs. Swift's value-type collections solve this at the standard library level.
2. **Inheritance is too intrusive** — single inheritance only; base class chosen at definition time; unwanted stored properties and initialization complexity; unclear override contracts (when to call `super`, where in the method).
3. **Poor type relationships** — symmetric operations (like comparison) cannot be expressed safely with classes; forced down-casts appear, signaling a static type-safety hole. Any forced down-cast is a "code smell" indicating a lost type relationship.

### The Protocol Solution
- Protocols eliminate all three problems: no implicit sharing (models can be value types), multiple protocol conformance, type relationships expressed via `Self`-requirements.
- **"Don't start with a class. Start with a protocol."**
- Replacing a class hierarchy with a protocol + struct conformances typically produces safer, more testable, and more performant code.

### Self-Requirements and Two Worlds
- A `Self`-requirement (`Self` used in protocol method signatures) moves a protocol into the **static/homogeneous world**: usable as a generic constraint, optimizable, but not usable as a heterogeneous existential type.
- Without `Self`-requirements: **dynamic/heterogeneous world** — usable as a type (`[Drawable]`), dynamically dispatched, supports mixed collections.
- Bridge between worlds: implement a non-`Self` protocol requirement (`isEqualTo`) via a constrained extension that performs the dynamic dispatch internally.

### Protocol Extensions **[NEW in Swift 2]**
- Add method implementations directly on a protocol; all conforming types get the implementation for free.
- A protocol **requirement** creates a customization point — even if an extension provides a default, conforming types can override it and the overridden version is dispatched when the static type is the concrete type.
- A method in a protocol extension **not** backed by a requirement only shadows (not overrides) the extension method; the extension version is called when the static type is the protocol.
- **Constrained extensions**: `extension Collection where Element: Equatable` — add APIs conditionally based on type constraints.
- Protocol extensions transformed the Swift Standard Library, converting free generic functions (angle-bracket-heavy signatures) into protocol methods (clean, discoverable call sites).

### Value Types and Equatable
- Prefer value types (structs/enums) for model types. Value semantics eliminates the implicit-sharing problem.
- Always make value types `Equatable` when meaningful.
- `Equatable` has `Self`-requirements, so adding `Equatable` to a protocol used as a heterogeneous existential type causes a contradiction. Solution: add a non-`Self` requirement (`isEqualTo(_:)`) and satisfy it via a constrained protocol extension for all `Equatable` conformers.

### When to Use Classes
- When copying semantics don't make sense (e.g., a `Window` — what would a copy mean?).
- When the instance lifetime is tied to an external side effect (file on disk, network connection).
- When the instance is a "sink" (accumulates state; `NSOutputStream`, `CGContext`).
- When Cocoa/Cocoa Touch APIs require subclassing (UIViewController, etc.) — don't fight the system, but be circumspect and factor logic into value types.
- Even then: no base class required, prefer `final`.

### Retroactive Conformance
- Extend any existing type — including system types — to conform to a protocol in any module.
- `extension CGContext: Renderer { ... }` — every `CGContext` becomes a `Renderer` without modifying Apple's source code.

### Testability via Protocol Decoupling
- Decoupling with a protocol (e.g., `Renderer`) allows plugging in a `TestRenderer` that captures drawing commands as strings — better than mocks because the interface is enforced by the type system, not fragile manual coupling.

## APIs & Frameworks

- `protocol` keyword — Swift protocol definition
- `extension` on protocols — **protocol extensions [NEW in Swift 2]**
- `Self` in protocol requirements — **Self-requirements** (type-safe homogeneous generics)
- Constrained protocol extensions: `extension P where Self: Q` / `extension Collection where Element: Equatable` **[NEW in Swift 2]**
- `struct` conforming to protocol (value-type model types)
- `Equatable` protocol — `==` operator, `Self`-requirement
- `Comparable` protocol — `<` operator
- `OptionSetType` protocol — new in Swift 2, powers option sets via protocol extensions **[NEW]**
- `CollectionType` / `Collection` protocol — `indexOf`, `binarySearch` via constrained extensions
- `SequenceType` / `Sequence` — `zip`, iteration
- `GeneratorType` / `IteratorProtocol`
- `CGContext` (CoreGraphics) — demonstrated as retroactively conforming to `Renderer`
- `NSUserActivity` (mentioned in context of value types session cross-reference)
- Swift Standard Library generic algorithms expressed as protocol extension methods (Swift 2 redesign) **[NEW]**

## Code Highlights

Protocol with Self-requirement (type-safe comparison):
```swift
protocol Ordered {
    func precedes(_ other: Self) -> Bool
}
struct Number: Ordered {
    var value: Double
    func precedes(_ other: Number) -> Bool { return value < other.value }
}
```

Protocol extension providing a default implementation:
```swift
protocol Renderer {
    func moveTo(_ point: CGPoint)
    func lineTo(_ point: CGPoint)
    func arcAt(_ center: CGPoint, radius: CGFloat, startAngle: CGFloat, endAngle: CGFloat)
}
extension Renderer {
    func circleAt(_ center: CGPoint, radius: CGFloat) {
        arcAt(center, radius: radius, startAngle: 0, endAngle: 2 * .pi)
    }
}
```

Retroactive conformance — making CGContext a Renderer:
```swift
extension CGContext: Renderer {
    func moveTo(_ point: CGPoint) { move(to: point) }
    func lineTo(_ point: CGPoint) { addLine(to: point) }
    // ...
}
```

Constrained extension for protocol interoperability:
```swift
extension Comparable where Self: Ordered {
    func precedes(_ other: Self) -> Bool { return self < other }
}
```

Bridge between static and dynamic worlds (heterogeneous equality):
```swift
extension Drawable {
    func isEqualTo(_ other: Drawable) -> Bool {
        guard let other = other as? Self else { return false }
        return self == other  // requires Equatable on Self
    }
}
```

## Takeaways
- Protocol-Oriented Programming is Swift's primary design philosophy — prefer protocols over base classes as the abstraction boundary, and value types as the implementation.
- Protocol extensions (new in Swift 2) enable default implementations and constrained generics, eliminating the need for free generic functions and enabling the redesigned standard library API surface.
- A forced down-cast in code is a signal that a type relationship has been lost — usually fixable by using a Self-requirement or a constrained protocol extension.
- Classes still have a place for reference semantics, external side effects, and Cocoa interop — but the default should be protocols + structs.

---
_Source: WWDC15 Session 408 page (abstract, chapter summaries, code samples, and resource links)._
