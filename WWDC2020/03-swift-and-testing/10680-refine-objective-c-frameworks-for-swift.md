# Refine Objective-C frameworks for Swift
**WWDC20 · Session 10680** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10680/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, watchOS 7, tvOS 14

## Overview
This session demonstrates how to annotate Objective-C framework headers so that the framework's API surface is natural, expressive, and Swifty when imported into Swift. Starting from a sample framework with typical Objective-C idioms, the session walks through a series of annotations and header restructuring techniques that transform the Swift-visible API without changing the Objective-C interface or ABI.

The session covers nullability annotations, typed enumerations, option sets, error domains, string constants, and the use of `NS_SWIFT_NAME`, `NS_REFINED_FOR_SWIFT`, and related macros to produce a high-quality framework API that feels native to Swift callers.

## Key Topics

### Nullability Annotations
- Unannotated Objective-C pointers import as implicitly unwrapped optionals (`Type!`) in Swift — a poor experience
- Wrap headers in `NS_ASSUME_NONNULL_BEGIN` / `NS_ASSUME_NONNULL_END` to default all pointers to non-optional
- Mark genuinely-nullable pointers explicitly with `nullable` qualifier
- Use `_Nullable` and `_Nonnull` for C pointers and function pointer parameters outside of methods

### Typed Enumerations and Option Sets
- Raw `#define` integer constants import as `Int` in Swift — lose type safety
- Replace with `NS_ENUM` for mutually-exclusive values → imports as Swift `enum`
- Replace with `NS_OPTIONS` for bitmask values → imports as Swift `OptionSet` (with `[]` empty value, `|` combinations)
- Use `NS_CLOSED_ENUM` for enums where no future cases will be added — allows exhaustive `switch` in Swift without `@unknown default`
- Use `NS_TYPED_ENUM` / `NS_TYPED_EXTENSIBLE_ENUM` for string/integer constant groups that aren't true enums

### Error Domains
- Use `NS_ERROR_ENUM(domain, name)` macro to associate an enum with an error domain
- This produces a Swift error type that conforms to `Error` with the correct domain, enabling pattern matching in `catch` blocks
- Declare the error domain string constant using `NS_SWIFT_NAME` if needed

### String Constants and Typed Wrappers
- Raw `NSString *const` or `typedef NSString *` constants import poorly
- Use `NS_TYPED_ENUM` or `NS_TYPED_EXTENSIBLE_ENUM` on a `typedef NSString *` to create a Swift struct-backed typed constant group
- `NS_TYPED_EXTENSIBLE_ENUM` allows Swift callers to add their own values via `extension`

### NS_SWIFT_NAME
- Rename any Objective-C type, method, property, or constant for Swift using `NS_SWIFT_NAME(SwiftName)`
- Particularly useful for: removing redundant type prefixes from enum cases, renaming factory methods to Swift initializers (`NS_SWIFT_NAME(init(...))`), renaming class methods to static properties or computed properties
- Can rename free functions, typedefs, and global constants

### NS_REFINED_FOR_SWIFT
- Mark a method with `NS_REFINED_FOR_SWIFT` to make it available in Swift as `__methodName` (double-underscore prefix)
- Allows writing a pure-Swift wrapper in an extension that delegates to the underlying Objective-C implementation for precise control over the Swift API shape (e.g., returning a Swift tuple instead of multiple out-parameters, using `throws` instead of error pointer parameters)

### Lightweight Generics
- Use `__covariant` parameterized classes (`NSArray<ElementType *> *`) to give Swift callers typed collections instead of `[Any]`
- Apply to `NSArray`, `NSDictionary`, `NSSet`, `NSOrderedSet`, and custom container classes

### Additional Annotations
- `NS_SWIFT_UNAVAILABLE("message")` — hide an Objective-C API from Swift entirely, with a diagnostic message
- `API_AVAILABLE(ios(14.0))` / `API_UNAVAILABLE(watchos)` — platform availability annotations carried through to Swift
- `__attribute__((swift_wrapper))` — used internally by `NS_TYPED_ENUM`

## APIs & Frameworks

- `NS_ASSUME_NONNULL_BEGIN` / `NS_ASSUME_NONNULL_END` — nullability region macros
- `nullable` / `nonnull` / `_Nullable` / `_Nonnull` — nullability qualifiers
- `NS_ENUM(type, name)` — typed enum macro → Swift `enum`
- `NS_OPTIONS(type, name)` — bitmask option set macro → Swift `OptionSet`
- `NS_CLOSED_ENUM(type, name)` — frozen enum (no future cases) → exhaustive Swift `switch`
- `NS_TYPED_ENUM` — typed constant group → Swift struct-backed constant namespace
- `NS_TYPED_EXTENSIBLE_ENUM` — extensible typed constant group
- `NS_ERROR_ENUM(domain, name)` — error domain + enum binding → Swift `Error`-conforming type
- `NS_SWIFT_NAME(name)` — rename symbol for Swift
- `NS_REFINED_FOR_SWIFT` — hide as `__name` in Swift for replacement by a Swift extension
- `NS_SWIFT_UNAVAILABLE("message")` — exclude API from Swift
- `__covariant` — lightweight generic type parameter for Objective-C collections
- `NS_NOESCAPE` — mark block parameters as non-escaping → Swift `@noescape` closure
- `NS_SWIFT_NOTHROW` — suppress automatic `throws` bridging for methods returning `BOOL` with `NSError **`

## Code Highlights

Before — poor Swift import:
```objc
typedef NSString * TLSVersion;
extern TLSVersion TLSVersion10;
extern TLSVersion TLSVersion12;
```
After — typed extensible enum in Swift:
```objc
typedef NSString * TLSVersion NS_TYPED_EXTENSIBLE_ENUM;
extern TLSVersion const TLSVersion10;
extern TLSVersion const TLSVersion12;
```
Swift sees: `TLSVersion.v10`, `TLSVersion.v12`, and callers can extend with their own values.

Before — multiple out-parameters (awkward in Swift):
```objc
- (void)fetchUserWithID:(NSString *)id
                   name:(NSString **)name
                  email:(NSString **)email NS_REFINED_FOR_SWIFT;
```
After — Swift wrapper returns a tuple:
```swift
extension MyClient {
    func fetchUser(id: String) -> (name: String, email: String) {
        var name: NSString?, email: NSString?
        __fetchUser(withID: id, name: &name, email: &email)
        return (name! as String, email! as String)
    }
}
```

## Takeaways

- Apply `NS_ASSUME_NONNULL_BEGIN/END` first — it eliminates implicitly-unwrapped optionals throughout the header with a single change.
- Replace `#define` integer constants and `typedef NSString *` groupings with `NS_ENUM`, `NS_OPTIONS`, or `NS_TYPED_ENUM` to restore type safety in Swift.
- Use `NS_SWIFT_NAME` to eliminate redundant prefixes, rename factory methods to initializers, and align naming with Swift API design guidelines.
- Use `NS_REFINED_FOR_SWIFT` when the Objective-C signature can't be made Swifty on its own — write a Swift extension to provide a clean API that delegates to the annotated method.

---
_Source: WWDC20 Session 10680 page (abstract, chapter summaries, code samples, and resource links)._
