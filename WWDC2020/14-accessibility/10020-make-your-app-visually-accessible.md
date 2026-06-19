# Make your app visually accessible
**WWDC20 · Session 10020** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10020/)

_Platforms:_ iOS 14, iPadOS 14

## Overview
Visual accessibility encompasses far more than VoiceOver. Vision loss is a broad continuum — including color blindness, low vision, light sensitivity, and motion sensitivity — so iOS provides a rich set of display accommodation settings that users can enable. This session walks through the developer APIs required to respond to these settings, organized into three areas: color and shapes, text readability, and display preferences.

The central message is to design adaptively: check accessibility settings and observe their change notifications so your app's appearance updates dynamically. iOS 14 introduces two new APIs: `UIAccessibility.buttonShapesEnabled` for detecting the Button Shapes setting, and `UIAccessibility.prefersCrossFadeTransitions` for detecting the Prefer Cross-Fade Transitions setting.

## Key Topics

### Color and Shapes
- Never rely on color alone to convey meaning; always pair with a shape, symbol, or other visual differentiator
- Use SF Symbols (1,500+ configurable symbols) to add visual meaning alongside color; they scale with text size
- **Button Shapes** (iOS 14 new API): `UIAccessibility.buttonShapesEnabled` and `UIAccessibility.buttonShapesEnabledStatusDidChangeNotification` — provide an alternate button appearance with visible shapes when enabled
- **Differentiate Without Color** (iOS 13+): `UIAccessibility.shouldDifferentiateWithoutColor` and the corresponding notification — use symbols or textures alongside color for status icons and other color-coded UI
- **Increase Contrast**: System colors automatically adapt; for custom colors and assets, provide high-contrast variants in the asset catalog (check "High Contrast" under Appearances in the Attributes Inspector)
- Color contrast minimum ratio: 4.5:1 for most cases; use the Xcode Accessibility Inspector's Color Contrast Calculator
- **Smart Invert Colors**: Use `UIView.accessibilityIgnoresInvertColors = true` to exempt photos, videos, and app icons from inversion

### Text Readability
- Support Dynamic Type: use system font styles so text scales with the user's preferred content size
- Wrap labels (`numberOfLines = 0`) so text is never truncated as content size increases
- **Large Text**: Override `traitCollectionDidChange(_:)` and use `UITraitCollection.preferredContentSizeCategory` with comparison operators to detect accessibility font sizes; adapt layout axes and alignment accordingly
- SF Symbols scale with text automatically — no special handling needed
- **Bold Text**: `UIAccessibility.isBoldTextEnabled` and `UIAccessibility.boldTextStatusDidChangeNotification` — for custom labels not using system font styles, manually apply bold/heavy font weight

### Display Preferences
- **Reduce Motion**: `UIAccessibility.isReduceMotionEnabled` and `UIAccessibility.reduceMotionStatusDidChangeNotification` — suppress parallax effects (`UIMotionEffect`), idle animations, auto-playing media, and slide transitions
- **Prefer Cross-Fade Transitions** (iOS 14 new API): `UIAccessibility.prefersCrossFadeTransitions` and `UIAccessibility.prefersCrossFadeTransitionsStatusDidChange` — replace custom slide transitions with cross-fade animations; `UINavigationController` handles this automatically
- **Reduce Transparency**: `UIAccessibility.isReduceTransparencyEnabled` and `UIAccessibility.reduceTransparencyStatusDidChangeNotification` — system blur/vibrancy effects automatically become opaque; replace custom blur effects with solid colors

## APIs & Frameworks

- **UIKit / UIAccessibility**
  - `UIAccessibility.buttonShapesEnabled` **[NEW in iOS 14]** — `Bool`; true when Button Shapes is on
  - `UIAccessibility.buttonShapesEnabledStatusDidChangeNotification` **[NEW in iOS 14]** — posted when setting changes
  - `UIAccessibility.shouldDifferentiateWithoutColor` — `Bool`; true when Differentiate Without Color is on
  - `UIAccessibility.differentiateWithoutColorDidChangeNotification` — posted when setting changes
  - `UIAccessibility.isBoldTextEnabled` — `Bool`; true when Bold Text is on
  - `UIAccessibility.boldTextStatusDidChangeNotification` — posted when setting changes
  - `UIAccessibility.isReduceMotionEnabled` — `Bool`; true when Reduce Motion is on
  - `UIAccessibility.reduceMotionStatusDidChangeNotification` — posted when setting changes
  - `UIAccessibility.prefersCrossFadeTransitions` **[NEW in iOS 14]** — `Bool`; true when Prefer Cross-Fade Transitions is on
  - `UIAccessibility.prefersCrossFadeTransitionsStatusDidChange` **[NEW in iOS 14]** — notification posted when setting changes
  - `UIAccessibility.isReduceTransparencyEnabled` — `Bool`; true when Reduce Transparency is on
  - `UIAccessibility.reduceTransparencyStatusDidChangeNotification` — posted when setting changes
  - `UIView.accessibilityIgnoresInvertColors` — `Bool`; set `true` to exempt a view from Smart Invert Colors
  - `UIMotionEffect` / `UIInterpolatingMotionEffect` — parallax effects; suppress when Reduce Motion is on
- **UITraitCollection**
  - `UITraitCollection.preferredContentSizeCategory` — current Dynamic Type size category
  - `UIContentSizeCategory` — comparable enum (supports `<`, `>` operators) for size threshold checks
- **UIViewController**
  - `traitCollectionDidChange(_:)` — called when display traits change (content size, color scheme, etc.)
- **Asset Catalog**
  - High Contrast appearance slot — provide alternate assets for Increase Contrast
  - Symbol configuration for High Contrast in Attributes Inspector
- **Xcode Accessibility Inspector**
  - Color Contrast Calculator tool for computing contrast ratios between foreground/background colors

## Code Highlights

Adapting layout to large accessibility text sizes:
```swift
override func traitCollectionDidChange(_ previousTraitCollection: UITraitCollection?) {
    if traitCollection.preferredContentSizeCategory < .accessibilityMedium {
        stackView.axis = .horizontal
        stackView.alignment = .center
    } else {
        stackView.axis = .vertical
        stackView.alignment = .leading
    }
}
```

Observing Button Shapes (new in iOS 14):
```swift
NotificationCenter.default.addObserver(self, selector: #selector(updateButtonShapes),
    name: UIAccessibility.buttonShapesEnabledStatusDidChangeNotification, object: nil)

@objc func updateButtonShapes() {
    if UIAccessibility.buttonShapesEnabled {
        // Apply shape-based button visualization
    }
}
```

Responding to Prefer Cross-Fade Transitions (new in iOS 14):
```swift
NotificationCenter.default.addObserver(self, selector: #selector(updateTransitions),
    name: UIAccessibility.prefersCrossFadeTransitionsStatusDidChange, object: nil)

@objc func updateTransitions() {
    if UIAccessibility.prefersCrossFadeTransitions {
        // Replace slide transitions with cross-fade
    }
}
```

Exempting a photo from Smart Invert Colors:
```swift
photoImageView.accessibilityIgnoresInvertColors = true
```

## Takeaways
- Pair color with shapes or symbols to convey meaning — never rely on color alone; observe `shouldDifferentiateWithoutColor` and `buttonShapesEnabled` (new in iOS 14) for guidance.
- Use system font styles and `traitCollectionDidChange` to adapt layout for Dynamic Type, especially at accessibility font sizes where switching from horizontal to vertical stacks is often necessary.
- Always observe accessibility setting change notifications in addition to checking the current value — users can change settings while your app is open and expect immediate response.
- `UIAccessibility.prefersCrossFadeTransitions` is new in iOS 14; `UINavigationController` handles it automatically, but custom slide transitions require explicit adoption.

---
_Source: WWDC20 Session 10020 page (abstract, transcript, code samples, and resource links)._
