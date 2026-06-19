# Get Started with Dynamic Type
**WWDC24 · Session 10074** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10074/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia 15, visionOS 2, watchOS 11

## Overview
Dynamic Type allows users to choose their preferred text size across the system and all apps. This session covers the fundamentals needed to support Dynamic Type correctly — from using built-in text styles to adapting layout, images, and navigation controls for larger accessibility sizes. Supporting Dynamic Type also improves adaptability across screen sizes, orientations, and platforms.

The session explains how iOS provides 7 default text sizes plus 5 additional accessibility sizes when "Larger Accessibility Sizes" is enabled. It demonstrates how to find and fix Dynamic Type issues using Xcode Previews and the Accessibility Inspector, and shows practical SwiftUI and UIKit code patterns for building fully dynamic interfaces.

## Key Topics

**Scaled Text**
- Use built-in text styles (`.body`, `.headline`, `.title`, etc.) instead of fixed fonts so text automatically scales with user preferences
- SwiftUI: use `.font(.title)` modifier; UIKit: set `adjustsFontForContentSizeCategory = true` on `UILabel` and use `preferredFont(forTextStyle:)`
- Set `numberOfLines = 0` on labels to allow unlimited wrapping and avoid truncation
- Use Xcode Previews "Dynamic Type Variants" button to see all sizes at once; use Xcode debugger to override Dynamic Type during testing

**Dynamic Layouts**
- At accessibility text sizes, horizontal layouts often need to switch to vertical to provide enough width for text
- SwiftUI: read `@Environment(\.dynamicTypeSize)` and use `AnyLayout` to switch between `HStackLayout` and `VStackLayout` based on `isAccessibilitySize`
- UIKit: use `UIStackView` and update the `axis` property; subscribe to `UIContentSizeCategory.didChangeNotification` to respond at runtime
- `traitCollection.preferredContentSizeCategory.isAccessibilityCategory` indicates whether an accessibility size is active

**Images and Symbols**
- Prioritize scaling text over decorative images at larger sizes
- In a `List`, SwiftUI automatically wraps text under inline images
- Use `NSTextAttachment` with `NSAttributedString` in UIKit to inline images within text
- For images that should scale: use `@ScaledMetric` in SwiftUI to automatically scale a dimension; use `UIImage.SymbolConfiguration(textStyle:)` in UIKit for SF Symbols
- Consider hiding purely decorative views at the largest sizes, but never hide functional content

**Large Content Viewer**
- Allows users to inspect navigation controls (e.g., tab bars) that cannot grow with text without taking over the screen
- Standard system bars support Large Content Viewer automatically — no code needed
- SwiftUI: add `.accessibilityShowsLargeContentViewer { Label(...) }` modifier to custom bar items
- UIKit: conform the view to `UILargeContentViewerItem`, implement `showsLargeContentViewer`, `largeContentImage`, `largeContentTitle`; add `UILargeContentViewerInteraction` to the view
- Use `gestureRecognizerForExclusionRelationship` to coordinate with existing gesture recognizers

## APIs & Frameworks

**SwiftUI**
- `.font(_:)` — apply text style (`.body`, `.headline`, `.title`, `.caption`, etc.)
- `@Environment(\.dynamicTypeSize)` — read current Dynamic Type size
- `DynamicTypeSize.isAccessibilitySize` — `true` when an accessibility text size is active
- `AnyLayout` — type-erased layout for switching between stack orientations at runtime
- `HStackLayout`, `VStackLayout` — concrete layout types used with `AnyLayout`
- `@ScaledMetric` — property wrapper that scales a numeric value with Dynamic Type
- `.accessibilityShowsLargeContentViewer(_:)` **[NEW]** — add Large Content Viewer support to a SwiftUI view
- Xcode Previews: Dynamic Type Variants button

**UIKit**
- `UILabel.adjustsFontForContentSizeCategory` — automatically updates font when system text size changes
- `UIFont.preferredFont(forTextStyle:)` — returns a font for a given text style that tracks system preferences
- `UIContentSizeCategory.didChangeNotification` — notification fired when text size changes
- `UITraitCollection.preferredContentSizeCategory.isAccessibilityCategory` — check for accessibility size
- `UIStackView.axis` — switch between `.horizontal` and `.vertical`
- `NSTextAttachment` — embed images inline in `NSAttributedString`
- `NSMutableAttributedString` — build attributed strings with inline images
- `UIImage.SymbolConfiguration(textStyle:)` — create size-aware SF Symbol configuration
- `UILargeContentViewerItem` — protocol for custom views that support the Large Content Viewer
- `UILargeContentViewerInteraction` — gesture interaction for Large Content Viewer
- `UILargeContentViewerInteraction.gestureRecognizerForExclusionRelationship` — coordinate with existing recognizers
- `showsLargeContentViewer`, `largeContentImage`, `scalesLargeContentImage`, `largeContentTitle` — `UILargeContentViewerItem` properties

**Accessibility**
- Accessibility Inspector (Xcode) — audit view hierarchy for clipped text, missing labels, low contrast
- Accessibility Settings > Display & Text Size > Larger Text

## Code Highlights

SwiftUI layout switching based on accessibility size:
```swift
struct FigureCell: View {
    @Environment(\.dynamicTypeSize) private var dynamicTypeSize: DynamicTypeSize

    var dynamicLayout: AnyLayout {
        dynamicTypeSize.isAccessibilitySize ?
            AnyLayout(HStackLayout()) : AnyLayout(VStackLayout())
    }
    var body: some View {
        dynamicLayout { FigureImage(...); FigureTitle(...) }
    }
}
```

Scaling an image with `@ScaledMetric` in SwiftUI:
```swift
@ScaledMetric var imageWidth = 125.0
Image("Spatula").resizable().frame(width: imageWidth)
```

## Takeaways
- Always use **built-in text styles** rather than fixed font sizes so text scales automatically.
- Use **`AnyLayout` with `dynamicTypeSize.isAccessibilitySize`** to switch layouts when text needs more space.
- Use **`@ScaledMetric`** (SwiftUI) or **`SymbolConfiguration(textStyle:)`** (UIKit) to scale images alongside text.
- Add **Large Content Viewer** support to any custom navigation controls that cannot grow with the text.

---
_Source: WWDC24 Session 10074 page (abstract, chapter summaries, code samples, and resource links)._
