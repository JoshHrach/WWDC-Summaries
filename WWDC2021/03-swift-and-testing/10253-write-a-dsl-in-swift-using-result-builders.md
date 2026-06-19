# Write a DSL in Swift Using Result Builders
**WWDC21 · Session 10253** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10253/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, watchOS 8, tvOS 15

## Overview
This session teaches how to design and implement embedded domain-specific languages (DSLs) in Swift using result builders (SE-0289, finalized in Swift 5.4). Using the Fruta sample app's smoothie recipe list as a running example, it walks through the motivation for DSLs, the mechanics of result builders, and the design principles behind modifier-style methods, trailing closure arguments, and the interplay between these features.

The session emphasizes that designing a Swift DSL is like designing a Swift API — the goal is the clearest possible use sites — and that result builders are a carefully scoped compile-time transformation that preserves the recognizability of Swift syntax.

## Key Topics

### What Is a DSL?
A **domain-specific language** is a miniature language tailored to a specific problem domain. A **standalone DSL** requires designing the language and writing a compiler for it. An **embedded DSL** uses the host language's (Swift's) features to add implicit behavior, making interop, tooling, and learning much easier. SwiftUI's view DSL is the canonical example.

### When to Create a DSL
- When plain Swift mechanics obscure the meaning of the code (e.g., appending to temporary arrays, juggling commas and brackets).
- When the best approach is to _describe_ something to another part of the code rather than write instructions.
- When the code will be maintained by people who are not primarily programmers.
- When a library or frequently-updated configuration will benefit many clients.
- Balanced against: DSLs take effort to design, implement, and learn.

### How Result Builders Work
A result builder type is marked with `@resultBuilder`. When applied as an attribute to a closure (via a function parameter) or a function body, the Swift compiler transforms the code: it captures each statement that produces a value into a temporary variable, then passes all of them to the builder's `buildBlock(_:)` static method, which returns the combined result. This is entirely a **compile-time transformation** and works on any OS version.

Supported builder methods:
- `buildBlock(_:)` — required; combines multiple results
- `buildOptional(_:)` — enables `if` without `else`
- `buildEither(first:)` / `buildEither(second:)` — enables `if`/`else` and `switch`
- `buildArray(_:)` — enables `for`-`in` loops
- `buildExpression(_:)` — enables type conversion or annotation
- `buildFinalResult(_:)` — post-processes the final result
- `buildLimitedAvailability(_:)` — handles `#available` checks

If a keyword like `if` or `for` is permitted, it behaves exactly as in normal Swift — result builders do not change Swift's control flow semantics, only what happens to statement results.

### Modifier-Style Methods
A modifier-style method returns a modified copy of `self` (or `self` wrapped in another type). Because they operate _before_ the result builder sees the value, multiple modifiers can be composed in a single statement. This is why SwiftUI uses modifiers so heavily — they combine naturally with result builders.

### DSL Design Process (Fruta Example)
The session redesigns the Fruta smoothie ingredient list:
- **Problem**: verbose `Measurement(value: 1.5, unit: .cups)` syntax, repeated struct name keywords, the need to manually build and filter arrays.
- **Solution**:
  - `SmoothieArrayBuilder` result builder collects `Smoothie` values from the function body, eliminating the explicit array.
  - `buildOptional` / `buildEither` support `if includingPaid { ... }` to replace manual filtering.
  - `Ingredient.measured(with:)` modifier-style method returns a `MeasuredIngredient`.
  - `MeasuredIngredient.scaled(by:)` modifier multiplies the quantity, enabling both recipe display and the recipe-scaling feature to reuse the same API.
  - A custom initializer with a trailing `@IngredientBuilder` closure replaces the flat array literal for ingredients, making the smoothie definition more compact and readable.

### Design Principles
1. Use trailing closure syntax to make DSL usage feel like native syntax.
2. Use modifier-style methods for transformations that should compose and be applied before the builder collects results.
3. Use result builders to collect statement results invisibly rather than requiring clients to build arrays manually.
4. Choose the design that produces the clearest call sites, not necessarily the easiest implementation.
5. Re-examine whether a DSL primitive can be reused in other contexts (e.g., `scaled(by:)` used both in the recipe list and in a per-ingredient UI row).

## APIs & Frameworks

- `@resultBuilder` attribute **[NEW in Swift 5.4]** — marks a type as a result builder
- `buildBlock(_:)` static method — required; assembles statement results
- `buildOptional(_:)` — opt-in support for `if` without `else`
- `buildEither(first:)` / `buildEither(second:)` — opt-in support for `if`/`else` and `switch`
- `buildArray(_:)` — opt-in support for `for`-`in` loops
- `buildExpression(_:)` — converts a statement expression before it is passed to `buildBlock`
- `buildFinalResult(_:)` — post-processes the builder's output
- `buildLimitedAvailability(_:)` — handles `#available` checks
- Trailing closure arguments — critical syntax for DSL readability
- Modifier-style methods — `func modifier(...) -> Self` or `func modifier(...) -> ModifiedSelf` pattern
- `@propertyWrapper` — related feature (covered in prior session); commonly combined with result builders in DSLs

## Code Highlights

Declaring a result builder:
```swift
@resultBuilder
enum SmoothieArrayBuilder {
    static func buildBlock(_ smoothies: Smoothie...) -> [Smoothie] {
        smoothies.filter { !$0.isAvailableForPurchase || includePaid }
    }
    static func buildOptional(_ component: [Smoothie]?) -> [Smoothie] {
        component ?? []
    }
    static func buildEither(first component: [Smoothie]) -> [Smoothie] { component }
    static func buildEither(second component: [Smoothie]) -> [Smoothie] { component }
}
```

Using the result builder with `if` control flow:
```swift
static func all(includingPaid: Bool = true) -> [Smoothie] {
    SmoothieArrayBuilder.buildBlock(
        // The compiler transforms the body:
    )
}

@SmoothieArrayBuilder
static func all(includingPaid: Bool = true) -> [Smoothie] {
    Lemonberry()
    Tropical()
    if includingPaid {
        PBJ()
        TropicalBlue()
    }
}
```

Modifier-style ingredient amounts:
```swift
// Instead of: Measurement(value: 1.5, unit: UnitVolume.cups)
Ingredient.oranges.measured(with: .cups).scaled(by: 1.5)
Ingredient.avocado.measured(with: .cups).scaled(by: 0.2)
```

Applying a result builder to a closure parameter:
```swift
struct Smoothie {
    init(id: Smoothie.ID,
         title: LocalizedStringKey,
         description: LocalizedStringKey,
         @IngredientBuilder makeIngredients: () -> [MeasuredIngredient]) {
        self.ingredients = makeIngredients()
        // ...
    }
}
```

## Takeaways

- Result builders are a compile-time feature that transforms statement results into a single combined value; they do not alter Swift's control-flow semantics, so `if`/`for`/`switch` work exactly as clients expect.
- The combination of trailing closures + result builders + modifier-style methods is the recipe for Swift embedded DSLs — each feature addresses a different readability concern.
- Design from the call site outward: write the usage code you want to see, then figure out what builder methods and types make it compile.
- DSLs are not always the right answer — if standard Swift methods and array literals are nearly as readable, prefer those; use a DSL when the boilerplate genuinely overwhelms the information.

---
_Source: WWDC21 Session 10253 page (abstract, chapter summaries, code samples, and resource links)._
