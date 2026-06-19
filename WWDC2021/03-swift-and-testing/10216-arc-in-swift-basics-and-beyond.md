# ARC in Swift: Basics and Beyond
**WWDC21 · Session 10216** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10216/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, watchOS 8, tvOS 15

## Overview
This session provides a thorough treatment of Automatic Reference Counting (ARC) in Swift: how it works, when object lifetimes are observable, and what problems can arise from relying on observed (rather than guaranteed) object lifetimes. The session is motivated by a new Xcode 13 build setting — "Optimize Object Lifetimes" — which applies more aggressive ARC optimizations and can expose latent lifetime bugs.

The key insight is that Swift object lifetimes are use-based (guaranteed to end at last use) rather than scope-based (ending at closing brace like C++). Observed lifetimes may extend beyond guaranteed minimums due to ARC optimizations, and code that silently relies on those extensions will break when optimizations improve or source code changes. The session covers two main observable lifetime features — weak/unowned references and deinitializer side effects — with safe techniques for handling each.

## Key Topics

### ARC Fundamentals
- Object lifetime: begins at initialization, ends at last use (use-based, not scope-based)
- ARC inserts `retain` (increment reference count) and `release` (decrement) around uses
- Object deallocated when reference count drops to zero
- Observed lifetimes may exceed guaranteed minimums due to compiler optimizations — never rely on this

### Observable Lifetime Feature 1: Weak and Unowned References
- **Weak** (`weak var`): automatically becomes `nil` when the referred object is deallocated; optional type
- **Unowned** (`unowned var`): traps (crash) if accessed after the referred object is deallocated
- Both used to break reference cycles — they do not participate in reference counting

**Reference Cycles**: when two objects hold strong references to each other, their reference counts never reach zero → memory leak. Mark one side `weak` or `unowned` to break the cycle.

**The Danger**: if a weak reference is accessed after the object's guaranteed lifetime ends (even if it's still alive due to observed lifetime), a future compiler optimization may cause it to become `nil` or trigger a trap. Optional binding with `if let` makes the bug silent, not safe.

**Safe Techniques for Weak/Unowned References**:
1. `withExtendedLifetime(_:_:)` — explicitly extend an object's lifetime over a code block
2. `defer { withExtendedLifetime(obj) {} }` — extend to end of scope
3. **Redesign to access via strong reference only** — move the method to the strongly-held side
4. **Eliminate the cycle** — restructure to a tree (e.g., extract shared data into a separate class both can reference)

### Observable Lifetime Feature 2: Deinitializer Side Effects
- `deinit` runs just before deallocation; side effects (file I/O, publishing metrics, etc.) are observable
- If external code expects deinit to run before some subsequent operation, that ordering is not guaranteed — the deinit may run earlier (right after last use) due to ARC optimization

**Safe Techniques for Deinitializer Side Effects**:
1. `withExtendedLifetime(_:_:)` — keep object alive until dependent operations complete
2. **Make deinit effects private** — compute and publish within deinit using `private` fields; no external sequencing needed
3. **Replace deinit side effects with `defer`** — call an explicit `publishAllMetrics()` method with `defer`; deinit becomes only an assertion/sanity check

### Xcode 13: "Optimize Object Lifetimes" Build Setting
- New experimental build setting in Xcode 13 **[NEW]**
- Enables aggressive lifetime-shortening ARC optimizations
- Objects deallocated much more consistently at last use → brings observed lifetimes closer to guaranteed minimums
- Turning this on may expose hidden lifetime bugs; use safe techniques to fix them

## APIs & Frameworks

### Swift Standard Library
- `withExtendedLifetime(_:_:)` — explicit lifetime extension
  - `withExtendedLifetime(obj) { /* dependent code */ }`
  - `withExtendedLifetime(obj) {}` — at end of scope as a no-op to extend to that point
- `defer { withExtendedLifetime(obj) {} }` — extend to end of current scope
- `weak var` — weak reference; optional, nil-ed on deallocation
- `unowned var` — unowned reference; traps on access after deallocation
- `deinit` — class deinitializer; runs before deallocation

## Code Highlights

Reference cycle with mutual strong references (causes memory leak):
```swift
class Traveler { var account: Account? }
class Account { var traveler: Traveler }  // strong → leak
```

Breaking the cycle with `weak`:
```swift
class Account { weak var traveler: Traveler? }  // safe
```

Unsafe pattern (relying on observed lifetime via force-unwrap):
```swift
func printSummary() {
    print("\(traveler!.name) has \(points) points")  // traveler may be nil!
}
```

Safe fix using `withExtendedLifetime`:
```swift
func test() {
    let traveler = Traveler(name: "Lily")
    let account = Account(traveler: traveler, points: 1000)
    traveler.account = account
    withExtendedLifetime(traveler) {
        account.printSummary()
    }
}
```

Better fix: eliminate the cycle by extracting shared data:
```swift
class PersonalInfo { var name: String }
class Traveler { var info: PersonalInfo; var account: Account? }
class Account { var info: PersonalInfo; var points: Int }
```

Replacing deinit side effects with `defer`:
```swift
func test() {
    let traveler = Traveler(name: "Lily", id: 1)
    defer { traveler.publishAllMetrics() }
    traveler.updateDestination("Big Sur")
    traveler.updateDestination("Catalina")
}
// deinit only does: assert(travelMetrics.published)
```

## Takeaways
- Swift object lifetimes are use-based — guaranteed to end at last use, but may extend longer in practice due to current ARC optimizations. Never rely on observed extensions.
- Weak/unowned references and deinitializer side effects are the two language features that make lifetimes observable; both require careful handling.
- The safest long-term fix for weak-reference bugs is to redesign class relationships to avoid cycles (tree structures, shared value types), rather than using `withExtendedLifetime`.
- The new "Optimize Object Lifetimes" build setting in Xcode 13 is a powerful tool to proactively find lifetime bugs before they surface in production.

---
_Source: WWDC21 Session 10216 page (abstract, chapter summaries, code samples, and resource links)._
