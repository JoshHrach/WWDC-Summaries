# What's New in Swift
**WWDC22 · Session 110354** · [Watch](https://developer.apple.com/videos/play/wwdc2022/110354/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, tvOS 16, watchOS 9, Linux

## Overview
Swift 5.7 delivers improvements across five areas: community and ecosystem growth (Linux native toolchains, Swift Mentorship Program year two), Swift Package Manager enhancements (TOFU security, command and build-tool plug-ins, module disambiguation), performance improvements (5–25% faster builds, dramatically faster type-checking, protocol-checking cached at launch), concurrency model maturation (distributed actors, Async Algorithms package, actor prioritization, new Instruments view), and language expressiveness (Swift Regex and RegexBuilder, `any`/`some` keyword clarity, optional shorthand unwrapping, improved closure type inference, and improved C interop for pointer conversions).

## Key Topics

### Swift Packages
- **TOFU (Trust On First Use)** — package fingerprint is recorded on first download; subsequent downloads validate it; mismatches are an error
- **Command plug-ins** — `CommandPlugin` protocol; runs tools like docC or formatters/linters on demand via Xcode menu or `swift package` CLI; can be granted permission to modify package files
- **Build tool plug-ins** — `BuildToolPlugin` protocol; injects build steps (source generation, custom file processing) into the build system sandbox; sandboxed by default
- **Module disambiguation** — `moduleAliases` in `Package.swift` renames a conflicting module from an external package; resolves "two packages define the same module name" errors

### Performance
- **Build times**: 5–25% faster on 10-core iMac thanks to Swift Driver running in-process in the Xcode build system (enables better parallelization)
- **Type-checking**: rewritten generics system core; pathological protocol sets that took 17 seconds now check in under a second
- **App launch**: protocol conformance checks now cached; up to 2× faster launches in iOS 16 for apps with heavy protocol use

### Concurrency
- **Concurrency back-deployment** — async/await and actors back-deployable to iOS 13 / macOS Catalina by bundling the Swift 5.5 concurrency runtime
- **Distributed actors [NEW]** — `distributed actor` keyword; `distributed func` for functions that may execute on a remote machine; calls are `try await`; open source Distributed Actors package built on SwiftNIO with SWIM consensus protocol
- **Swift Async Algorithms package [NEW]** — open source package for combining and grouping `AsyncSequence` values (zip, merge, chunking, etc.)
- **Actor prioritization** — actors execute highest-priority work first; priority-inversion prevention built in
- **Swift Concurrency instrument [NEW]** — "Swift Tasks" and "Swift Actors" instruments in Xcode; Task Forest visualization of parent-child task relationships; statistics on concurrent task counts

### Language Improvements
- **Optional shorthand `if let` / `guard let`** — `if let someOptional { ... }` without repeating the name on the right-hand side; works with `guard let` and `while let`
- **Multi-statement closure type inference** — Swift now infers closure result types from multi-statement closures with do-catch, if-else, and other control flow
- **Improved C pointer interop** — Swift now allows pointer-type conversions that are legal in C when calling imported C functions, eliminating previously-required `withMemoryRebound` boilerplate
- **`any` keyword for existential types** — clarifies where an existential "box" is used vs. a generic constraint; not mandatory but encouraged; `any Mailmap` vs. a generic `<T: Mailmap>` parameter
- **Existential types with `Self`/associated types** — protocols with `Self` requirements or associated types (e.g., `Equatable`, `Collection`) can now be used as `any` types
- **Primary associated types** — `Collection<Element>` syntax constrains the primary associated type inline; e.g., `any Collection<MailmapEntry>`
- **`some` keyword shorthand for generics** — `some Collection<MailmapEntry>` as a parameter type is equivalent to a single-use generic parameter; as easy to write as `any` but with generic semantics and performance
- **Swift Regex [NEW]** — regex literals (`/.../`) with Unicode correctness; available on macOS 13 / iOS 16+
- **RegexBuilder [NEW]** — SwiftUI-style DSL for constructing regexes; `Regex { }`, `Capture`, `OneOrMore`, `ZeroOrMore`, `ChoiceOf`, `Optionally`, `Anchor`; supports custom `RegexComponent` types; Foundation types (date formats, etc.) integrate directly

## APIs & Frameworks

**Swift Standard Library**
- Shorthand `if let foo { }` / `guard let foo else { }` **[NEW]**
- Multi-statement closure result type inference **[NEW]**
- C-family pointer conversion at call sites **[NEW]**
- `any` keyword for existential types **[NEW]**
- Primary associated types syntax `Protocol<AssocType>` **[NEW]**
- `some Protocol` shorthand for single-use generic params **[NEW]**

**Swift Regex (stdlib + RegexBuilder)**
- Regex literal `/.../` **[NEW]** — Unicode-correct; requires macOS 13 / iOS 16
- `RegexBuilder` framework **[NEW]** — `Regex { }` builder with `Capture`, `OneOrMore`, `ZeroOrMore`, `Optionally`, `ChoiceOf`, `Anchor`, `.horizontalWhitespace`, `.noneOf(_:)`
- `RegexComponent` protocol **[NEW]** — create reusable regex components
- `String.prefixMatch(of:)`, `String.wholeMatch(of:)`, `String.firstMatch(of:)` **[NEW]**
- Strongly typed captures from regex matches **[NEW]**

**Swift Package Manager**
- `CommandPlugin` protocol **[NEW]** — Xcode/CLI-invokable build tool plug-ins
- `BuildToolPlugin` protocol **[NEW]** — sandboxed build-step plug-ins
- `moduleAliases` in `Package.swift` **[NEW]** — module name disambiguation
- TOFU fingerprint security **[NEW]** — recorded on first download, validated on subsequent downloads

**Swift Concurrency**
- `distributed actor` **[NEW]** — actor that may reside on a remote machine
- `distributed func` **[NEW]** — method requiring `try await` at call site
- Swift Async Algorithms package **[NEW]** — open source; `zip`, `merge`, `chunks`, `debounce`, etc. on `AsyncSequence`
- Swift Concurrency Instruments: Swift Tasks + Swift Actors **[NEW]**

## Code Highlights

```swift
// Shorthand optional unwrapping (Swift 5.7)
if let workingDirectoryMailmapURL {
    mailmapLines = try String(contentsOf: workingDirectoryMailmapURL).split(separator: "\n")
}
guard let workingDirectoryMailmapURL else { return }

// some keyword for generic parameters
func addEntries(_ entries: some Collection<MailmapEntry>, to mailmap: inout some Mailmap) { ... }
```

```swift
// Swift Regex literal
let regex = /\h*([^<#]+?)??\h*<([^>#]+)>\h*(?:#|\Z)/
guard let match = line.prefixMatch(of: regex) else { throw MailmapError.badLine }
return MailmapEntry(name: match.1, email: match.2)

// RegexBuilder DSL equivalent
import RegexBuilder
let regex = Regex {
    ZeroOrMore(.horizontalWhitespace)
    Optionally { Capture(OneOrMore(.noneOf("<#"))) }.repetitionBehavior(.reluctant)
    ZeroOrMore(.horizontalWhitespace)
    "<"
    Capture(OneOrMore(.noneOf(">##")))
    ">"
    ZeroOrMore(.horizontalWhitespace)
    ChoiceOf { "#"; Anchor.endOfSubjectBeforeNewline }
}
```

```swift
// Distributed actor (Swift 5.7)
distributed actor Player {
    var gameState: GameState
    distributed func makeMove() -> GameMove { ... }
}
func endOfRound(players: [Player]) async throws {
    for player in players {
        let move = try await player.makeMove()
    }
}
```

## Takeaways

- Adopt `if let foo { }` shorthand for optional unwrapping to eliminate repetitive same-name bindings; use `some Protocol` parameters instead of `any Protocol` wherever performance matters — they're now equally easy to write.
- Use Swift Regex literals for terse pattern matching and RegexBuilder for readable, maintainable patterns; combine them freely — Foundation types (dates, currencies) integrate directly into regex builders.
- Migrate to Swift Package Manager plug-ins (command and build-tool) for documentation generation, code formatting, and source generation; they integrate with Xcode and eliminate separate shell scripts.
- Adopt distributed actors to build back-end services in Swift; the open source Distributed Actors package provides clustering over SwiftNIO; use the new Swift Concurrency Instruments to profile task and actor performance.

---
_Source: WWDC22 Session 110354 page (abstract, chapter summaries, code samples, and resource links)._
