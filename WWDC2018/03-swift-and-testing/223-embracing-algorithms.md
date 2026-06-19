# Embracing Algorithms
**WWDC18 · Session 223** · [Watch](https://developer.apple.com/videos/play/wwdc2018/223/)

_Platforms:_ iOS, macOS, watchOS, tvOS (Swift / Swift Standard Library)

## Overview
This session makes the case that every app is already full of algorithms — loops, searches, mutations — but most developers don't recognize them as such. Speaker Dave Abrahams walks through a worked example: a vector drawing app ("Shapes") where a naive O(n²) delete-selection implementation is discovered, profiled, and incrementally replaced with Swift Standard Library generic algorithms. Along the way the session covers complexity analysis, the importance of documented semantics and performance, how to make concrete algorithms generic by removing irrelevant dependencies, and the "No Raw Loops" guideline.

The session is not about any specific algorithm; it is about cultivating the habit of identifying the fundamental computation in your code and expressing it as a named, tested, documented, generic algorithm.

## Key Topics

### Recognizing Quadratic Complexity

- A delete-selected-shapes loop calling `remove(at:)` inside a loop is O(n²): each `remove(at:)` is O(n) (slides following elements), and the outer loop iterates up to n times.
- The "elegant" reverse-iteration fix is still O(n²) — the asymptotic complexity is unchanged regardless of iteration direction.
- Small test data (10–20 items) hides quadratic performance because the constant factor makes O(n²) and O(n) look equivalent. Scalability matters as users accumulate more data.
- Fix: replace the loop+`remove(at:)` pattern with `removeAll(where:)` — a single O(n) pass over the collection.

### Know the Swift Standard Library

- The Standard Library ships with a documented suite of generic algorithms with stated complexity guarantees in Quick Help and online documentation.
- Key algorithms covered: `removeAll(where:)`, `firstIndex(where:)`, `partition(by:)`, `stablePartition(by:)` (open-source Swift), `rotate(to:)`, `removeSubrange(_:)`.
- Every algorithm has a dot-comment describing **what it does** and its **complexity** — read this before writing a loop.
- Familiarity does not mean memorization; knowing what exists and how to search for it is sufficient.

### Replace Raw Loops with Algorithm Calls

- "No Raw Loops" (Shawn Perin's guideline): every loop is either a known algorithm in disguise or a new algorithm that should be extracted, named, and documented.
- Benefits: shorter code, obviously correct code, faster code (library algorithms are O(n) or O(n log n) vs. hand-written O(n²)), and reusable code.
- Example: five different shape-reordering commands (bringToFront, sendToBack, bringForward, sendBackward, drag-in-list) all collapsed to one or two calls to `stablePartition(by:)` after algorithm identification.

### Stable Partition

- `stablePartition(by:)` moves all elements matching a predicate to the end (or beginning) while preserving relative order of both groups — O(n log n).
- Available in the Swift open-source project (swift/stdlib/public/core/Algorithms.swift); not yet in the standard library at time of WWDC18 but referenced as an aspirational addition.
- `bringToFront`: stable-partition unselected shapes to the end.
- `sendToBack`: stable-partition selected shapes to the end (inverted predicate).
- `bringForward` / `sendBackward`: stable-partition a **slice** of the collection starting at the predecessor of the first selected element.
- Drag-reorder: two consecutive stable-partitions with inverted predicates replaced ~30 lines of carefully maintained, bug-prone code.

### Slices Enable Algorithm Composition

- `Collection.Slice` (e.g., `shapes[predecessor...]`) shares indices with the underlying collection — indices do not restart at 0.
- Generic algorithms written against `MutableCollection` and `RangeReplaceableCollection` work on both full collections and slices without modification.
- Pitfall: comparing an index against the literal `0` breaks on slices; always compare against `startIndex`.

### Making Code Generic

- Steps to generalize a concrete algorithm:
  1. Move it off the concrete type (e.g., off `Canvas`) onto the collection type (e.g., `[Shape]` → `Array` extension → `MutableCollection` extension).
  2. Replace application-domain predicates (`.isSelected`) with a closure parameter.
  3. Remove remaining type constraints that are irrelevant to what the algorithm actually does.
  4. Document: describe the semantics and the complexity in a doc comment.
- Generic algorithms are more testable: they can be exercised on simple value types (integers, strings) in a playground without standing up the full application stack.
- Generic algorithms are more readable: removing domain-specific noise leaves only the essential computation.

### Documentation as a Contract

- Every algorithm must document: **what it does** (semantic contract) and **how fast** (complexity).
- Downstream callers compose algorithms by relying on these contracts without inspecting implementations — the same way you rely on `sort` being O(n log n) without reading its source.
- Undocumented code cannot be safely composed; whoever reads it next must reverse-engineer its invariants.

## APIs & Frameworks

**Swift Standard Library**
- `Collection.removeAll(where:)` — O(n); removes elements matching predicate in-place without a loop
- `Collection.firstIndex(where:)` — O(n); returns index of first matching element
- `MutableCollection.partition(by:)` — O(n); unstable partition (does not preserve relative order)
- `stablePartition(by:)` — O(n log n); preserves relative order of both partitions (open-source Swift; aspirational stdlib addition as of WWDC18)
- `rotate(to:)` — O(n); rotates elements in-place (used internally by stable partition)
- `RangeReplaceableCollection.removeSubrange(_:)` — O(k) where k = removed element count
- `Collection.Slice` — zero-copy sub-range view; indices match the parent collection
- Partial range operator (`startIndex...`) — convenient way to express a range to the end of a collection
- `remove(at:)` — O(n); slides following elements; avoid inside loops

## Code Highlights

Replacing a quadratic delete loop with a linear algorithm call:
```swift
// Before (O(n²)):
var i = shapes.count - 1
while i >= 0 {
    if shapes[i].isSelected { shapes.remove(at: i) }
    else { i -= 1 }
}

// After (O(n)):
shapes.removeAll(where: { $0.isSelected })
```

Bring-to-front using `stablePartition` (O(n log n)):
```swift
/// Moves selected shapes to the front, preserving their relative order.
/// Complexity: O(n log n)
extension Array where Element == Shape {
    mutating func bringToFront(where isSelected: (Element) -> Bool) {
        stablePartition(by: { !isSelected($0) })
        // Unselected go to back; selected collect at front.
    }
}
```

Bring-forward on a slice, fully generic (O(n log n)):
```swift
/// Moves elements satisfying `predicate` forward by one position
/// relative to the first non-matching predecessor.
/// Complexity: O(n log n)
extension MutableCollection {
    mutating func bringForward(where predicate: (Element) -> Bool) {
        guard let predecessor = indexBeforeFirst(where: predicate) else { return }
        self[predecessor...].stablePartition(by: { !predicate($0) })
    }
}
```

Rubber-band displacement formula (helper used in bringForward):
```swift
/// Returns the index before the first element satisfying `predicate`,
/// or `nil` if no such element exists or the matching element is at `startIndex`.
/// Complexity: O(n)
extension Collection {
    func indexBeforeFirst(where predicate: (Element) -> Bool) -> Index? {
        guard let first = firstIndex(where: predicate),
              first != startIndex else { return nil }
        return index(before: first)
    }
}
```

## Takeaways
- Every loop in your app is either a known algorithm (find it in the Standard Library) or a new algorithm that deserves to be extracted, named, documented, and tested.
- O(n²) behavior from `remove(at:)` inside a loop is a common and serious scalability bug; `removeAll(where:)` is almost always the right fix.
- `stablePartition(by:)` solves a surprisingly wide range of reordering problems (bring-to-front, send-to-back, drag-reorder) — recognize it.
- Generalize algorithms by removing irrelevant domain dependencies one level at a time; the result is more testable, more readable, and composable across different contexts.

---
_Source: WWDC18 Session 223 page (abstract, full transcript, and resource links)._
