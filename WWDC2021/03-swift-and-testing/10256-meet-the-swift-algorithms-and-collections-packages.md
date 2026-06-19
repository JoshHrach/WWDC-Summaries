# Meet the Swift Algorithms and Collections Packages
**WWDC21 · Session 10256** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10256/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, tvOS 15, watchOS 8 (Swift open-source packages — available on all platforms supporting Swift)

## Overview
Apple introduces two open-source Swift packages that extend the standard library's collection capabilities: **Swift Algorithms** (sequence and collection algorithms) and **Swift Collections** (new data structures). Both serve as incubation venues — algorithms and types proven useful in these packages are candidates for eventual inclusion in the Swift Standard Library. Both packages are available immediately via Swift Package Manager from GitHub.

## Key Topics

**Swift Algorithms: Vocabulary-Building**
The Algorithms package provides over 40 algorithms that complement built-in Swift operations like `map`, `filter`, `flatMap`, and `compactMap`. Its philosophy: replace raw `for` loops with named algorithms to make code clearer, faster (lazy adapters avoid intermediate allocations), and more correct.

**Lazy Adapters**
Many standard library algorithms return lazy adapters (e.g., `joined()` returns `FlattenSequence`, `reversed()` returns `ReversedCollection`). These are thin, allocation-free wrappers that compute elements on demand. Adding `.lazy` to the start of a chain makes closure-based algorithms (`map`, `filter`, `compactMap`) lazy too. To materialize a lazy chain into an array, wrap in `Array(...)`. For collections iterated repeatedly, collect into an array first rather than recomputing lazily on each iteration.

**Key Algorithms in the Algorithms Package**
- `windows(ofCount:)` — sliding window of a fixed size; vends `ArraySlice` subsequences.
- `adjacentPairs()` — convenient window-of-2; vends `(Element, Element)` tuples.
- `chunks(ofCount:)` — non-overlapping fixed-size chunks; last chunk may be shorter.
- `chunked(on:)` — chunks consecutive elements sharing a property value; vends `(value, chunk)` tuples.
- `chunked(by:)` — chunks via custom predicate on adjacent pairs; returns true to keep in same chunk.
- `joined(by:)` — variant of `joined` that computes the separator from adjacent chunks.
- `combinations(ofCount:)`, `permutations(ofCount:)` — combinatorics.
- `randomSample(count:)` — random subset of exactly N elements.
- `min(count:)` / `max(count:)` — the N smallest / largest elements.

**Swift Collections: Three New Data Structures**
The standard library provides `Array`, `Set`, and `Dictionary`. Swift Collections adds order-preserving and double-ended variants:

**`Deque<Element>`**
A double-ended queue with O(1) prepend and append. Internally stores elements in a ring buffer (wrap-around), so `prepend` does not shift existing elements. Conforms to `RandomAccessCollection`, `MutableCollection`, `RangeReplaceableCollection` — same API surface as `Array`. Random-element removal is ~2x faster than `Array` on average because the shorter side is moved to fill the gap.

**`OrderedSet<Element: Hashable>`**
An array-like collection guaranteeing element uniqueness while preserving insertion order. Conforms to `RandomAccessCollection` (integer indices). Does not conform to `SetAlgebra` (order matters for equality) but provides `.unordered` view that does conform to `SetAlgebra` for order-insensitive operations. `append(_:)` returns `(inserted: Bool, index: Int)`. `insert(_:at:)` is supported but O(n). Comparable memory use to `Set` thanks to compressed index storage. Not a drop-in for `NSOrderedSet` — no bridging.

**`OrderedDictionary<Key: Hashable, Value>`**
Key-value sequence with well-defined insertion order. Key-based subscript `dict[key]` is the primary subscript (same as `Dictionary`). Does not provide an integer-index subscript to avoid ambiguity with integer keys. Conforms to `Sequence` only (not `Collection`). Use `.elements` view (`RandomAccessCollection` of `(key, value)` pairs) when collection conformance is needed. Memory-efficient: one compressed hash table + two parallel arrays.

## APIs & Frameworks

### Swift Algorithms Package **[NEW — open source]**
Add via SPM: `https://github.com/apple/swift-algorithms`

**Iteration**
- `Sequence.windows(ofCount: Int)` — sliding windows **[NEW]**
- `Sequence.adjacentPairs()` — window of 2 as tuple **[NEW]**
- `Sequence.chunks(ofCount: Int)` — fixed-size non-overlapping chunks **[NEW]**
- `Sequence.chunked(on:)` — chunks by key path equality **[NEW]**
- `Sequence.chunked(by:)` — chunks by adjacent-pair predicate **[NEW]**

**Joining**
- `Sequence.joined(by:)` — join with computed separator from adjacent chunks **[NEW]**

**Selection**
- `Sequence.min(count:)` / `.max(count:)` — N smallest/largest elements **[NEW]**
- `Sequence.randomSample(count:)` — random N elements **[NEW]**

**Combinatorics**
- `Sequence.combinations(ofCount:)` **[NEW]**
- `Sequence.permutations(ofCount:)` **[NEW]**

### Swift Collections Package **[NEW — open source]**
Add via SPM: `https://github.com/apple/swift-collections`

**Deque**
- `Deque<Element>: RandomAccessCollection, MutableCollection, RangeReplaceableCollection` **[NEW]**
- `deque.prepend(_:)` — O(1) front insertion **[NEW]**
- `deque.append(_:)` — O(1) back insertion **[NEW]**
- `deque.removeFirst()` / `.removeLast()` — O(1) **[NEW]**

**OrderedSet**
- `OrderedSet<Element: Hashable>: RandomAccessCollection` **[NEW]**
- `orderedSet.append(_:) -> (inserted: Bool, index: Int)` **[NEW]**
- `orderedSet.insert(_:at:) -> (inserted: Bool, index: Int)` **[NEW]**
- `orderedSet.unordered: UnorderedView` — `SetAlgebra`-conforming view **[NEW]**
- `orderedSet.formUnion(_:)`, `.subtract(_:)`, `.intersection(_:)` — order-preserving set operations **[NEW]**

**OrderedDictionary**
- `OrderedDictionary<Key: Hashable, Value>: Sequence` **[NEW]**
- `dict[key]` — key-based subscript (primary) **[NEW]**
- `dict.elements` — `RandomAccessCollection` of `(key, value)` pairs **[NEW]**
- `dict.elements[index]` — integer-indexed key-value pair access **[NEW]**

## Code Highlights

Chunking and joining with separator (transcript timestamps):
```swift
import Algorithms

transcript = Array(
    messages
        .lazy
        .flatMap { $0.makeMessageParts() }
        .chunked { $1.date.timeIntervalSince($0.date) < 60 * 60 }
        .joined { DateItem(date: $1.first!.date) }
)
```

Deque for O(1) prepend/append:
```swift
import Collections

var queue: Deque = ["A", "B", "C"]
queue.prepend("Z")  // O(1)
queue.append("D")   // O(1)
queue.removeFirst() // O(1)
```

OrderedSet preserving insertion order:
```swift
import Collections

var items: OrderedSet = ["E", "D", "C", "B", "A"]
items.append("F")           // (inserted: true, index: 5)
items.insert("B", at: 1)    // (inserted: false, index: 3) — already exists
items.formUnion(["X", "Y"])
print(items.unordered == otherSet.unordered) // order-insensitive comparison
```

OrderedDictionary with elements view:
```swift
import Collections

var dict: OrderedDictionary = [2: "two", 1: "one", 0: "zero"]
print(dict[1])           // Optional("one") — key-based
print(dict.elements[0])  // (key: 2, value: "two") — index-based
```

## Takeaways
- `chunked(by:)` and `windows(ofCount:)` eliminate the most common verbose raw-loop patterns: tracking previous values and sliding-window comparisons.
- Lazy algorithm chains avoid intermediate allocations for pipelines that only consume a small prefix/suffix; wrap in `Array(...)` as the final step when you need a materialized result or iterate the chain multiple times.
- `Deque` is a drop-in for `Array` when prepending is frequent — it offers O(1) front insertion vs. O(n) for `Array`.
- `OrderedSet` is the right choice when uniqueness and user-controlled ordering both matter; use `.unordered` for set algebra operations that shouldn't depend on order.

---
_Source: WWDC21 Session 10256 page (abstract, chapter summaries, code samples, and resource links)._
