# Size Classes and Core Components
**WWDC17 · Session 812** · [Watch](https://developer.apple.com/videos/play/wwdc2017/812/)

_Platforms:_ iOS 11

## Overview
This design-focused session lays out the three-pillar framework Apple recommends for building adaptive iOS apps: size classes, Dynamic Type, and standard UIKit elements. Rather than treating each iPhone and iPad screen size as a separate design problem, the session shows how these three tools together reduce the design surface to a manageable set of variations that UIKit handles automatically.

The talk is structured as a progressive build-up: first understand the two-dimensional size class grid (compact/regular for both width and height), then learn how Dynamic Type and readability margins produce comfortable reading experiences across devices, and finally appreciate that standard UIKit components like UITableView, navigation bars, and tab bars already encode all these adaptations — leaving app developers free to focus on what makes their app unique rather than reimplementing layout adaptation from scratch.

## Key Topics
- **Two size classes, two dimensions** — every iOS device/orientation combination is classified as either compact or regular along both width and height axes; the goal is a two-state layout system rather than per-device designs
- **Size class reference cases** — iPhone portrait = compact width, regular height; iPhone landscape (most models) = compact width, compact height; iPhone 7 Plus landscape = regular width, compact height (same as iPad, enables Split View); iPad = regular width, regular height in all orientations
- **UIKit margins** — standard margins are sized per size class; wider screens get "generous" margins; developers should not hard-code edge insets
- **Readability margins** — narrower than UIKit margins; applied to text containers to prevent over-long line lengths on large displays; dynamic — they widen as text size increases; enabled by default in UITableView and UITextView
- **Dynamic Type text styles** — ten predefined styles (Large Title, Title 1–3, Headline, Subhead, Body, Callout, Footnote, Caption 1–2); Body is the system default; iOS 11 increases Title weight from Light to Regular
- **Recommended style count** — two to three text styles per screen to maintain clear hierarchy; combining smaller/larger and bolder creates visual structure
- **Custom fonts + Dynamic Type** — **[NEW in iOS 11]** custom fonts can now participate in Dynamic Type scaling via new API; requires assigning separate point sizes per text style because font proportions differ from system fonts
- **Etsy example** — app using Dynamic Type correctly: default size and maximum Accessibility size both look polished with no layout breakage
- **UIKit standard elements** — UITableView (swipe actions, edit mode, readability margins, Dynamic Type), navigation bars, toolbars, tab bars all adapt automatically; using them reduces per-device design work
- **UIKit element scaling baseline** — all components in design resources are sized for iPhone 7 (logical points); start there then verify on other sizes
- **iPhone to iPad adaptation** — regular-width iPads use Split Views; Master view = left navigation list; Detail view = right content pane (corresponds to the drill-down destination on iPhone); iPad and compact-width layouts share the same navigation hierarchy and tab bar structure
- **Multitasking / Split Screen** — when multitasking is enabled, the app transitions between compact (narrow column) and regular (full/half) size classes depending on allocated width; layout adapts automatically if size class logic is implemented correctly

## APIs & Frameworks

### UIKit
- **`UITraitCollection.horizontalSizeClass`** / **`.verticalSizeClass`** — `.compact` or `.regular`; observed via `traitCollectionDidChange(_:)` on `UIViewController` / `UIView`
- **`UIContentSizeCategory`** — the selected Dynamic Type size; observed via `UIContentSizeCategory.didChangeNotification`
- **`UIFontTextStyle`** — enum of predefined styles: `.largeTitle` **[NEW iOS 11]**, `.title1`, `.title2`, `.title3`, `.headline`, `.subheadline`, `.body`, `.callout`, `.footnote`, `.caption1`, `.caption2`
- **`UIFont.preferredFont(forTextStyle:)`** — returns system font scaled to current Dynamic Type setting for a given style
- **`UIFont.preferredFont(forTextStyle:compatibleWith:)`** — scaled system font respecting a specific `UITraitCollection`
- **`UIFontMetrics`** — **[NEW iOS 11]** scales custom `UIFont` instances to the current Dynamic Type size for a given text style; use `UIFontMetrics(forTextStyle:).scaledFont(for:)`
- **`UILabel.adjustsFontForContentSizeCategory`** — `Bool`; when `true`, label automatically re-renders at the new size when Dynamic Type setting changes
- **`UITableView.cellLayoutMarginsFollowReadableWidth`** — `Bool`; when `true`, table cell margins follow readability margins rather than screen-edge margins
- **`readableContentGuide`** (`UIView`) — layout guide inset from edges to enforce comfortable line lengths; available on all `UIView` instances
- **`UISplitViewController`** — manages Master + Detail column layout on regular-width size classes; collapses to push-based navigation on compact width

## Code Highlights

```swift
// Apply Dynamic Type to a label
label.font = UIFont.preferredFont(forTextStyle: .body)
label.adjustsFontForContentSizeCategory = true

// Scale a custom font to Dynamic Type (iOS 11+)
let customFont = UIFont(name: "MyBrandFont-Regular", size: 17)!
let scaledFont = UIFontMetrics(forTextStyle: .body).scaledFont(for: customFont)
label.font = scaledFont
label.adjustsFontForContentSizeCategory = true

// Respond to size class changes in a view controller
override func traitCollectionDidChange(_ previousTraitCollection: UITraitCollection?) {
    super.traitCollectionDidChange(previousTraitCollection)
    if traitCollection.horizontalSizeClass == .regular {
        // Apply regular-width layout (e.g., side-by-side columns)
    } else {
        // Apply compact-width layout (single column)
    }
}

// Enable readability margins on a table view
tableView.cellLayoutMarginsFollowReadableWidth = true
```

## Takeaways
- The entire iOS device matrix collapses to two size classes (compact and regular) along each axis — designing for those two states rather than individual screen sizes is the correct mental model.
- Dynamic Type adoption is low-cost (one line per label) and high-impact: it simultaneously enables accessibility, improves cross-device layout flexibility, and makes localization easier because text containers automatically accommodate different character heights.
- `UIFontMetrics` (new in iOS 11) removes the last excuse for not using Dynamic Type with custom brand fonts — assign the correct point size per text style and let UIKit handle the rest.
- Standard UIKit components (table views, navigation bars, split view controllers) already encode size class adaptation and readability margin logic; the less custom layout code an app writes, the more it automatically benefits from future UIKit improvements.

---
_Source: WWDC17 Session 812 page (abstract, transcript, and resource links)._
