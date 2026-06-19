# Unleash the UIKit Trait System
**WWDC23 · Session 10057** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10057/)

_Platforms:_ iOS 17, iPadOS 17

## Overview
The UIKit trait system received major enhancements in iOS 17, enabling developers to define custom traits, apply trait overrides with a new API, and handle trait changes more efficiently. The session covers the fundamentals of how traits and trait collections work, then introduces the new custom trait mechanism that is conceptually similar to SwiftUI environment keys.

A unified trait hierarchy was introduced in iOS 17, where view controllers now inherit their trait collection from their view's superview instead of directly from a parent view controller. This creates a simple linear flow and eliminates surprising edge cases. A new `viewIsAppearing` lifecycle method back-deploys to iOS 13 and replaces many uses of `viewWillAppear` for code that needs up-to-date trait values.

The session concludes with a powerful bridging mechanism that lets a UIKit custom trait and a SwiftUI custom environment key share the same underlying storage, enabling seamless data propagation across mixed UIKit/SwiftUI hierarchies.

## Key Topics

### Understanding Traits
- Traits are independent pieces of data propagated automatically through the trait hierarchy (window scenes → windows → presentation controllers → view controllers → views).
- New `UITraitCollection` initializer takes a closure providing a `UIMutableTraits` container; `modifyingTraits` creates a new instance by modifying an existing collection.
- Unified hierarchy: view controllers now inherit traits from their view's superview; views update traits immediately before `layoutSubviews`.

### Defining Custom Traits
- Conform a struct to `UITraitDefinition`; provide `defaultValue` (associated type is inferred).
- Optional static properties: `affectsColorAppearance` (triggers automatic redraw), `name` (debug label), `identifier` (reverse-DNS, enables encoding).
- Best value types: `Bool`, `Int`, `Double`, or `enum` with `Int` raw value. Custom structs need efficient `Equatable`.
- Add extensions on `UITraitCollection` (read-only) and `UIMutableTraits` (read-write) for property-syntax access.

### Applying Overrides
- New `traitOverrides` property on `UIWindowScene`, `UIWindow`, `UIViewController`, `UIView`, and `UIPresentationController`.
- Setting an override propagates the new value to all descendants; changes may not reflect immediately (views update before layout).
- `contains(_:)` checks for existing override; `remove(_:)` removes it entirely.
- Performance: minimize override count and change frequency; apply overrides at the nearest common ancestor.

### Handling Changes
- `traitCollectionDidChange` is deprecated in iOS 17.
- New `registerForTraitChanges(_:handler:)` (closure) and `registerForTraitChanges(_:target:action:)` pattern.
- Register only for specific traits; callback fires only when those traits change.
- Can register on any trait environment object, not just `self`.
- Semantic sets: `UITraitCollection.systemTraitsAffectingColorAppearance`, `systemTraitsAffectingImageLookup`.
- Returns a registration token for manual `unregisterForTraitChanges(_:)` (rarely needed).

### SwiftUI Bridging
- Conform a SwiftUI `EnvironmentKey` to `UITraitBridgedEnvironmentKey`; implement `read(from:)` and `write(to:value:)`.
- UIKit trait overrides propagate into SwiftUI environment; SwiftUI `.environment` modifier writes back to UIKit traits.

## APIs & Frameworks

- `UITraitDefinition` **[NEW]** — protocol for defining custom traits
- `UIMutableTraits` **[NEW]** — protocol providing mutable access to trait values inside closures and overrides
- `UITraitCollection.init(_:)` (closure-based) **[NEW]** — build a trait collection from scratch
- `UITraitCollection.modifyingTraits(_:)` **[NEW]** — derive a new collection by modifying an existing one
- `UITraitCollection.systemTraitsAffectingColorAppearance` **[NEW]** — semantic set of color-affecting system traits
- `UITraitCollection.systemTraitsAffectingImageLookup` **[NEW]** — semantic set of image-lookup system traits
- `UITraitEnvironment.traitOverrides` **[NEW]** — property on `UIWindowScene`, `UIWindow`, `UIViewController`, `UIView`, `UIPresentationController`
- `UITraitOverrides.contains(_:)` **[NEW]** — check if an override is set
- `UITraitOverrides.remove(_:)` **[NEW]** — remove a specific override
- `registerForTraitChanges(_:handler:)` **[NEW]** — closure-based trait change registration
- `registerForTraitChanges(_:target:action:)` **[NEW]** — target-action trait change registration
- `unregisterForTraitChanges(_:)` **[NEW]** — manual unregistration via token
- `UITraitBridgedEnvironmentKey` **[NEW]** — protocol to bridge UIKit traits with SwiftUI environment keys
- `UITraitHorizontalSizeClass`, `UITraitVerticalSizeClass`, `UITraitUserInterfaceStyle`, etc. **[NEW]** — `UITrait` symbols for system traits
- `UITraitDefinition.affectsColorAppearance` **[NEW]** — static flag for color-affecting traits
- `UITraitDefinition.identifier` **[NEW]** — reverse-DNS identifier enabling encoding
- `UIViewController.viewIsAppearing(_:)` **[NEW]** — new lifecycle method (back-deploys to iOS 13)
- `UITraitCollection` (existing) — subscript access via custom trait types
- `UIMutableTraits` subscript — get/set custom trait values
- `UIColor(dynamicProvider:)` — dynamic colors that resolve using trait collection
- `UIHostingConfiguration` — SwiftUI views in UIKit collection/table view cells
- `UIViewControllerRepresentable` — embed UIKit view controllers in SwiftUI
- `EnvironmentKey` — SwiftUI protocol for custom environment values
- `EnvironmentValues` — SwiftUI environment storage

## Code Highlights

```swift
// Define a custom trait
struct MyAppThemeTrait: UITraitDefinition {
    static let defaultValue = MyAppTheme.standard
    static let affectsColorAppearance = true
    static let identifier = "com.myapp.theme"
}

// Extend UITraitCollection and UIMutableTraits for property syntax
extension UITraitCollection {
    var myAppTheme: MyAppTheme { self[MyAppThemeTrait.self] }
}
extension UIMutableTraits {
    var myAppTheme: MyAppTheme {
        get { self[MyAppThemeTrait.self] }
        set { self[MyAppThemeTrait.self] = newValue }
    }
}

// Apply an override
windowScene.traitOverrides.myAppTheme = .monochrome

// Register for trait changes (closure)
registerForTraitChanges([UITraitHorizontalSizeClass.self]) { (self: Self, previous: UITraitCollection) in
    self.updateViews(sizeClass: self.traitCollection.horizontalSizeClass)
}

// Bridge UIKit trait to SwiftUI environment key
extension MyAppThemeKey: UITraitBridgedEnvironmentKey {
    static func read(from traitCollection: UITraitCollection) -> MyAppTheme { traitCollection.myAppTheme }
    static func write(to mutableTraits: inout UIMutableTraits, value: MyAppTheme) { mutableTraits.myAppTheme = value }
}
```

## Takeaways
- Custom traits (conforming to `UITraitDefinition`) provide a first-class mechanism for propagating app-specific data through the UIKit hierarchy, replacing ad-hoc patterns.
- The unified trait hierarchy (iOS 17) simplifies reasoning about trait propagation; `viewIsAppearing` is the new drop-in replacement for `viewWillAppear` in most cases.
- `registerForTraitChanges` replaces `traitCollectionDidChange`, giving precise, performant callbacks only for the traits you care about.
- `UITraitBridgedEnvironmentKey` enables seamless, zero-duplication data sharing between UIKit traits and SwiftUI environment keys.

---
_Source: WWDC23 Session 10057 page (abstract, chapter summaries, code samples, and resource links)._
