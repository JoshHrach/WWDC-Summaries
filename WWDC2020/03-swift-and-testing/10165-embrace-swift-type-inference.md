# Embrace Swift Type Inference
**WWDC20 · Session 10165** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10165/)

_Platforms:_ iOS, iPadOS, macOS, watchOS, tvOS (Swift 5.3 / Xcode 12)

## Overview
Swift's type inference lets you omit explicit type annotations when the compiler can determine the correct types from surrounding context, enabling concise and readable code without sacrificing type safety. This session explains how the type inference algorithm works, how it handles generic types (a common case in SwiftUI code), and how Swift 5.3's new integrated error-tracking architecture produces better, more actionable compiler diagnostics with breadcrumb notes.

The session uses a SwiftUI smoothie-ordering app called Fruta as a running example, building a generic `FilteredList` view and demonstrating how the compiler solves the "type inference puzzle" step by step. It then shows how typing mistakes trigger targeted error messages and notes that connect errors across files.

## Key Topics

### What Type Inference Does
Type inference allows the compiler to deduce types that are omitted from source code. The simplest case is a variable with no explicit type annotation; the compiler infers the type from the assigned value. The benefit compounds with generic code: a call site can omit all generic type parameters, key-path base types, closure parameter types, and more, as long as the surrounding context provides enough clues.

### Solving the Type Inference Puzzle
The compiler treats type inference as a constraint-solving puzzle. Each argument at a call site is a clue. The algorithm fills in placeholders one at a time; filling in one placeholder often reveals the type for the next:

1. The `data` argument (an `[Element]`-typed parameter) is matched against `smoothies: [Smoothie]` → infers `Element = Smoothie`.
2. `Element = Smoothie` fills in the key-path base type → `\.title` resolves to `KeyPath<Smoothie, String>` → infers `FilterKey = String`.
3. The `@ViewBuilder` trailing closure's return type is `SmoothieRowView` → infers `RowContent = SmoothieRowView`.

No angle-bracket type parameter list, no explicit key-path base type, no closure type annotation — the call site stays clean.

### Generic Code and SwiftUI
SwiftUI views compose heavily generic types, making type inference essential for readable DSL code. A generic `FilteredList<Element, FilterKey, RowContent>` view demonstrates how three type parameters can all be inferred from arguments: an `[Element]` array, a `KeyPath<Element, FilterKey>`, an `(FilterKey) -> Bool` closure, and a `@ViewBuilder (Element) -> RowContent` closure. The `@ViewBuilder` attribute enables SwiftUI DSL syntax in the trailing closure and causes the closure's return type to become the inferred concrete type for `RowContent`.

### Integrated Error Tracking (Swift 5.3 / Xcode 12)
When a clue leads the type inference algorithm to a dead end (e.g., a wrong key-path property infers `FilterKey = Bool`, which lacks `hasSubstring`), the compiler must report what went wrong. Swift 5.3 integrates error tracking directly into the type inference algorithm:

- **Errors are recorded** as the algorithm runs, not just at the end.
- **Heuristics continue inference** after an error, allowing the compiler to gather additional context.
- **Notes** are emitted alongside errors, pointing to other files or locations where the inferred concrete type originated — "breadcrumbs" that connect an error at a call site to the generic declaration that imposed the violated constraint.
- **Fix-its** are provided where the compiler can automatically correct the mistake (e.g., suggesting `$searchPhrase` when a non-binding `String` is passed where a `Binding<String>` is expected).

This approach was introduced for many messages in Swift 5.2 (Xcode 11.4) and extended to all expression-level error messages in Swift 5.3 (Xcode 12).

### Xcode Workflow Tips
- Add a build-failure behavior (Xcode → Behaviors → Edit Behaviors) to automatically show the Issue Navigator when a build fails — see all errors across the project at once.
- Use `Option+Shift+click` on a compiler note to open the target file in a split editor alongside the current file, allowing side-by-side inspection of the error and the relevant generic declaration.
- Use `Option+click` (QuickHelp) on any expression to see the concrete type the compiler inferred for it.

## APIs & Frameworks

### Swift 5.3 / Xcode 12
- Type inference for generic type parameters — inferred from call-site argument types and key-path literals
- `@ViewBuilder` — function-builder attribute; return type of the closure becomes the inferred concrete type for the generic `RowContent` parameter
- `KeyPath<Root, Value>` — key-path literal type; base type inferred from surrounding generic constraints
- Integrated error-tracking diagnostics **[improved in Swift 5.3]** — compiler notes link errors to inferred concrete types in other files
- Fix-its for common mistakes (e.g., `$` prefix for `Binding` parameters)
- `Identifiable` protocol — conformance required by `FilteredList`; missing conformance reported via a cross-file compiler note

## Code Highlights

Generic `FilteredList` with three inferred type parameters:
```swift
public struct FilteredList<Element, FilterKey, RowContent>: View
    where Element: Identifiable, RowContent: View {

    private let data: [Element]
    private let filterKey: KeyPath<Element, FilterKey>
    private let isIncluded: (FilterKey) -> Bool
    private let rowContent: (Element) -> RowContent

    public init(
        _ data: [Element],
        filterBy key: KeyPath<Element, FilterKey>,
        isIncluded: @escaping (FilterKey) -> Bool,
        @ViewBuilder rowContent: @escaping (Element) -> RowContent
    ) { ... }
}
```

Call site — all three type parameters (`Element`, `FilterKey`, `RowContent`) inferred by the compiler:
```swift
FilteredList(
    smoothies,                           // infers Element = Smoothie
    filterBy: \.title,                   // infers FilterKey = String
    isIncluded: { title in title.hasSubstring(searchPhrase) }
) { smoothie in
    SmoothieRowView(smoothie: smoothie)  // infers RowContent = SmoothieRowView
}
```

## Takeaways
- Type inference removes the need to explicitly spell out generic type parameters, key-path base types, and closure parameter types — write the arguments, and the compiler fills in the rest.
- The compiler solves inference as a sequential constraint puzzle: each resolved placeholder provides more information for the next, so argument order in the initializer signature matters.
- Swift 5.3's integrated error tracking records errors mid-inference and emits cross-file breadcrumb notes — use the Issue Navigator and split-editor workflow to connect a call-site error to its origin in a generic declaration.
- QuickHelp (`Option+click`) reveals the concrete type the compiler inferred for any expression, useful for debugging type inference failures.

---
_Source: WWDC20 Session 10165 page (abstract, transcript, and code samples)._
