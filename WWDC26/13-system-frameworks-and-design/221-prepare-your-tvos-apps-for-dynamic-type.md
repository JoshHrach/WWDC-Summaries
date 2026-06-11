# Prepare your tvOS apps for Dynamic Type
**WWDC26 · Session 221** · [Watch](https://developer.apple.com/videos/play/wwdc2026/221/)

_Platforms:_ tvOS 27

## Overview
tvOS 27 introduces system-wide Large Text support, bringing Dynamic Type to Apple TV for the first time. Apps that support Dynamic Type will automatically scale their text to match the user's preferred size, and their accessibility compliance will be surfaced in the new Accessibility Nutrition Labels on the App Store.

The session covers the two phases of adoption: identifying common issues (hardcoded font sizes and fixed constraints that cause truncation and clipping at larger sizes) and adapting layouts (replacing hardcoded values with standard text styles, and conditionally restructuring layouts when the accessibility size range is active).

Both SwiftUI and UIKit patterns are covered. In SwiftUI, the `@Environment(\.dynamicTypeSize)` value enables conditional layout changes — for example, switching a carousel from 6 columns to 4 at accessibility sizes, or switching a horizontal `HStack` to a vertical `VStack`. In UIKit, `UIFont.preferredFont(forTextStyle:)` with `adjustsFontForContentSizeCategory = true` is the replacement for hardcoded `UIFont.boldSystemFont(ofSize:)` calls, and `registerForTraitChanges` handles layout updates.

## Key Topics

### Introduction: Dynamic Type on tvOS
tvOS 27 adds Large Text as a system accessibility setting that scales all text across the UI. Supporting it correctly qualifies apps for the Accessibility Nutrition Label on the App Store. The key requirement is removing hardcoded sizes and constraints that prevent text from scaling.

### Identify Common Issues
- **Fixed font sizes**: `UIFont.boldSystemFont(ofSize: 28)` or `.font(.system(size: 28))` — do not respond to Dynamic Type
- **Fixed-width frames**: `.frame(width: 300)` on text containers clips or truncates content at larger sizes
- Use `.lineLimit(1)` only when overflow is truly unacceptable; prefer allowing multiline at larger sizes
- Fixed item counts in grids/carousels do not leave enough space for larger text

### Adapt Your Layout

**Adopt standard text styles**: replace hardcoded sizes with semantic text styles (`.headline`, `.caption`, `.body`, etc.) in SwiftUI or `UIFont.preferredFont(forTextStyle:)` in UIKit.

**Use flexible constraints**: replace `.frame(width:)` with `.frame(maxWidth: .infinity)`.

**Conditional layout for accessibility sizes**: check `dynamicTypeSize.isAccessibilitySize` in SwiftUI or `traitCollection.preferredContentSizeCategory.isAccessibilityCategory` in UIKit to switch layouts — e.g., reduce columns in a LazyHStack, or switch from `HStackLayout` to `VStackLayout` for card content.

**UIKit trait-change registration**: use `registerForTraitChanges([UITraitPreferredContentSizeCategory.self], action:)` (iOS 17+ / tvOS 17+) rather than `traitCollectionDidChange` override.

## APIs & Frameworks

### SwiftUI
- `@Environment(\.dynamicTypeSize) var dynamicTypeSize` — read current Dynamic Type size **[NEW on tvOS 27]**
- `DynamicTypeSize.isAccessibilitySize` — Bool; `true` for the five accessibility size levels (AX1–AX5)
- `.font(.headline)`, `.font(.caption)`, `.font(.body)` etc. — semantic text styles that scale with Dynamic Type
- `.frame(maxWidth: .infinity, alignment:)` — flexible width constraint
- `AnyLayout` — type-erased layout container enabling conditional `VStackLayout` / `HStackLayout` switch
- `VStackLayout`, `HStackLayout` — concrete layout types usable with `AnyLayout`
- `.containerRelativeFrame(.horizontal, count:spacing:)` — responsive column sizing; pass `count` based on `isAccessibilitySize`
- `LazyHStack` — horizontal lazy layout; reduce column count at accessibility sizes

### UIKit
- `UIFont.preferredFont(forTextStyle:)` — returns a font that scales with Dynamic Type; replacement for fixed-size fonts
- `UILabel.adjustsFontForContentSizeCategory = true` — required on UIKit labels for automatic font scaling
- `UIFont.TextStyle` — `.headline`, `.caption1`, `.body`, etc.
- `UITraitPreferredContentSizeCategory` — trait type for registering layout change callbacks
- `UIViewController.registerForTraitChanges(_:action:)` — preferred tvOS 17+ API for responding to size category changes
- `UITraitCollection.preferredContentSizeCategory.isAccessibilityCategory` — Bool; mirrors `DynamicTypeSize.isAccessibilitySize`
- `UIStackView.axis` — switch `.horizontal` / `.vertical` in layout update callback

### Documentation Resources
- [Applying custom fonts to text (SwiftUI)](https://developer.apple.com/documentation/SwiftUI/Applying-Custom-Fonts-to-Text)
- [Scaling fonts automatically (UIKit)](https://developer.apple.com/documentation/UIKit/scaling-fonts-automatically)

## Code Highlights

SwiftUI: responsive carousel column count:
```swift
@Environment(\.dynamicTypeSize) private var dynamicTypeSize

.containerRelativeFrame(
    .horizontal,
    count: dynamicTypeSize.isAccessibilitySize ? 4 : 6,
    spacing: 40)
```

SwiftUI: conditional stack orientation:
```swift
let layout = dynamicTypeSize.isAccessibilitySize
    ? AnyLayout(VStackLayout(alignment: .leading, spacing: 10))
    : AnyLayout(HStackLayout(alignment: .top, spacing: 10))
layout { /* content */ }
```

UIKit: adopt Dynamic Type for a label:
```swift
titleLabel.font = UIFont.preferredFont(forTextStyle: .headline)
titleLabel.adjustsFontForContentSizeCategory = true
```

UIKit: respond to content size category changes:
```swift
registerForTraitChanges([UITraitPreferredContentSizeCategory.self],
                        action: #selector(updateLayout))

@objc private func updateLayout() {
    stackView.axis = traitCollection.preferredContentSizeCategory
        .isAccessibilityCategory ? .vertical : .horizontal
}
```

## Takeaways
- tvOS 27 adds system-wide Dynamic Type; apps that support it earn the Accessibility Nutrition Label on the App Store.
- Replace all hardcoded `ofSize:` font calls with `preferredFont(forTextStyle:)` (UIKit) or semantic SwiftUI text styles; set `adjustsFontForContentSizeCategory = true` on every UIKit label.
- Use `dynamicTypeSize.isAccessibilitySize` / `isAccessibilityCategory` to conditionally restructure layouts (fewer columns, vertical stacking) rather than just scaling text into clipped frames.
- Flexible constraints (`maxWidth: .infinity`) are essential — fixed widths are the most common cause of truncation at large sizes.

---
_Source: WWDC26 Session 221 page (abstract, chapter summaries, code samples, and resource links)._
