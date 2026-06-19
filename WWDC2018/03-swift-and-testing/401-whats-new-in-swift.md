# What's New in Swift
**WWDC18 · Session 401** · [Watch](https://developer.apple.com/videos/play/wwdc2018/401/)

_Platforms:_ iOS 12, macOS Mojave 10.14, watchOS 5, tvOS 12 (Swift 4.2)

## Overview
The annual Swift state-of-the-language session covering two major areas: community and open source developments, and the Swift 4.2 language/compiler changes shipping in Xcode 10. The session is presented by two Apple compiler engineers covering the road to ABI stability (Swift 5, early 2019), build performance improvements in Xcode 10, runtime optimizations, and a comprehensive tour of every new language feature in Swift 4.2.

Swift 4.2 is a major update focused on developer productivity: removing common boilerplate, improving safety, adding missing standard library primitives, and refining behavior of existing features. Xcode 10 is announced as the last release to support Swift 3 compatibility mode, making 4.2 migration the appropriate next step for all projects.

## Key Topics

### Swift Open Source Community
- Over 600 contributors, 18,000+ merged pull requests since late 2015
- Community-hosted CI nodes now supported for testing Swift on non-standard platforms
- Moved from mailing lists to Swift Forums (forums.swift.org) for all language evolution discussions
- Developer forums available for open source Swift library projects
- Swift Programming Language book relocated to docs.swift.org
- Apple actively participates in community conferences, meetups, and podcasts
- Swift Evolution: all Swift 4.2 proposals are community-visible on swift.org/swift-evolution; many designed and implemented by community contributors

### ABI Stability / Swift 5
- Swift 5 (early 2019) will achieve binary compatibility — compiled Swift modules will be interoperable at the binary level across compiler versions
- Apple will ship the Swift runtime in the OS, eliminating it from app bundles → smaller apps, faster launch, lower memory usage
- Progress tracked on ABI Stability dashboard at swift.org

### Build Performance (Xcode 10)
- Debug builds up to 2x faster on typical mixed Objective-C/Swift projects; pure Swift target can be 3x faster vs. Xcode 9
- Root cause: eliminated redundant cross-file compilation work within a Swift target (Swift's module-wide visibility previously caused duplicate work)
- Better parallelism across CPU cores
- **New Compilation Mode settings** (separated from Optimization Level):
  - **Debug**: default is Incremental (only rebuild changed files)
  - **Release**: default is Whole Module (all files together for maximum optimization opportunity)
  - Whole Module + No Optimization (previous stopgap for faster debug builds) is no longer needed in Xcode 10 and is slower due to preventing incremental builds

### Runtime Optimizations

**ARC Calling Convention Change**
- Swift 4.1: callee was responsible for releasing argument objects (extra retain/release traffic on every call)
- Swift 4.2: calling convention changed — caller retains responsibility, eliminating intermediate retain/release pairs on object arguments passed to multiple API calls
- Result: significant code size reduction and runtime performance improvement

**String Size Reduction**
- `String` reduced from 24 bytes to 16 bytes in Swift 4.2
- Small string optimization: strings ≤15 bytes are stored inline in the String value (no heap allocation) — similar to `std::string` SSO
- Memory and performance win for short, common strings

**New Optimization Level: Optimize for Size (`-Osize`)**
- New compiler flag; trades ~5% runtime performance for 10%–30% reduction in compiled machine code size
- Disables aggressive inlining and speculative devirtualization that trade code size for speed
- Useful for apps near cellular OTA download limits; try it if binary size matters more than raw speed

### Language Features — Swift 4.2

**`CaseIterable` Protocol [NEW]**
- Declare `enum MyEnum: CaseIterable {}` — compiler synthesizes `allCases: [MyEnum]` property automatically
- Eliminates hand-maintained `allCases` arrays; new enum cases are included automatically

**Conditional Conformances in Standard Library [NEW]**
- Already a language feature; now applied throughout the standard library
- `Array: Equatable where Element: Equatable` — arrays of Equatable elements are now Equatable
- `Array: Hashable where Element: Hashable`, `Optional: Equatable/Hashable`, `Dictionary: Equatable/Hashable where Value: Equatable/Hashable`
- Enables composing collections: `Set<[[Int?]]>` just works

**Synthesized `Equatable`/`Hashable` for Generic Types [NEW]**
- Previously available for non-generic types; now compiler can synthesize `Equatable`/`Hashable` for generic types with conditional conformances
- Example: `Either<Left, Right>` gets `Equatable` synthesis `where Left: Equatable, Right: Equatable`

**Redesigned `Hashable` Protocol [NEW]**
- Old requirement: `var hashValue: Int { get }` — developers wrote brittle, potentially insecure hash combining code
- New requirement: `func hash(into hasher: inout Hasher)` — feed fields to `Hasher` which handles combination internally
- `Hasher` uses a random seed per app launch — provides defense against hash-flooding denial-of-service attacks
- **Breaking change**: hash values and dictionary/set iteration order are no longer stable across app launches — fix any code that relied on stable hashing
- `SWIFT_DETERMINISTIC_HASHING=1` environment variable (set in scheme) disables random seed temporarily during migration

**Random Number Generation [NEW]**
- `Int.random(in: 1...6)` / `Float.random(in: 0..<1.0)` — unified, correctly uniform random number API on all numeric types
- `collection.randomElement()` — returns `Optional<Element>` (nil for empty collection)
- `collection.shuffled()` — returns new array with uniformly random permutation
- `RandomNumberGenerator` protocol — implement custom RNG; pass via `using:` parameter overloads
- Replaces error-prone C APIs (`arc4random_uniform`, etc.) that varied by platform

**`#if canImport(ModuleName)` Build Directive [NEW]**
- Expresses intent: "if I can import this module" rather than listing specific OS versions
- Cleaner cross-platform code sharing between iOS (UIKit) and macOS (AppKit)

**`#if targetEnvironment(simulator)` Build Directive [NEW]**
- Replaces `#if (arch(i386) || arch(x86_64)) && (os(iOS) || os(tvOS) || os(watchOS))` boilerplate
- Explicitly checks for simulator environment

**`#warning` and `#error` Build Directives [NEW]**
- `#warning("message")` — emits a compiler warning at build time (replaces `// FIXME:` with compiler enforcement)
- `#error("message")` — emits a compile-time error (enforces preconditions, e.g., "must configure before building for this platform")

**Implicitly Unwrapped Optionals (IUO) Refinement**
- Mental model clarified: IUOs are a declaration attribute, not a type; compiler uses IUO as plain Optional when context allows, force-unwraps only when required
- Fixed edge cases where IUOs nested in type aliases produced confusing behavior
- `typealias Foo = Int!; var x: [Foo]` now correctly treated as `[Int?]` with a warning
- Most code unaffected; check Swift.org blog post for edge cases

**Memory Exclusivity Checking Improvements [NEW in scope]**
- Static (compile-time) exclusivity checking now catches more cases in generic closures
- Runtime exclusivity checking now optionally available in **Release** builds (not just Debug)
- Future goal: enable runtime exclusivity checking by default in all builds (like bounds checking)
- Fix pattern: pass the captured value as a closure parameter rather than capturing it by reference when the value is being mutated by the caller

## APIs & Frameworks

**Swift 4.2 Language**
- `CaseIterable` protocol — `allCases: [Self]` synthesis
- Conditional `Equatable`/`Hashable` for `Array`, `Optional`, `Dictionary`, slices
- Synthesized `Equatable`/`Hashable` for generic types (conditional conformances)
- `Hashable.hash(into:)` — new requirement replacing `hashValue`
- `Hasher` — secure hash combiner with random seed
- `Int.random(in:)`, `Float.random(in:)`, `Double.random(in:)` — uniform random in range
- `Collection.randomElement()` → `Element?`
- `Collection.shuffled()` → `[Element]`
- `RandomNumberGenerator` protocol — custom RNG
- `#if canImport(Module)` — module-presence build condition
- `#if targetEnvironment(simulator)` — simulator build condition
- `#warning("message")`, `#error("message")` — build-time diagnostics
- `SWIFT_DETERMINISTIC_HASHING` — environment variable for deterministic hashing in tests

**Compiler Flags**
- `-Osize` / Optimization Level: Optimize for Size — new optimization mode

## Code Highlights

CaseIterable for exhaustive enum iteration:
```swift
enum Direction: CaseIterable {
    case north, south, east, west
}
print(Direction.allCases)  // [north, south, east, west]
```

Redesigned Hashable with Hasher:
```swift
struct City: Hashable {
    let name: String
    let state: String
    let population: Int

    static func == (lhs: City, rhs: City) -> Bool {
        return lhs.name == rhs.name && lhs.state == rhs.state
    }

    func hash(into hasher: inout Hasher) {
        hasher.combine(name)
        hasher.combine(state)
        // population intentionally excluded
    }
}
```

Uniform random number generation:
```swift
let roll = Int.random(in: 1...6)         // uniform, no modulo bias
let pct = Double.random(in: 0..<1.0)
let card = deck.randomElement()           // Optional<Card>
let shuffled = deck.shuffled()            // [Card]
```

Cross-platform import condition:
```swift
#if canImport(UIKit)
import UIKit
typealias PlatformColor = UIColor
#elseif canImport(AppKit)
import AppKit
typealias PlatformColor = NSColor
#else
#error("This platform is not supported")
#endif
```

Build-time warning for unfixed code:
```swift
#if targetEnvironment(simulator)
#warning("Simulator-only path: replace with real device implementation before shipping")
return mockData()
#else
return fetchFromNetwork()
#endif
```

## Takeaways
- Xcode 10 is the last release supporting Swift 3 mode — migrate to Swift 4.2 now; the migrator in Xcode handles most changes automatically.
- `CaseIterable` eliminates hand-maintained `allCases` arrays entirely; add it to any enum where you iterate all cases.
- The new `Hashable`/`Hasher` API is both safer and more correct — hash values are no longer stable across launches, so fix any code that stores or relies on them.
- Use `Int.random(in:)` instead of `arc4random_uniform()` — it is cross-platform, correctly uniform, and expressive.

---
_Source: WWDC18 Session 401 page (abstract, full transcript, and resource links)._
