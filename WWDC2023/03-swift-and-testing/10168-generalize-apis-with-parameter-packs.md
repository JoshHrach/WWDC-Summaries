# Generalize APIs with Parameter Packs
**WWDC23 · Session 10168** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10168/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, visionOS 1, watchOS 10, tvOS 17

## Overview
Swift 5.9 introduces parameter packs, a first-class language feature that allows generic code to abstract over both types and the number of arguments simultaneously. Before parameter packs, the only way to write APIs that accepted a variable number of differently-typed arguments — while preserving each argument's specific static type — was to write a series of overloads with an arbitrary upper limit on argument count. Parameter packs eliminate this pattern entirely.

A type parameter pack (declared with `each`) allows a function or type to hold a collection of type placeholders. A value parameter pack allows a matching collection of values to be passed and iterated. The `repeat` keyword introduces a repetition pattern that expands across every element in a pack — at the type level for return types and stored properties, and at the value level for expressions and function calls.

This capability is directly used by Swift's standard library and enables API authors to write a single generic function that previously required four or more overloads, with no artificial ceiling on argument count.

## Key Topics

### The Problem: Overload Proliferation
- Variadic parameters accept any count of a single type but cannot map argument count to return type arity or preserve heterogeneous static types.
- Without parameter packs, APIs that need to return a tuple whose arity matches the number of arguments (e.g., parallel query functions) require one overload per supported argument count, with an enforced upper limit.

### Type Parameter Packs and Value Parameter Packs
- Declare a type parameter pack with `each`: `func query<each Payload>(...)`.
- A type pack holds zero or more individual types; a value pack holds corresponding values.
- Each type and value are positionally aligned within their respective packs.
- Singular naming convention recommended: `each Payload` not `each Payloads`.

### Repetition Patterns
- `repeat` followed by a pattern type or expression expands the pattern once per pack element.
- `each` inside a pattern is replaced with each individual pack element during expansion.
- At type level: `repeat Request<each Payload>` expands to `Request<Bool>, Request<Int>, Request<String>` for a three-element pack.
- At value level: `(repeat (each item).evaluate())` iterates over every item in the value pack and collects results into a tuple.
- Repetition patterns can be used in: parenthesized tuple types, function parameter lists, generic argument lists, and value expressions.

### Constraints on Parameter Packs
- Conformance requirements: `<each Payload: Equatable>` applies `Equatable` to every element.
- `where` clause form: `where repeat each Payload: Equatable`.
- Minimum argument count: add a leading non-pack type parameter and corresponding parameter before the pack parameters.

### Implementing Code with Parameter Packs
- Store a value pack as a tuple property: `var item: (repeat each Request)`.
- Iterate over a pack in an expression using `repeat (each item).evaluate(each input)`.
- Pack iteration integrates with Swift's control flow: use `throws` to break out of iteration early; wrap iteration in do-catch to convert errors to optional returns.

## APIs & Frameworks
- Swift 5.9 **[NEW]** — language version introducing parameter packs
- `each` keyword **[NEW]** — declares a type parameter pack or references pack elements
- `repeat` keyword **[NEW]** — introduces a repetition pattern that expands over all pack elements
- Type parameter pack **[NEW]** — `<each T>` syntax in generic parameter list
- Value parameter pack **[NEW]** — `_ item: repeat Request<each Payload>` parameter syntax
- Repetition pattern (type level) **[NEW]** — `repeat Request<each Payload>` in type positions
- Repetition pattern (value level) **[NEW]** — `repeat (each item).evaluate()` in expression positions
- Pack conformance constraint **[NEW]** — `<each Payload: Protocol>` or `where repeat each Payload: Protocol`
- Minimum pack length pattern **[NEW]** — leading non-pack parameter plus pack parameter

## Code Highlights

Single generic function replacing four overloads:
```swift
func query<each Payload>(_ item: repeat Request<each Payload>) -> (repeat each Payload) {
    return (repeat (each item).evaluate())
}

// Works with any number of arguments:
let result  = query(Request<Int>())
let results = query(Request<Int>(), Request<String>(), Request<Bool>())
```

Parameter pack with conformance constraint and minimum count:
```swift
func query<FirstPayload: Equatable, each Payload: Equatable>(
    _ first: Request<FirstPayload>, _ item: repeat Request<each Payload>
) -> (FirstPayload, repeat each Payload)
```

Generic type storing a value pack and supporting different input/output types:
```swift
protocol RequestProtocol {
    associatedtype Input
    associatedtype Output
    func evaluate(_ input: Input) throws -> Output
}

struct Evaluator<each Request: RequestProtocol> {
    var item: (repeat each Request)

    func query(_ input: repeat (each Request).Input) -> (repeat (each Request).Output)? {
        do {
            return (repeat try (each item).evaluate(each input))
        } catch {
            return nil
        }
    }
}
```

## Takeaways
- Parameter packs eliminate the overloading pattern for functions/types that need to accept heterogeneous, variably-counted typed arguments — replacing N overloads with a single declaration.
- Use `each` to declare type parameter packs and reference individual pack elements; use `repeat` to expand a pattern over all elements.
- Constraints and `where` clauses on packs apply to every element in the pack, just as with ordinary generics.
- Pack iteration integrates naturally with Swift error handling (`throws`/`try`) for early exit from repetition patterns.

---
_Source: WWDC23 Session 10168 page (abstract, chapter summaries, code samples, and resource links)._
