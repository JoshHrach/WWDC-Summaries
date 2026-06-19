# Swift and Objective-C Interoperability
**WWDC15 · Session 401** · [Watch](https://developer.apple.com/videos/play/wwdc2015/401/)

_Platforms:_ iOS 9, OS X El Capitan 10.11, watchOS 2

## Overview
This session by Jordan Rose and Doug Gregor covers the substantial improvements made to Swift–Objective-C bridging in Xcode 7 / Swift 2, with three major Objective-C language additions: nullability annotations, lightweight generics, and kindof types. The session also details how Swift's new error-handling model (`throws`) maps bidirectionally to Objective-C's NSError conventions, and covers C function pointer support in Swift 2.

Together, these improvements enable developers to write Objective-C APIs that look natural in Swift (true optionals, typed collections, proper error types), and give Objective-C better static type safety in its own right. Apple audited the majority of its SDK frameworks using these features for the iOS 9 / OS X El Capitan release.

## Key Topics

### Xcode 7: Generated Interface Viewer
- Any Objective-C header now shows its Swift mapping via the "Show Related Items" → "Generated Interface" menu in the source editor. Previously required "Jump to Definition" in Xcode 6.

### Swift Exposure to Objective-C
- **Defaults**: Methods on `NSObject` subclasses are exposed by default; `private` methods and Swift-only return types are not.
- **`@objc` attribute**: Explicitly expose a method/property to ObjC; produces a compiler error if the type cannot be represented in ObjC.
- **`@nonobjc` attribute [NEW]**: Prevent a normally-exposed method from being visible in ObjC — useful when two Swift methods have the same ObjC selector.
- **`IBAction`, `IBOutlet`, `dynamic`**: Dedicated attributes for IB integration and KVO/Cocoa Bindings.
- A class conforming to an ObjC protocol via `@nonobjc` methods produces a Xcode 7 warning.

### C Function Pointers in Swift 2 **[NEW]**
- C function pointers are now a special `@convention(c)` closure type in Swift 2.
- Can be created, passed, and called directly from Swift.
- Cannot capture state from the surrounding scope (no implicit captures allowed) — unlike a regular closure.
- Enables direct access to C callback-based APIs (e.g., `funopen`, `atexit`).

### Swift Error Handling ↔ Objective-C NSError **[NEW]**
- Swift's `throws` / `throw` / `do-catch` maps bidirectionally to Objective-C's `NSError**` convention.
- ObjC method with `NSError**` parameter → Swift as `throws`, dropping the `error:` parameter and adjusting the return type to non-optional (or `Void`).
- ObjC method names ending in `AndReturnError` are trimmed when bridged to Swift (e.g., `checkResourceIsReachableAndReturnError` → `checkResourceIsReachable()`).
- Plain `NSError` parameters (not `NSError**`) bridge normally as `Optional<NSError>`.
- `NSError` conforms to Swift's `Error` protocol (new in Swift 2) — use in `catch` patterns.
- Swift error types (enums conforming to `Error`) bridge to ObjC as `NSError` with the enum type and raw value as domain/code.
- `@objc` on a Swift error enum causes the enum and a domain constant to be emitted in the generated ObjC header **[NEW in Xcode 7]**.
- Common Cocoa error domains (URLError, etc.) are extended to conform to `Error` for direct `catch` usage in Swift.

### Objective-C Nullability Annotations **[NEW in Xcode 6.3, extended in Xcode 7]**
- Qualifiers: `nullable`, `nonnull`, `null_unspecified` (and `__nullable`, `__nonnull`, `__null_unspecified` for C pointers).
- Qualifier position: immediately after the `*` it applies to (same position as `const`/`volatile`).
- Map to Swift: `nullable` → Optional, `nonnull` → non-optional, `null_unspecified` → implicitly unwrapped optional.
- **Audited regions**: `NS_ASSUME_NONNULL_BEGIN` / `NS_ASSUME_NONNULL_END` — within these, single-level pointers default to `nonnull`; `NSError**` parameters default to `nullable` at both levels.
- Apple audited the majority of SDK frameworks for iOS 9 / OS X El Capitan, replacing implicitly unwrapped optionals with true optionals and non-optionals.
- `null_unspecified` is appropriate for pointers with complex or unknown nil semantics.

### Objective-C Lightweight Generics **[NEW]**
- Parameterize Objective-C classes with type arguments using angle brackets: `NSArray<UIView *> *`.
- **Type erasure model**: The compiler uses the type information for static checking only; no runtime changes, no ARC changes, full binary compatibility — deploy to any OS version.
- Generates typed collections in Swift (e.g., `NSArray<UIView *>` → `[UIView]`).
- Produces compiler warnings when assigning incompatible element types (e.g., assigning `NSArray<NSString *>` elements from a `NSURL` return value).
- Covariant for immutable collections; compiler warns on mutable assignments that would violate variance safety.
- Implicit conversions provided for gradual adoption: can add or remove type arguments without source changes to old code.
- Usage in Foundation: `NSArray<ObjectType>`, `NSDictionary<KeyType, ObjectType>`, etc. — used throughout Apple's SDKs.
- Works in categories and extensions: `@interface NSDictionary<KeyType, ObjectType> (MyCategory)`.

### kindof Types **[NEW]**
- Syntax: `__kindof ClassName *` — means "an instance of `ClassName` or any of its subclasses," like `id` but with implicit downcasting to subclasses and without allowing cross-casts.
- Bridges to Swift as the class type itself (e.g., `__kindof NSView *` → `NSView?`).
- Preferred over `id` when the type is known to be a subclass of a specific class.
- Applied to collection element types (e.g., `NSArray<__kindof UIView *> *`) to preserve implicit downcasting behavior without spurious "incompatible types" warnings.
- Example: `NSApp` reclassified as `__kindof NSApplication *` — allows accessing `MyApplication`-specific properties without a cast, while preventing completely unrelated cross-casts.

### Evolving Away from `id`
- `instancetype` — method returns an instance of the receiver's dynamic type (introduced with ARC).
- Typed collections — eliminate `id` from collection APIs.
- `__kindof` — replaces `id` return types where subclass access is expected.
- `id<SomeProtocol>` — still correct when any class conforming to a protocol is acceptable.
- Plain `id` — correct only when truly any type is possible (e.g., `NSDictionary` values with heterogeneous types).

## APIs & Frameworks

- `@objc` attribute (Swift) — explicit ObjC exposure
- `@nonobjc` attribute (Swift) **[NEW]** — prevent ObjC exposure
- `@convention(c)` closure (Swift 2) **[NEW]** — C function pointer type
- `throws` / `throw` / `do-catch` / `try` (Swift 2) **[NEW]** — Swift error handling
- `Error` protocol (Swift 2) **[NEW]** — base protocol for all Swift error types (replacing `ErrorType`)
- `NSError` — conforms to `Error` **[NEW]**
- `NS_ASSUME_NONNULL_BEGIN` / `NS_ASSUME_NONNULL_END` macros **[NEW in Xcode 6.3]**
- `nullable` / `nonnull` / `null_unspecified` Objective-C qualifiers **[NEW in Xcode 6.3]**
- `__nullable` / `__nonnull` / `__null_unspecified` (C pointer variants) **[NEW]**
- ObjC lightweight generics: `@interface Foo<T>` **[NEW]**
- `__kindof` type qualifier **[NEW]**
- `instancetype` return type
- `NSArray<ObjectType>`, `NSMutableArray<ObjectType>` — typed in Foundation SDK **[NEW]**
- `NSDictionary<KeyType, ObjectType>`, `NSMutableDictionary<KeyType, ObjectType>` **[NEW]**
- `NSSet<ObjectType>`, `NSMutableSet<ObjectType>` **[NEW]**
- `NSOrderedSet<ObjectType>`, `NSMutableOrderedSet<ObjectType>` **[NEW]**
- `URLError` / `NSURLError` domain — extended to conform to `Error` **[NEW]**
- Xcode 7 "Generated Interface" view for ObjC headers **[NEW]**

## Code Highlights

Nullability audited region:
```objc
NS_ASSUME_NONNULL_BEGIN
@interface MyView : UIView
@property (nullable, readonly) UIView *superview;
- (nullable UIView *)hitTest:(CGPoint)point withEvent:(nullable UIEvent *)event;
@end
NS_ASSUME_NONNULL_END
```

Typed Objective-C collection:
```objc
@property (readonly, copy) NSArray<UIView *> *subviews;
```

kindof type for downcasting:
```objc
- (__kindof UITableViewCell *)dequeueReusableCellWithIdentifier:(NSString *)identifier;
```

Swift C function pointer (new in Swift 2):
```swift
// C function pointer — no captured state allowed
let callback: @convention(c) (Int32) -> Void = { code in
    // handle signal
}
signal(SIGINT, callback)
```

Swift error mapped to ObjC:
```swift
@objc enum RequestError: Int, Error {
    case invalidURL
}
func sendRequest() throws {
    throw RequestError.invalidURL
}
```

## Takeaways
- Nullability annotations, lightweight generics, and kindof types are the three biggest Objective-C improvements of 2015 — adopting them makes Objective-C APIs look significantly better in Swift and adds compile-time safety to Objective-C itself.
- Swift 2's `throws` / `Error` system maps cleanly to the NSError convention in both directions — Swift errors become `NSError` in ObjC, and ObjC `NSError**` parameters become `throws` in Swift.
- C function pointers are now first-class `@convention(c)` closures in Swift 2, opening up a broad range of C callback APIs.
- `__kindof` and typed collections together eliminate most legitimate uses of `id` in Cocoa APIs.

---
_Source: WWDC15 Session 401 page (abstract, chapter summaries, code samples, and resource links)._
