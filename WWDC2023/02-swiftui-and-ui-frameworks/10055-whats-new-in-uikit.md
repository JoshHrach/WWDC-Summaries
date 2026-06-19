# What's new in UIKit
**WWDC23 · Session 10055** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10055/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14 (Catalyst), tvOS 17

## Overview
UIKit in iOS 17 delivers five key architectural improvements — Xcode preview support, a new `viewIsAppearing` lifecycle callback, an overhauled trait system, animated SF Symbol support, and a new empty-state API — alongside significant enhancements for internationalization, iPad-specific features, and a broad set of general improvements to collection views, animations, text, drag-and-drop, HDR images, page controls, and menus.

The release also deepens SwiftUI integration throughout, from `UIHostingConfiguration` for empty states to bridging custom UIKit traits with SwiftUI environment keys, and better SwiftUI-UIKit interop in general.

## Key Topics

**Key Features**
- `#Preview` macro: Xcode Previews now work directly with `UIViewController` and `UIView` subclasses — no SwiftUI wrapper needed
- `viewIsAppearing(_:)` **[NEW]**: Called after `viewWillAppear` but before `viewDidAppear`; view is in hierarchy with valid trait collection and laid out by superview; back-deploys to iOS 13
- Trait system: custom `UITrait` types, new override APIs on any view/view controller, flexible change callbacks without subclassing, SwiftUI environment key bridging
- Animated symbol images: `UIImageView.addSymbolEffect(_:)`, `removeSymbolEffect(_:)`, `setSymbolImage(_:contentTransition:)` **[NEW]**
- `UIContentUnavailableConfiguration` **[NEW]**: composable empty-state descriptions (`.empty()`, `.loading()`, `.search()`); set via `contentUnavailableConfiguration`; updated in `updateContentUnavailableConfiguration(using:)`; call `setNeedsUpdateContentUnavailableConfiguration()` to trigger updates

**Internationalization**
- Dynamic line-height adjustment: `UILabel` automatically adjusts for languages with tall ascenders/descenders (Arabic, Hindi, Thai)
- Improved line-breaking and hyphenation for Chinese, German, Japanese, Korean based on text style
- `UITraitTypesettingLanguage` **[NEW]**: indicate interface language for correct line-height and hyphenation
- Locale-specific image variants: `UIImage.SymbolConfiguration` with `locale` parameter **[NEW]**

**iPad Improvements**
- Window dragging: drag anywhere in `UINavigationBar` moves Stage Manager window; `UIWindowSceneDragInteraction` **[NEW]** for custom views
- `UISplitViewController` column-style: auto-hide sidebars in Stage Manager; overlay/displace behavior at narrow widths
- `UIDocumentViewController` **[NEW]**: base class for document content view controllers; automatic title menu, sharing, drag-and-drop, key commands
- `UIDocument` now conforms to `UINavigationItemRenameDelegate`
- Apple Pencil hover: `UIHoverGestureRecognizer` with `zOffset`, `altitude`, `azimuth` properties
- PencilKit new inks: monoline pen, fountain pen, watercolor, crayon **[NEW]**
- `PKDrawing.contentVersion`, `PKStroke.contentVersion` **[NEW]** for backward compatibility detection
- `PKCanvasView.maximumSupportedContentVersion` **[NEW]**
- Keyboard scrolling for `UIScrollView`: Page Up/Down, Home, End key support; `allowsKeyboardScrolling` **[NEW]**

**General Enhancements**
- Collection view performance: ~2x faster sort inversion, ~3x faster large deletions at 10K items
- `NSCollectionLayoutDimension.uniformAcrossSiblings` **[NEW]**: makes self-sizing items in a group adopt the height of the tallest sibling
- Spring animations: `UIView.animate(duration:bounce:)` **[NEW]**; two-parameter model (duration + bounce); system default spring with zero-argument `animate`
- Text interactions: redesigned selection loupe; system selection UI available without full `UITextInteraction`; `UITextViewDelegate` new APIs for text item primary action and menu; custom range tagging for interactions
- Status bar: default style now continuously adapts to app content color, auto-switching dark/light; can split across content regions
- Drag and drop: files dropped onto Home Screen icons open in the correct app using `CFBundleDocumentTypes`; no adoption needed
- HDR images: `UIImageView` displays ISO HDR images; `UIGraphicsImageRenderer` supports HDR; `UIImageReader` **[NEW]** for controlled loading and HDR conversion
- `UIPageControl` fractional progress: `UIPageControlProgress` **[NEW]**, `UIPageControlTimerProgress` **[NEW]** with built-in timer; `progress` property on `UIPageControl`
- Palette menus: `UIMenu` with `.displayAsPalette` option **[NEW]**; `UIMenuLeaf.selectedImage` **[NEW]** for custom selection indicators
- tvOS 17: UIKit menu APIs fully available with native tvOS appearance and behaviors

## APIs & Frameworks

**UIKit**
- `#Preview("name") { ... }` macro for `UIViewController`/`UIView` **[NEW]**
- `UIViewController.viewIsAppearing(_:)` **[NEW]** (back-deploys to iOS 13)
- Custom `UITrait` protocol **[NEW]**
- `UITraitCollection` custom trait support **[NEW]**
- `UIView.traitOverrides` / `UIViewController.traitOverrides` **[NEW]**
- `UITraitChangeObservable` **[NEW]** (flexible trait change callbacks)
- `UITraitTypesettingLanguage` **[NEW]**
- `UIImageView.addSymbolEffect(_:)` **[NEW]**
- `UIImageView.addSymbolEffect(_:options:animated:)` **[NEW]**
- `UIImageView.removeSymbolEffect(ofType:)` **[NEW]**
- `UIImageView.setSymbolImage(_:contentTransition:)` **[NEW]**
- `UIContentUnavailableConfiguration` **[NEW]**
  - `.empty()`, `.loading()`, `.search()` static configs
- `UIViewController.contentUnavailableConfiguration` **[NEW]**
- `UIViewController.updateContentUnavailableConfiguration(using:)` **[NEW]**
- `UIContentUnavailableConfigurationState` **[NEW]**
- `UIViewController.setNeedsUpdateContentUnavailableConfiguration()` **[NEW]**
- `UIWindowSceneDragInteraction` **[NEW]**
- `UIDocumentViewController` **[NEW]**
- `UIHoverGestureRecognizer.zOffset` **[NEW]**
- `PKDrawing.contentVersion`, `PKStroke.contentVersion` **[NEW]**
- `PKCanvasView.maximumSupportedContentVersion` **[NEW]**
- `UIScrollView.allowsKeyboardScrolling` **[NEW]**
- `NSCollectionLayoutDimension.uniformAcrossSiblings` **[NEW]**
- `UIView.animate(springDuration:bounce:initialSpringVelocity:delay:options:animations:completion:)` **[NEW]**
- `UIImageReader` **[NEW]**
- `UIPageControlProgress` **[NEW]**
- `UIPageControlTimerProgress` **[NEW]**
- `UIPageControl.progress` **[NEW]**
- `UIMenu.Options.displayAsPalette` **[NEW]**
- `UIMenuLeaf.selectedImage` **[NEW]**
- `UIImage.SymbolConfiguration(locale:)` **[NEW]**

## Code Highlights

Empty state with search configuration:
```swift
override func updateContentUnavailableConfiguration(
    using state: UIContentUnavailableConfigurationState
) {
    var config: UIContentUnavailableConfiguration?
    if searchResults.isEmpty {
        config = .search()
    }
    contentUnavailableConfiguration = config
}
searchResults = backingStore.results(for: query)
setNeedsUpdateContentUnavailableConfiguration()
```

Animated symbol effect:
```swift
imageView.addSymbolEffect(.bounce)          // one-shot
imageView.addSymbolEffect(.variableColor)   // continuous
imageView.removeSymbolEffect(ofType: .variableColor)
imageView.setSymbolImage(pauseImage, contentTransition: .replace.downUp)
```

UIKit Xcode preview:
```swift
#Preview("Library") {
    let controller = LibraryViewController()
    controller.displayCuratedContent = true
    return controller
}
```

## Takeaways
- Add `viewIsAppearing` to replace the common pattern of using `viewDidAppear` for geometry-dependent setup — it's available back to iOS 13.
- Adopt `UIContentUnavailableConfiguration` for empty and loading states rather than building custom placeholder views.
- Use the new trait system APIs (custom traits, override APIs, change observation) to pass data through UIKit hierarchies without singletons or notification center.
- `NSCollectionLayoutDimension.uniformAcrossSiblings` solves the long-standing problem of mismatched self-sizing cells in the same group row.

---
_Source: WWDC23 Session 10055 page (abstract, chapter summaries, code samples, and resource links)._
