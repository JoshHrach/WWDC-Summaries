# A Swift Tour: Explore Swift's Features and Design
**WWDC24 · Session 10184** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10184/)

_Platforms:_ iOS, iPadOS, macOS, tvOS, watchOS, visionOS, Linux, Windows (cross-platform)

## Overview
This session provides a comprehensive tour of the Swift programming language's core features and design philosophy, using the construction of a multi-component Swift package as its running example. The package includes a library for a social graph data model, an HTTP server using the open-source Hummingbird framework, and a command-line client using swift-argument-parser.

The session covers value types, error handling, optionals, code organization with modules and packages, classes and automatic reference counting, protocols and generics, Swift concurrency (Tasks, async/await, actors), and language extensibility features like property wrappers, result builders, and macros. The Swift 6 language mode's complete data-race safety is introduced in context of the server example.

Presented by Allan Shortlidge (Swift compiler team), this is aimed at both newcomers and experienced developers who want a coherent mental model of Swift's design decisions. Tools shown run on macOS (Xcode) and Ubuntu/Windows (VS Code), reinforcing Swift's cross-platform nature.

## Key Topics
- **Value types** — `struct`, arrays, and other value-semantic types copy on assignment; contrasted with reference types
- **Errors and optionals** — `throws`/`try`/`do-catch`, associated values on enum errors, optional binding with `if let`
- **Code organization** — Swift modules, packages, Swift Package Manager, Swift Package Index
- **Classes and ARC** — single inheritance, `override`, Automatic Reference Counting, weak/unowned to break cycles
- **Protocols and generics** — `protocol`, `extension`, `Collection` algorithms (map, filter, reduce), constrained extensions
- **Concurrency** — `Task`, `async`/`await`, `actor` for shared mutable state, Swift 6 complete data-race safety
- **Extensibility** — property wrappers (`@Argument`), result builders (`Regex`), macros

## APIs & Frameworks
### Swift Standard Library
- `struct` — value-semantic composite types; automatically value types when composed of value types
- `enum` — sum types; associated values for rich error context; conforming to `Error`
- `protocol` — abstract requirements; conformance via `extension` on any type
- `Collection` — common interface for `Array`, `Set`, `Dictionary`, `String`; algorithms: `map`, `compactMap`, `flatMap`, `filter`, `reduce`
- Optional (`?`) — `if let`, `guard let`, force-unwrap (`!`), `as?` conditional cast
- `throws` / `try` / `do`/`catch` — structured error propagation
- Access control — `private`, `internal` (default), `package`, `public`
- `weak var` / `unowned let` — non-retaining references to break ARC cycles

### Swift Concurrency
- `Task` — lightweight unit of concurrent execution; can cancel and await results
- `async` / `await` — function suspension markers
- **`actor`** — reference type with serialized access; eliminates data races on shared mutable state
- **Swift 6 language mode** — full data-race safety verified at compile time; `Sendable` protocol

### Swift Package Manager
- `Package.swift` with `swift-tools-version: 6.0`
- `.library`, `.target`, `.testTarget`, `.executableTarget` products
- Open-source dependencies: `swift-testing`, `Hummingbird` (HTTP server), `swift-argument-parser`

### Property Wrappers / Result Builders / Macros
- `@Argument` (swift-argument-parser) — command-line argument binding via property wrapper
- `@main` + `AsyncParsableCommand` — entry point for async command-line tools
- **Result builders** — `Regex { ... }` syntax from `RegexBuilder`
- Swift macros — compiler plugins transforming syntax trees (see "Expand on Swift macros")

## Code Highlights
```swift
// Value types — structs copy on assignment
struct User {
    let username: String
    var isVisible: Bool = true
    private(set) var friends: [String] = []

    mutating func addFriend(username: String) throws {
        guard username != self.username else { throw SocialError.befriendingSelf }
        guard !friends.contains(username) else {
            throw SocialError.duplicateFriend(username: username)
        }
        friends.append(username)
    }
}

// Protocols + constrained generic extension
extension Collection where Element: Hashable {
    func uniqued() -> [Element] { Array(Set(self)) }
}

// Actor for safe shared mutable state (Swift 6)
actor UserStore {
    var allUsers: [String: User] = [:]
    func friendsOfFriends(_ username: String) throws -> [String] { ... }
}

// Async route handler with actor access
router.get("friendsOfFriends") { request, context -> [String] in
    let username = try request.queryArgument(for: "username")
    return try await UserStore.shared.friendsOfFriends(username)
}
```

## Takeaways
- Swift's emphasis on value types and immutability makes code easier to reason about, especially in concurrent contexts
- Protocols and generics provide more flexible polymorphism than class inheritance and work with both value and reference types
- Swift 6's actor-based concurrency model catches data races at compile time—adopt it incrementally per module
- The cross-platform Swift ecosystem (Linux, Windows, embedded) via Swift Package Manager makes Swift a viable systems and server language, not just an Apple-platform language

---
_Source: WWDC24 Session 10184 page (abstract, chapter summaries, code samples, and resource links)._
