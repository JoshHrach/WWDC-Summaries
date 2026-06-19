# What's New in Swift
**WWDC26 · Session 262** · [Watch](https://developer.apple.com/videos/play/wwdc2026/262/)

_Platforms:_ Swift toolchain (Apple platforms, Linux, Windows, WebAssembly, Android, Embedded)

## Overview
This session surveys the most significant additions in Swift 6.3 and 6.4, organized into four themes: everyday language improvements, library updates, cross-platform expansion, and performance tuning. It serves as the annual state-of-the-language address, touching both ergonomic refinements for everyday app developers and deep low-level capabilities for systems programmers.

The library updates section covers the Swift standard library, Swift Testing, the new Subprocess package, and Foundation. Each area received meaningful additions that reduce boilerplate, improve correctness, and expand what is possible without leaving the Swift ecosystem.

The performance section introduces explicit optimizer controls and a substantially more powerful ownership system — including noncopyable and non-escapable types reaching parity with ordinary Swift types — enabling library authors and performance-critical application code to opt into zero-copy, allocation-free data structures.

## Key Topics

### Everyday Language Improvements (Swift 6.4)
- Optional parentheses in control flow and concurrency constructs removed as a requirement.
- Improved concurrency diagnostics: the compiler now catches unhandled errors in `Task {}` closures and warns about `try` without `await`.
- `weak let` bindings are now supported, allowing weak references in non-property contexts.
- `~Sendable` suppression on classes and `@unchecked Sendable` on subclasses to handle legacy class hierarchies.
- Memberwise initializers are now generated at multiple access levels: one matching the most restrictive stored property, one matching the struct's access level with default values for private properties.

### `anyAppleOS` Availability
A new availability platform token, `anyAppleOS`, condenses `macOS, iOS, watchOS, tvOS, visionOS` repetition into a single condition. Works with `@available`, `#available`, `#if os(...)`, and `@available(..., unavailable)` overrides.

### `@diagnose` Attribute
A new attribute that controls how specific diagnostic groups behave within a declaration scope. You can suppress deprecation warnings (`as: ignored`), promote a specific warning to an error, or control forthcoming Swift version diagnostics independently from project-wide settings.

### Module Selectors (`::`)
The double-colon syntax lets you unambiguously qualify a type or member with its module name, resolving ambiguity when multiple imported modules export identically named types or extension methods (e.g., `Rocket::SaturnV()` vs `GiftShopToys::SaturnV()`).

### Standard Library Updates
- **`withTaskCancellationShield`** — runs a closure even if the current task is cancelled, protecting cleanup and safety-critical code paths.
- **`Dictionary.mapKeyedValues`** — transforms dictionary values while retaining key access, replacing verbose `init(uniqueKeysWithValues:lazy.map:)` patterns.
- **`FilePath`** — a new cross-platform path type with component-aware parsing, available outside of Apple platforms. Handles macOS named-resource forks (e.g., `..namedresource/rsrc`) transparently.

### Swift Testing Updates
- **`Issue.record(_:severity:)`** — issues can now be recorded as `.warning` (non-failing) or `.error` (failing), enabling rich test output without aborting a test run.
- **`Test.cancel(_:)`** — dynamically cancels a test from within its body, replacing `XCTSkipIf`/`XCTSkipUnless`.
- **Flaky test repetition** — a new trait to automatically retry tests a configurable number of times.
- **XCTest interoperability** — `XCTAssert*` functions can now be called from Swift Testing tests, and `#expect`/`#require` can be used within `XCTestCase` methods.

### Subprocess 1.0
The Subprocess package reaches its 1.0 API with streaming line-by-line output via `AsyncSequence`, improved error handling with structured `SubprocessError` types, and cross-platform support including Windows.

### Foundation Updates
- **`ProgressManager`** — a new Swift-native progress reporting type built around structured concurrency. Supports `Subprogress` for hierarchical reporting, observation via `Observations {}`, and typed metadata keys (e.g., custom `deltaV` properties).
- Continued migration of Core Foundation internals to Swift, with measurable performance improvements to `Data`, `NSURL`, and `CFURL`.

### Cross-Platform: `@C` Attribute
A new `@C` attribute exposes Swift functions to C callers without a bridging header. The compiler generates a C-compatible symbol and validates that the function signature uses only C-representable types, enabling incremental Swift migration of existing C codebases.

### Swift-Java
The Swift-Java package now supports calling `async` and `throwing` Swift functions from Java, constrained generic extensions, and conforming Java classes to Swift protocols.

### WebAssembly & JavascriptKit
Swift compiles to WebAssembly. The JavascriptKit library received up to 40x faster safe bridging between Swift and JavaScript, unlocking practical client-side Swift web apps.

### Embedded Swift
The Embedded Swift language subset gains existential types (`any Protocol`), untyped `throws`, and improved DWARF debug info for coredump debugging on constrained hardware targets.

### Performance: Optimizer Control
- **`@inline(never)`** / **`@inline(always)`** — explicit control over per-function inlining decisions.
- **`@specialized(where T == SomeType)`** — request that the compiler emit a specialized copy of a generic function for a specific concrete type, even when called from a different module.

### Performance: Ownership System Expansion
- `Equatable`, `Comparable`, `Hashable`, `Codable`, and associated types now support `~Copyable` and `~Escapable` constraints.
- New **`Iterable`** protocol with a borrow-based for-loop over noncopyable elements, and `IterableIteratorProtocol` with `nextSpan` and `skip`.
- **`borrow`** and **`mutate`** computed-property accessors eliminate hidden copies in computed properties on noncopyable types.
- New standard library types: **`UniqueBox<Value: ~Copyable>`**, **`UniqueArray`**, **`Continuation`**, **`Ref<T>`**, and **`MutableRef<T>`** for safe, high-performance ownership patterns.
- `MutableRef` can hoist repeated subscript accesses (e.g., `[key, default:]`) out of loops to eliminate redundant copy-on-write traffic.

## APIs & Frameworks

**Language features (new/changed)**
- **[NEW]** `anyAppleOS` availability platform token
- **[NEW]** `@diagnose(DiagnosticGroup, as: ignored|warning|error, reason:)` attribute
- **[NEW]** Module selector syntax `ModuleName::TypeOrMember`
- **[NEW]** `weak let` bindings
- **[NEW]** `~Sendable` class suppression; `@unchecked Sendable` subclass annotation
- **[NEW]** Multiple-access-level memberwise initializer generation
- **[NEW]** `@C` attribute for C-callable Swift functions
- **[NEW]** `@inline(always)` / `@inline(never)` function attributes
- **[NEW]** `@specialized(where ...)` generic specialization attribute
- **[NEW]** `borrow` and `mutate` computed property accessors
- **[NEW]** `Iterable` / `IterableIteratorProtocol` protocols
- **[NEW]** Associated types support `~Copyable` and `~Escapable` constraints
- **[NEW]** `InlineArray` (fixed-size stack array, e.g., `[256 of Int]`)

**Standard Library (new types/methods)**
- **[NEW]** `withTaskCancellationShield(_:)`
- **[NEW]** `Dictionary.mapKeyedValues(_:)`
- **[NEW]** `FilePath` (cross-platform, component-aware)
- **[NEW]** `UniqueBox<Value: ~Copyable>`
- **[NEW]** `UniqueArray`
- **[NEW]** `Ref<T>` / `MutableRef<T>`
- **[NEW]** `Continuation` (ownership-safe continuation type)

**Swift Testing**
- **[NEW]** `Issue.record(_:severity:)` with `Issue.Severity.warning` / `.error`
- **[NEW]** `Test.cancel(_:)` — dynamic test cancellation
- **[NEW]** Flaky test repetition trait
- **[NEW]** XCTest ↔ Swift Testing two-way interoperability (`XCTAssert*` in `@Test`, `#expect`/`#require` in `XCTestCase`)

**Subprocess**
- **[NEW]** `Subprocess.run(_:input:output:error:)` with `.sequence` output mode
- **[NEW]** `execution.standardOutput.strings()` async sequence
- `SubprocessError` structured error type

**Foundation**
- **[NEW]** `ProgressManager(totalCount:)`
- **[NEW]** `Subprogress` / `progress.start(totalCount:)` / `stage.complete(count:)`
- **[NEW]** `Observations { }` observation block for `ProgressManager`
- Performance improvements: `Data`, `NSURL`, `CFURL`

**Swift-Java**
- `async` / `throws` Swift functions callable from Java
- Java class conformance to Swift protocols

## Code Highlights

**`anyAppleOS` before/after:**
```swift
// Before
@available(macOS 27, iOS 27, watchOS 27, tvOS 27, visionOS 27, *)
// After
@available(anyAppleOS 27, *)
```

**`@diagnose` suppression:**
```swift
@diagnose(DeprecatedDeclaration, as: ignored, reason: "Flying with surplus hardware")
func makeApolloSoyuzMission() -> Mission { ... }
```

**Module selectors:**
```swift
launchPadTechnician.HumanResources::fire()  // calls HumanResources.Employee.fire(), not Chemistry.Flammable.fire()
```

**`withTaskCancellationShield`:**
```swift
func sendSOS() {
    withTaskCancellationShield { radio.send(makeSOSPacket()) }
}
```

**`borrow` / `mutate` accessors:**
```swift
public var value: Value {
    borrow { valuePointer.pointee }
    mutate { &valuePointer.pointee }
}
```

**`MutableRef` hoisting:**
```swift
var countRef = MutableRef(&counts[key, default: 0])
for set in sets { if set.contains(key) { countRef.value += 1 } }
```

## Takeaways
- Adopt `anyAppleOS` immediately to drastically reduce platform availability boilerplate in cross-platform frameworks.
- Start writing new tests in Swift Testing today — the new XCTest interoperability mode removes the last barrier to incremental migration.
- Explore `@inline(always)` and `@specialized` for hot paths in generic library code where cross-module optimization is insufficient.
- Use `UniqueBox`, `UniqueArray`, and `MutableRef` when building performance-critical data structures that cannot afford hidden copies.

---
_Source: WWDC26 Session 262 page (abstract, chapter summaries, code samples, and resource links)._
