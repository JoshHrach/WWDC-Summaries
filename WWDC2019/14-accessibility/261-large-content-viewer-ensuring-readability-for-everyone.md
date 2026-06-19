# Large Content Viewer - Ensuring Readability for Everyone
**WWDC19 · Session 261** · [Watch](https://developer.apple.com/videos/play/wwdc2019/261/)

_Platforms:_ iOS 13, iPadOS 13

## Overview
The Large Content Viewer is an accessibility feature that surfaces enlarged versions of tab bar items and other fixed-size UI elements when a user is running one of the five Accessibility Dynamic Type sizes. Standard UIKit bars (tab bars, tool bars, navigation bars) already support it, but custom bars and fixed-size controls need explicit adoption in iOS 13.

iOS 13 introduces new API — the `UILargeContentViewerItem` protocol and `UILargeContentViewerInteraction` — so developers can enable the same system-provided heads-up display overlay for their own custom bars without building a viewer from scratch. Users long-press on a custom tab bar, drag to the desired item, and release to activate it, mirroring the built-in tab bar behavior.

The session walks through three progressive cases: a custom bar containing standard UIKit views, a custom bar with fully custom button subclasses, and the more advanced scenario of coordinating the Large Content Viewer's long-press gesture with an existing long-press gesture recognizer on the same control.

## Key Topics
- **Dynamic Type & Accessibility sizes** — seven standard sizes plus five Accessibility-only sizes; at Accessibility sizes, tab bars and other fixed-size chrome cannot grow, so the Large Content Viewer activates
- **Standard UIKit bars** — already show the Large Content Viewer; PDF images in the asset catalog should have "Preserve Vector Data" checked to render crisply at enlarged sizes; use `UIBarItem.largeContentSizeImage` for non-vector images
- **UILargeContentViewerItem protocol** — marks individual views/controls for display in the viewer; provides title and image metadata
- **UILargeContentViewerInteraction** — gesture interaction added to the container (tab bar); manages the long-press → drag → release flow
- **Gesture coordination** — when an existing long-press exists on the same container, increase its `minimumPressDuration` to give the viewer time to appear first; set up `UIGestureRecognizerDelegate` to allow simultaneous recognition
- **Delegate customization** — specify custom lift behavior, custom item-at-point lookup, and custom hosting view controller

## APIs & Frameworks
- **UIKit**
  - `UILargeContentViewerItem` protocol **[NEW]**
    - `showsLargeContentViewer: Bool` **[NEW]**
    - `largeContentTitle: String?` **[NEW]**
    - `largeContentImage: UIImage?` **[NEW]**
    - `scalesLargeContentImage: Bool` **[NEW]**
    - `largeContentImageInsets: UIEdgeInsets` **[NEW]**
  - `UILargeContentViewerInteraction` class **[NEW]**
    - `init()` / `init(delegate:)` **[NEW]**
    - `gestureRecognizer: UIGestureRecognizer` **[NEW]**
    - `isEnabled: Bool` (static) **[NEW]**
    - `UILargeContentViewerInteraction.enabledStatusDidChangeNotification` **[NEW]**
  - `UILargeContentViewerInteractionDelegate` protocol **[NEW]**
    - `largeContentViewerInteraction(_:didEndOn:point:)` **[NEW]**
    - `largeContentViewerInteraction(_:itemAt:)` **[NEW]**
    - `viewController(for:)` **[NEW]**
  - `UIView.addInteraction(_:)` — used to attach `UILargeContentViewerInteraction` to a container
  - `UIBarItem.largeContentSizeImage: UIImage?` — provide high-res image for non-vector bar item images
  - `UIBarItem.largeContentSizeImageInsets: UIEdgeInsets`
  - `UIView` already conforms to `UILargeContentViewerItem`; `UIButton` and `UILabel` provide default title/image
  - `UIGestureRecognizerDelegate.gestureRecognizer(_:shouldRecognizeSimultaneouslyWith:)` — allow parallel recognition with existing long-press
  - `UILongPressGestureRecognizer.minimumPressDuration` — increase duration when coordinating with Large Content Viewer

## Code Highlights

```swift
// Simple case: custom bar with standard UIKit views
button.showsLargeContentViewer = true
label.showsLargeContentViewer = true

let interaction = UILargeContentViewerInteraction()
customTabBar.addInteraction(interaction)
```

```swift
// Custom button subclass
class MyButton: UIButton {
    override var showsLargeContentViewer: Bool { true }
    override var largeContentTitle: String? { titleLabel?.text }
    override var largeContentImage: UIImage? { image(for: .normal) }
    override var scalesLargeContentImage: Bool { true }
}
```

```swift
// Coordinating with an existing long-press gesture
existingLongPress.minimumPressDuration = 0.5 // give viewer time to appear

existingLongPress.delegate = self

func gestureRecognizer(_ g: UIGestureRecognizer,
    shouldRecognizeSimultaneouslyWith other: UIGestureRecognizer) -> Bool {
    return other == largeContentInteraction.gestureRecognizer
}
```

## Takeaways
- Use the Large Content Viewer only for UI that genuinely cannot scale with Dynamic Type; always prefer Dynamic Type scaling first.
- Add `UILargeContentViewerInteraction` to any custom fixed-size bar; mark interactive items with `showsLargeContentViewer = true`.
- Ensure PDF/vector bar images have "Preserve Vector Data" checked; supply `largeContentSizeImage` for PNG-only assets.
- When an existing long-press coexists, increase its `minimumPressDuration` and use simultaneous gesture recognition so both features remain accessible.

---
_Source: WWDC19 Session 261 page (abstract, chapter summaries, code samples, and resource links)._
