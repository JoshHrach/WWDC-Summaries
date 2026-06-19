# Protocol and Value Oriented Programming in UIKit Apps
**WWDC16 · Session 419** · [Watch](https://developer.apple.com/videos/play/wwdc2016/419/)

_Platforms:_ iOS 10, macOS Sierra 10.12

## Overview
This session builds directly on WWDC15's Protocol-Oriented Programming and Value Types sessions and shows how to apply those ideas to the view and controller layers of a real MVC-based UIKit app ("Lucid Dreams" — a dream-logging app). The central theme is **local reasoning**: the ability to understand a function or type purely from the code directly in front of you, without needing to trace interactions across a large class hierarchy.

The session is divided into three parts: the model layer (structs with value semantics), the view layer (protocol-backed layout abstractions with generics and composition), and the controller layer (composing model properties into a single struct and encoding mutually exclusive UI state in an enum). Sample code ("Lucid Dreams") accompanies the session.

## Key Topics

### Model Layer: Value Semantics
- Replacing `class Dream` with `struct Dream` eliminates implicit sharing — a change to one variable cannot silently mutate another.
- Value semantics make unit tests for model types accurate: a test struct in isolation actually represents the full object graph, unlike a class that may participate in complex ownership relationships.

### View Layer: Protocol-Backed Layout Structs
- Layout logic trapped inside a `UITableViewCell` subclass cannot be reused in plain `UIView` or `SKNode` hierarchies.
- Moving layout logic to a plain `struct` (`DecoratingLayout`, `CascadingLayout`) isolates it completely from UIKit.
- Define a `Layout` protocol with a `layout(in: CGRect)` method; make `UIView` and `SKNode` conform retroactively — this is **retroactive modeling** (protocol conformance for types you don't own).
- Use generics (`struct DecoratingLayout<Child: Layout>`) to enforce type-homogeneous children (all `UIView` or all `SKNode`, never mixed).
- Add an `associatedtype Content` to the protocol so layouts can return their contents in the correct stacking order as a strongly-typed array.
- Generic constraints (`where Child.Content == Decoration.Content`) express that two children must have the same content type at compile time.
- Composing structs instead of views is cheap: structs have no heap allocation overhead, unlike `UIView`, which carries drawing and event-handling machinery even when used purely for layout.

### Unit Testing with Protocols
- Replace `UIView` children with a trivial test struct conforming to `Layout` for pure unit tests of layout math.
- Tests don't require table views, view controllers, or layout callbacks — just create the struct, call `layout(in:)`, assert resulting frames.
- Background thread image rendering can also use the same `Layout` protocol.

### Controller Layer: Composing Model Properties
- Problem: two separate model properties (`dreams` and `favoriteCreature`) on a view controller require separate undo code paths; adding a third property means adding a third code path — maintenance nightmare.
- Solution: compose both properties into a single `Model` struct; implement undo by recording and restoring entire `Model` value snapshots.
- With snapshot-based undo, operations are order-independent: the undo step is always "replace current model with the stack value" followed by diffing old vs. new model to update UI.
- A single `modelDidChange(old:new:)` method compares old and new model values and applies minimal UITableView updates.

### Controller Layer: Enum for Mutually Exclusive UI State
- Problem: multiple Boolean/optional properties for UI mode (viewing, selecting, sharing) must be cleared atomically; forgetting to clear one causes inconsistent states (e.g., selection indicators remaining visible after cancelling share mode).
- Solution: encode all mutually exclusive modes in a single `enum UIState { case viewing, selecting(Set<Dream>), sharing }`.
- The type system prevents invalid intermediate states; all state transitions happen at once; state restoration is straightforward.

## APIs & Frameworks

- **Swift** — `struct`, `protocol`, `enum`, generics, associated types, retroactive conformance
- `UITableViewCell` — layout logic extracted out of subclasses into composable structs
- `UIView` — retroactively conformed to `Layout` protocol
- `SKNode` (SpriteKit) — retroactively conformed to same `Layout` protocol; layouts become platform-agnostic
- `UITableView` row/section updates — driven by `modelDidChange` diff
- `UndoManager` (implicit via `UIResponder` chain) — registered undo handler replaces entire `Model` value
- Protocol with associated type — `protocol Layout { associatedtype Content; mutating func layout(in: CGRect); var contents: [Content] { get } }`
- Generic struct with type constraints — `struct DecoratingLayout<Child: Layout, Decoration: Layout> where Child.Content == Decoration.Content`
- `@discardableResult`, `mutating func`, value type `struct` — Swift language features central to the patterns
- State restoration — single `UIState` enum property simplifies encoding/decoding app state

## Code Highlights

Protocol-backed layout struct (simplified):
```swift
protocol Layout {
    associatedtype Content
    mutating func layout(in rect: CGRect)
    var contents: [Content] { get }
}

struct DecoratingLayout<Child: Layout, Decoration: Layout>: Layout
    where Child.Content == Decoration.Content {
    var content: Child
    var decoration: Decoration

    mutating func layout(in rect: CGRect) {
        content.layout(in: contentRect(in: rect))
        decoration.layout(in: decorationRect(in: rect))
    }
    var contents: [Child.Content] { return content.contents + decoration.contents }
}
```

Snapshot-based undo in the controller:
```swift
func modelDidChange(old: Model, new: Model) {
    if old.favoriteCreature != new.favoriteCreature {
        tableView.reloadSections(IndexSet(integer: creatureSection), with: .automatic)
    }
    undoManager?.registerUndo(withTarget: self) { $0.model = old }
}
```

Enum state preventing invalid intermediate states:
```swift
enum UIState {
    case viewing
    case selecting(selectedDreams: Set<Dream>)
    case sharing
}
var state: UIState = .viewing
```

## Takeaways
- Use `struct` for model types to eliminate implicit sharing; if two variables must be independent, make them value types.
- Extract layout logic from `UITableViewCell` subclasses into plain structs backed by a protocol; this makes the logic reusable across `UIView`, `SKNode`, and background rendering, and trivially unit-testable without the UIKit stack.
- Compose small value types into a larger model struct and implement undo by recording whole-model snapshots; this collapses N undo code paths down to one regardless of how many model properties you add.
- Replace sets of mutually exclusive Boolean/optional state properties with a single `enum`; the compiler then makes invalid states unrepresentable.

---
_Source: WWDC16 Session 419 page (abstract, transcript, and resource links)._
