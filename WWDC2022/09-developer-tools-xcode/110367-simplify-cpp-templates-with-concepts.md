# Simplify C++ templates with concepts
**WWDC22 · Session 110367** · [Watch](https://developer.apple.com/videos/play/wwdc2022/110367/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, tvOS 16 (no minimum deployment target for C++20)

## Overview
Xcode 14 adds full support for C++20 features, with this session focusing on two: **concepts** (type constraints for templates) and improved **`constexpr`** support (compile-time code evaluation). Concepts replace ad-hoc documentation comments and fragile `enable_if` hacks with first-class compiler-enforced type constraints that produce clear error messages pointing to the call site rather than deep inside template instantiations. The session builds from standard library concepts to custom concept authoring, then covers `constexpr` function and variable annotations to move initialization work to compile time and reduce app launch overhead.

## Key Topics

### Why Templates Need Concepts
- Without constraints, template type mismatches surface as errors inside template code, not at the call site — hard to diagnose.
- Prior to C++20: developers used `enable_if`, documentation comments, or obscure parameter names to communicate type requirements.
- C++20 concepts: explicit, compiler-enforced constraints on template parameters; errors point to the call site with readable messages.

### Using Standard Library Concepts (`<concepts>` header)
The C++ standard library provides a `<concepts>` header with core language concepts:
- **Type categories**: `integral`, `floating_point`, `signed_integral`, `unsigned_integral`
- **Lifecycle**: `constructible_from`, `default_initializable`, `move_constructible`, `copy_constructible`, `destructible`
- **Comparison**: `equality_comparable`, `totally_ordered`, `three_way_comparable`
- **Conversion**: `convertible_to`, `same_as`, `common_with`
- **Callable**: `invocable`, `regular_invocable`, `predicate`

Three syntaxes for applying a concept to a template:
1. **Concept instead of `class/typename`**: `template<std::integral T>`
2. **`requires` clause**: `template<typename T> requires std::equality_comparable<T> && std::default_initializable<T>`
3. **Abbreviated function template**: `void f(std::integral auto x)`

### Creating Custom Concepts with `requires` Expressions
- Identify which operations the generic code actually performs on the type parameter.
- Use a `requires` expression with an argument list to declare test values and a body of **requirements**:
  - **Simple expression requirement**: `{ shape.getDistanceFrom(0.0f, 0.0f) };` — checks if the expression compiles.
  - **Compound requirement** (with return type check): `{ shape.getDistanceFrom(0.0f, 0.0f) } -> std::same_as<float>;` — additionally validates that the expression returns the expected type.
- Build concept hierarchies: a more specific concept can include a broader concept as its first requirement.
- The compiler picks the **most specific constrained overload** when multiple overloads exist, enabling concept-based dispatch.

### `constexpr` for Compile-Time Evaluation
- **`constexpr` function**: annotate a function with `constexpr` so the compiler can execute it at compile time when used in a compile-time context. The function can still run at runtime.
  - Eligible if all operations inside (if-statements, arithmetic, comparisons, calls to other `constexpr` functions) are compile-time-evaluable.
- **`constexpr` variable**: annotate a variable to guarantee it is initialized entirely at compile time; any call that cannot be evaluated at compile time becomes a compile error.
- Use case: complex constant initialization (e.g., parsing color hex codes at startup) is moved to compile time, reducing app launch overhead.

### Xcode 14 C++20 `constexpr` Improvements
- Xcode 14 adds `constexpr` to numerous standard library types and algorithms that previously required runtime initialization.
- Setting: Xcode project → Build Settings → **C++ Language Dialect** → set to `C++20`.
- C++20 mode requires no higher minimum deployment target — existing OS targets are unaffected.

### Enabling C++20 in Xcode
- Set **"C++ Language Dialect"** to `C++20` (or `gnu++20`) in Xcode Build Settings.
- C++20 features available: concepts, `constexpr` improvements, ranges, modules (partial), `coroutines` (partial), `format` (partial).

## APIs & Frameworks

**C++20 Standard Library — `<concepts>` header**
- `std::integral<T>` — satisfied by all built-in integer types
- `std::floating_point<T>` — satisfied by `float`, `double`, `long double`
- `std::signed_integral<T>` / `std::unsigned_integral<T>`
- `std::same_as<T, U>` — `T` and `U` are the same type
- `std::convertible_to<From, To>` — `From` is implicitly convertible to `To`
- `std::equality_comparable<T>` — `T` has valid `==` operator
- `std::default_initializable<T>` — `T` has a default constructor
- `std::move_constructible<T>` — `T` can be constructed from rvalue of same type
- `std::copy_constructible<T>` — `T` can be copy-constructed

**C++20 Language Features (Xcode 14)** **[NEW full support]**
- `concept` keyword — declare named concept: `template<typename T> concept MyConcept = requires(T t) { ... };`
- `requires` clause — constrain a template: `template<typename T> requires MyConcept<T>`
- `requires` expression — create inline constraints with argument list and requirement body
  - Simple expression requirement: `{ expr };`
  - Compound requirement with return type: `{ expr } -> Concept;`
- Abbreviated function templates: `void f(MyConcept auto x)`
- `constexpr` function / variable — extended support in Xcode 14 standard library
- `consteval` — functions that must be evaluated at compile time
- `constinit` — variables that must be initialized at compile time but can be modified at runtime

**Xcode 14 Build Settings**
- **C++ Language Dialect** — set to `c++20` or `gnu++20`

## Code Highlights

Constrain a template with a standard library concept (concept instead of `class`):
```cpp
#include <concepts>

template<std::integral T>
bool isOdd(T value) {
    return value % 2 != 0;
}
```

Constrain a template to multiple concepts with `requires` clause:
```cpp
template<typename T>
    requires std::equality_comparable<T> && std::default_initializable<T>
bool isDefaultValue(T value) {
    return value == T{};
}
```

Define a custom concept using `requires` expression with compound requirement:
```cpp
template<typename T>
concept Shape = requires(T shape, float x, float y) {
    { shape.getDistanceFrom(x, y) } -> std::same_as<float>;
};
```

Build a concept hierarchy (more specific concept includes a broader one):
```cpp
template<typename T>
concept GradientShape = Shape<T> && requires(T shape, float x, float y) {
    { shape.getGradientColor(x, y) } -> std::same_as<Color>;
};
```

Compiler picks the most specific constrained overload:
```cpp
// Renders any shape (plain fill)
template<Shape T>
Color computePixelColor(T shape, float x, float y) { ... }

// Renders gradient shapes — picked for GradientCircle (more specific)
template<GradientShape T>
Color computePixelColor(T shape, float x, float y) { ... }
```

Move constant initialization to compile time with `constexpr`:
```cpp
constexpr Color fromHexCode(const char* hex) { /* parse hex string */ }

constexpr Color colorPalette[] = {
    fromHexCode("#FF5733"),
    fromHexCode("#C70039"),
    fromHexCode("#900C3F"),
};
// All three calls are evaluated at compile time — zero runtime parsing cost
```

## Takeaways
- C++20 concepts replace `enable_if` and documentation comments with compiler-enforced type constraints that produce clear call-site error messages, dramatically reducing template debugging time.
- Use `<concepts>` standard library concepts (`std::integral`, `std::equality_comparable`, etc.) for common type requirements; author custom concepts using `requires` expressions with compound requirements to validate method signatures and return types.
- Build concept hierarchies so that the compiler automatically dispatches to the most specific constrained overload — enabling clean, readable generic function variants without runtime branching.
- Mark functions and variables `constexpr` to move complex constant initialization (color palettes, lookup tables, parsed constants) to compile time, reducing app launch overhead with no deployment target cost.
- Switch to C++20 in Xcode Build Settings (`C++ Language Dialect = C++20`) now — it requires no higher minimum deployment target.

---
_Source: WWDC22 Session 110367 page (abstract, transcript, and resource links)._
