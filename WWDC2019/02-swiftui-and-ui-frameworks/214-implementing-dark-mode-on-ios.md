# Implementing Dark Mode on iOS
**WWDC19 · Session 214** · [Watch](https://developer.apple.com/videos/play/wwdc2019/214/)

_Platforms:_ iOS 13, iPadOS 13, tvOS 13, macOS 10.15 Catalina (Mac Catalyst)

## Overview
iOS 13 introduces system-wide Dark Mode — a completely re-designed dark appearance built on top of a flexible design system anchored in three concepts: semantic dynamic colors, adaptive materials (blur + vibrancy), and adaptive built-in controls. As soon as an app is built against the iOS 13 SDK it participates in Dark Mode, but every app requires manual review and likely some code changes to look correct in both appearances.

The session walks through live Xcode demos updating a sample app first via Interface Builder (storyboard color overrides, asset catalog dark-appearance slots for colors and images) and then in code (using system background colors, dynamic blur/vibrancy effects). Trait collections are the underlying mechanism that carries the current appearance through the view hierarchy, and the session explains the updated iOS 13 trait-propagation model that pre-populates traits at initialization time.

The session also covers API changes for the status bar and activity indicators, best practices for attributed strings and web content, and how to use the new `overrideUserInterfaceStyle` property to force a specific appearance on a subset of your UI.

## Key Topics

**Semantic Dynamic Colors**
- System provides a full palette of named dynamic colors: `systemBackground`, `secondarySystemBackground`, `tertiarySystemBackground`, `label`, `secondaryLabel`, `tertiaryLabel`, `quaternaryLabel`, `systemBlue`, etc.
- Each color resolves to a different RGB value depending on the active appearance (light/dark) and elevated vs. base UI level
- Elevated level (presented view controllers, iPad multitasking split view) gets lighter background values in Dark Mode to distinguish from the base black background
- Use semantic names when building UI; avoid hard-coded RGB values

**Custom Dynamic Colors**
- Create in asset catalog: open a color's dark appearance slot and set the dark-mode value
- Create in code: `UIColor(dynamicProvider:)` — takes a closure receiving a `UITraitCollection`, returns a resolved `UIColor`
- `UIColor.resolvedColor(with:)` — explicitly resolve a dynamic color for a specific trait collection

**Dynamic Images**
- Asset catalog supports a dark appearance slot for image assets alongside colors
- `UIImageView` automatically picks the correct image variant using its trait collection
- `UIImage.imageAsset` — access the underlying `UIImageAsset`; call `image(with:)` to manually resolve; or `register(_:with:)` to add runtime variants

**Trait Collections (iOS 13 Changes)**
- `UITraitCollection.userInterfaceStyle` (`.light` / `.dark`) — primary appearance trait
- `UITraitCollection.userInterfaceLevel` (`.base` / `.elevated`) — layering trait
- `UITraitCollection.current` — new class property; UIKit sets it before calling `draw(_:)`, `layoutSubviews`, `traitCollectionDidChange(_:)`, etc.
- `UITraitCollection.performAsCurrent(_:)` — temporarily sets a trait collection as `.current` for the duration of a closure
- Traits are now **predicted at initialization time** in iOS 13; `traitCollectionDidChange` is only called afterward if the initial prediction was wrong
- Use `UITraitCollection.hasDifferentColorAppearance(comparedTo:)` to check whether re-resolving colors is necessary
- `UITraitCollection.current` is thread-safe: setting it on a background thread only affects that thread

**Materials (Blur and Vibrancy)**
- New `UIBlurEffect.Style` cases: `.systemMaterial`, `.systemThickMaterial`, `.systemThinMaterial`, `.systemUltraThinMaterial` — each works correctly in light and dark
- New `UIVibrancyEffect.Style` cases for text (`.label`, `.secondaryLabel`, `.tertiaryLabel`, `.quaternaryLabel`), fill areas (`.fill`, `.secondaryFill`, `.tertiaryFill`), and separators (`.separator`)
- Add views with vibrancy through `UIVisualEffectView.contentView`, not directly

**Forcing Appearance**
- `UIViewController.overrideUserInterfaceStyle` — force `.light`, `.dark`, or `.unspecified` for a view controller and all its children
- `UIView.overrideUserInterfaceStyle` — same for a view subtree (use view controller property when possible)
- `UIUserInterfaceStyle` Info.plist key — force the entire app to a fixed appearance

**Other API Changes**
- Status bar: new `UIStatusBarStyle.darkContent` **[NEW]**; `.default` now auto-switches based on the view controller's user interface style
- `UIActivityIndicatorView.Style`: `.medium` **[NEW]** and `.large` **[NEW]** replace deprecated `.gray`, `.white`, `.whiteLarge`
- Attributed text: always set `NSAttributedString.Key.foregroundColor` when using attributed strings — unadorned attributed text defaults to black regardless of appearance
- Web content: opt in to Dark Mode via `color-scheme` CSS property/meta tag; use `prefers-color-scheme` media query

## APIs & Frameworks

**UIKit**
- `UIColor(dynamicProvider:)` **[NEW]** — creates a dynamic color resolved via trait collection closure
- `UIColor.resolvedColor(with:)` **[NEW]** — resolve a dynamic color for a given `UITraitCollection`
- `UIColor.systemBackground`, `.secondarySystemBackground`, `.tertiarySystemBackground` **[NEW]**
- `UIColor.label`, `.secondaryLabel`, `.tertiaryLabel`, `.quaternaryLabel` **[NEW]**
- `UIColor.systemFill`, `.secondarySystemFill`, `.tertiarySystemFill`, `.quaternarySystemFill` **[NEW]**
- `UIColor.systemBlue`, `.systemGreen`, `.systemRed`, `.systemOrange`, etc. (adaptive variants) **[NEW]**
- `UITraitCollection.userInterfaceStyle: UIUserInterfaceStyle` **[NEW enum cases: `.dark`]**
- `UITraitCollection.userInterfaceLevel: UIUserInterfaceLevel` **[NEW]** (`.base` / `.elevated`)
- `UITraitCollection.current: UITraitCollection` **[NEW]** — class property
- `UITraitCollection.performAsCurrent(_:)` **[NEW]**
- `UITraitCollection.hasDifferentColorAppearance(comparedTo:)` **[NEW]**
- `UIViewController.overrideUserInterfaceStyle` **[NEW]**
- `UIView.overrideUserInterfaceStyle` **[NEW]**
- `UIBlurEffect.Style.systemMaterial`, `.systemThickMaterial`, `.systemThinMaterial`, `.systemUltraThinMaterial` **[NEW]**
- `UIVibrancyEffect(blurEffect:style:)` **[NEW]** — takes a style parameter
- `UIVibrancyEffect.Style` **[NEW]**: `.label`, `.secondaryLabel`, `.tertiaryLabel`, `.quaternaryLabel`, `.fill`, `.secondaryFill`, `.tertiaryFill`, `.separator`
- `UIStatusBarStyle.darkContent` **[NEW]**; `.default` becomes automatic switcher
- `UIScrollView.indicatorStyle` — now has automatic/dynamic behavior
- `UIActivityIndicatorView.Style.medium` **[NEW]**, `.large` **[NEW]**; `.gray`, `.white`, `.whiteLarge` deprecated
- `UIImageView` — resolves dark/light image asset automatically from its trait collection
- `UIImageAsset` — `image(with:)` to manually resolve; `register(_:with:)` to add runtime variants

## Code Highlights

Creating a custom dynamic color in code:
```swift
let dynamicColor = UIColor { (traitCollection: UITraitCollection) -> UIColor in
    if traitCollection.userInterfaceStyle == .dark {
        return UIColor(red: 1, green: 0.8, blue: 0, alpha: 1) // yellow in dark
    } else {
        return UIColor(red: 0, green: 0.6, blue: 0.2, alpha: 1) // green in light
    }
}
```

Resolving a dynamic color for a CA layer (which requires a plain `CGColor`):
```swift
// Option 1 — resolveColor
let cgColor = dynamicColor.resolvedColor(with: view.traitCollection).cgColor
layer.borderColor = cgColor

// Option 2 — performAsCurrent
view.traitCollection.performAsCurrent {
    layer.borderColor = dynamicColor.cgColor
}

// Option 3 — set current directly (safe on background threads)
let saved = UITraitCollection.current
UITraitCollection.current = view.traitCollection
layer.borderColor = dynamicColor.cgColor
UITraitCollection.current = saved
```

Updating layer-based colors when traits change:
```swift
override func traitCollectionDidChange(_ previousTraitCollection: UITraitCollection?) {
    super.traitCollectionDidChange(previousTraitCollection)
    if traitCollection.hasDifferentColorAppearance(comparedTo: previousTraitCollection) {
        layer.borderColor = dynamicColor.resolvedColor(with: traitCollection).cgColor
    }
}
```

## Takeaways
- Replace all hard-coded `UIColor` RGB values with semantic system colors from the new palette; this alone resolves the majority of Dark Mode issues with no additional code.
- Create asset catalog dark-appearance slots for custom brand colors and images — this enables zero-code Dark Mode support for those assets.
- Vibrancy and blur materials must use the new `system*` styles to adapt automatically; the old `.light`/`.dark` styles are static and will not update when the appearance changes.
- Use `overrideUserInterfaceStyle` on a view controller (not a view) to lock a portion of your UI to a fixed appearance; the Info.plist key locks the entire app.

---
_Source: WWDC19 Session 214 page (abstract, chapter summaries, code samples, and resource links)._
