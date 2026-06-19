# Consume Noncopyable Types in Swift
**WWDC24 · Session 10170** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10170/)

_Platforms:_ Swift 6, iOS 18, macOS 15, Linux, Windows

## Overview
Noncopyable types (introduced in Swift 5.9 as `~Copyable`) reach full maturity in Swift 6, gaining support in generics, protocols, and existentials. This session explains the ownership model that noncopyable types enforce, how to think about consuming and borrowing values, and how to write generic code that works with both copyable and noncopyable types using the new `~Copyable` constraint syntax.

The talk is structured as a progression: first building intuition around uniqueness and ownership, then showing how the compiler enforces those rules, then demonstrating the expanded Swift 6 generics support. The running example is a file descriptor abstraction — something that must not be accidentally duplicated.

## Key Topics
- **The Copyable protocol** — all Swift types implicitly conform to `Copyable`; suppressing that conformance with `~Copyable` gives the compiler ownership information to enforce uniqueness.
- **Consuming vs borrowing** — `consuming` parameters take ownership (no copy made, caller loses the value); `borrowing` parameters read without ownership transfer; `inout` grants temporary exclusive write access.
- **`consume` expression** — explicitly move a value out of scope, invalidating the original binding without a copy.
- **`discard` for deinit suppression** — `discard self` in a noncopyable type's `deinit` or a `consuming` method prevents the compiler from running the deinitializer.
- **Generics with `~Copyable`** — Swift 6 allows type parameters constrained to `~Copyable`; protocol conformances can be written for both copyable and noncopyable types.
- **Noncopyable enums** — enums with associated values can be `~Copyable`; `switch` on them consumes the enum, moving associated values out without copies.

## APIs & Frameworks

**Swift Language**
- **[NEW]** `~Copyable` suppression syntax — `struct Foo: ~Copyable` opts a type out of implicit `Copyable` conformance
- **[NEW]** `Copyable` protocol — the implicit protocol all types conform to by default; first-class in Swift 6 generics
- `consuming` parameter modifier — caller transfers ownership; function may mutate or discard the value
- `borrowing` parameter modifier — read-only, non-owning access; no copy made
- `inout` parameter modifier — exclusive mutable access for the duration of the call
- **[NEW]** `consume` expression — `let x = consume y` moves `y` into `x`, invalidating `y`
- **[NEW]** `discard` statement — `discard self` in a noncopyable `deinit` or consuming method skips deinitializer execution
- **[NEW]** Generic `~Copyable` constraint — `func foo<T: ~Copyable>(_ value: consuming T)` writes code generic over both copyable and noncopyable types
- **[NEW]** Protocol `~Copyable` adoption — `protocol P: ~Copyable` allows noncopyable conforming types
- **[NEW]** Noncopyable enum support — `enum E: ~Copyable { case x(SomeNoncopyable) }` with `switch` consuming associated values

**Swift Standard Library**
- **[NEW]** `Optional` extended to support `~Copyable` wrapped types in Swift 6
- **[NEW]** `Result` extended to support `~Copyable` success/failure types in Swift 6
- **[NEW]** `Unsafe[Mutable]Pointer` extended for noncopyable pointee types

## Code Highlights
Define a noncopyable file descriptor:

```swift
struct FileDescriptor: ~Copyable {
    private var fd: Int32

    init(opening path: String) throws {
        fd = open(path, O_RDONLY)
    }

    consuming func close() {
        Foundation.close(fd)
        discard self          // prevent deinit from double-closing
    }

    deinit { Foundation.close(fd) }
}
```

Generic function accepting both copyable and noncopyable types:

```swift
func process<T: ~Copyable>(_ value: consuming T) { … }
```

## Takeaways
- Annotate resource handles (file descriptors, locks, GPU command buffers) as `~Copyable` to get compiler-enforced unique ownership at zero runtime cost.
- Use `borrowing` for read-only access to avoid accidental copies in hot paths, even on copyable types.
- Swift 6's generic `~Copyable` support means library authors can write a single generic algorithm that works for both kinds of types — no overloading required.
- `discard self` is the escape hatch to skip `deinit` after a consuming method has already performed cleanup; use it to prevent double-free bugs.

---
_Source: WWDC24 Session 10170 page (abstract, chapter summaries, code samples, and resource links)._
