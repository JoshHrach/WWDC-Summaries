# Optimize the interface of your Mac Catalyst app
**WWDC20 · Session 10056** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10056/)

_Platforms:_ macOS Big Sur 11 (Mac Catalyst), iOS 14, iPadOS 14

## Overview
This session introduces the new "Optimize Interface for Mac" build option in Xcode 12, which runs a Mac Catalyst app in the **Mac idiom** rather than the scaled iPad idiom. Optimizing for Mac delivers native macOS controls, true unscaled text, Mac system spacing, and sharper graphics — at the cost of requiring layout audits wherever the app made implicit assumptions about iPad control sizes.

The session uses a Cookbook app demo to walk through: enabling the Xcode 12 build option, fixing layouts with idiom-aware code and Interface Builder trait variations, providing Mac-specific assets in the Asset Catalog, and hiding navigation bars in favor of Mac-appropriate patterns. It also covers how SwiftUI controls automatically adapt to the Mac idiom with minimal code changes.

## Key Topics

### Mac Idiom vs. Scaled iPad Idiom
- **Scaled to match iPad**: 100 points in code = 77 points on screen (content scaled down 77%)
- **Optimized for Mac**: 100 points in code = 100 points on screen (true 1:1 rendering)
- Enabling optimization is a build setting change (Xcode 12 General > Mac > "Optimize Interface for Mac"); no compile errors result from the change itself
- Can only choose one idiom per app; cannot have both simultaneously
- Good candidates: text-heavy apps (sharper unscaled fonts), graphics-intensive apps using Metal/SceneKit (higher frame rates, lower power), apps with detailed artwork/icons (pixel-perfect rendering), apps with many controls

### UIKit Control Appearance Changes
- `UIButton`, `UISlider`, `UISwitch`, `UIActivityIndicator`, and other system controls render as native macOS controls with Mac metrics and sizes
- Body text style: 17pt on iPad idiom → 13pt on Mac idiom (use dynamic text styles, not hardcoded sizes)
- Hardcoded font sizes are **not** adjusted — audit and replace with dynamic text styles
- Auto Layout system spacing is generally larger on Mac
- Control tint colors (e.g., `systemTeal`) are dropped on Mac — tinted system buttons are not standard on macOS
- Gesture recognizers on `UIButton` do not fire when the button renders with the native Mac appearance; use `UIButton.menu` property or menu bar items instead
- Slider / control customizations (track tint, custom thumb images) are unavailable in Mac idiom — use `custom` button type to preserve custom event tracking

### Layout Audit After Optimizing
- Check for "unsatisfiable constraint" warnings after enabling Mac idiom
- Provide Mac-specific assets in Asset Catalog (check the "Mac" device checkbox in the asset inspector); fallback order: Mac → Mac Scaled → iPad → Universal
- Use `traitCollection.userInterfaceIdiom == .mac` for idiom-specific code paths
- Use `#if targetEnvironment(macCatalyst)` for Mac Catalyst build-time checks (true regardless of idiom)
- Interface Builder: new idiom chooser in device bar; add trait variations (Installed, size constraints) conditioned on Mac idiom using the "+" button in the Attributes inspector

### Navigation and Window Conventions
- Navigation bars feel out of place on Mac; hide them with `navigationController?.setNavigationBarHidden(true, animated: false)` when `userInterfaceIdiom == .mac`
- Place Save/Cancel buttons at the bottom of views or in the toolbar rather than in navigation bars
- Use `UIButton.menu` to replace long-press gesture recognizers with macOS-style pull-down menus

### SwiftUI in Optimized Catalyst Apps
- SwiftUI controls automatically adopt Mac idiom appearance; less manual layout work required than UIKit
- `GroupBox` **[NEW on iOS/iPadOS]**: renders with Mac-native GroupBox style (padding, label outside box) in Mac idiom
- `Toggle`: default renders as sliding switch on iPad → Mac checkbox in Mac idiom; use `ToggleStyle` to override
- `Button`: system buttons adopt native Mac appearance; custom `ButtonStyle` implementations pass through unchanged
- `DatePicker`: compact (default) style renders identically across idioms
- `Picker`: default style becomes a native Mac popup button in Mac idiom (vs. scrolling picker on iPad)
- `UIHostingController` allows dropping SwiftUI views into UIKit Mac Catalyst apps with no additional code for idiom adaptation

## APIs & Frameworks

- **Mac Catalyst** / **UIKit**
- `UIUserInterfaceIdiom.mac` **[NEW]** — Mac idiom constant
- `UITraitCollection.userInterfaceIdiom` — check idiom at runtime
- `#if targetEnvironment(macCatalyst)` — compile-time Mac Catalyst check
- `UIButton.menu` (`UIMenu`) **[NEW]** — attach pull-down menu to button; replaces long-press gesture on Mac
- `UIContextMenuInteraction` — right-click menus
- `UINavigationController.setNavigationBarHidden(_:animated:)` — hide navigation bar in Mac idiom
- `NSLayoutConstraint.priority` — use `.defaultHigh` to allow constraint breaking when idiom changes layout
- Asset Catalog Mac device slot — Mac, Mac Scaled, iPad, Universal fallback chain
- Interface Builder trait variations — "Installed" attribute conditioned on Mac idiom **[NEW in Xcode 12]**
- Interface Builder device bar idiom chooser **[NEW in Xcode 12]**
- **SwiftUI**
- `GroupBox` **[NEW on iOS 14/iPadOS 14]** — layered structured content with semantic background colors
- `Toggle` / `ToggleStyle` — `DefaultToggleStyle` → checkbox on Mac idiom
- `Picker` / `PickerStyle` — `DefaultPickerStyle` → popup button on Mac idiom, `SegmentedPickerStyle`
- `DatePicker` / `DatePickerStyle` — `DefaultDatePickerStyle`, `CompactDatePickerStyle`
- `ButtonStyle` — custom styles pass through unchanged in Mac idiom
- `UIHostingController` — embed SwiftUI in UIKit Mac Catalyst apps

## Code Highlights

Check idiom to hide navigation bar:
```swift
if traitCollection.userInterfaceIdiom == .mac {
    navigationController?.setNavigationBarHidden(true, animated: false)
}
```

Idiom check vs. conditional compilation — use both together when needed:
```swift
if traitCollection.userInterfaceIdiom == .mac {
    // Mac idiom-specific code
} else if traitCollection.userInterfaceIdiom == .pad {
    #if targetEnvironment(macCatalyst)
        // Mac Catalyst scaled-iPad code
    #else
        // iPad-native code
    #endif
}
```

SwiftUI Toggle (renders as checkbox on Mac idiom automatically):
```swift
Toggle("Complete?", isOn: $completed)
```

## Takeaways

- Enabling "Optimize Interface for Mac" delivers native Mac controls, sharper text, and better graphics performance, but requires a layout audit — control metrics, system spacing, and text sizes all change.
- Always use dynamic text styles (`.body`, `.title`, etc.) instead of hardcoded font sizes; Mac idiom adjusts text styles automatically.
- Replace gesture recognizers on `UIButton` with `UIButton.menu` or menu bar actions, and provide Mac-specific assets in the Asset Catalog.
- SwiftUI adapts to the Mac idiom largely automatically (`Toggle` → checkbox, `Picker` → popup button, `GroupBox` → Mac style), making it the lowest-friction path for Mac Catalyst UI work.

---
_Source: WWDC20 Session 10056 page (abstract, chapter summaries, code samples, and resource links)._
