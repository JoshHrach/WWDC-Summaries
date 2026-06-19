# What's New in UIKit
**WWDC24 · Session 10118** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10118/)

_Platforms:_ iOS 18, iPadOS 18, macOS Catalyst, visionOS 2

## Overview
UIKit in iOS 18 delivers a broad set of improvements across key feature areas: a redesigned document launch experience, updated tab bar and sidebar APIs (using new `UITab`/`UITabGroup`), a new continuously-interactive zoom transition, deeper SwiftUI animation and gesture interoperability, automatic trait tracking, simplified list cell configuration, `UIUpdateLink` for efficient animation loops, three new SF Symbols animation presets, enhanced sensory feedback with `UICanvasFeedbackGenerator`, a rich text formatting panel with text highlight, Writing Tools integration, and Apple Pencil Pro support in UIKit and PencilKit.

## Key Topics

### Document Launch Experience
Apps built with `UIDocumentViewController` now get a redesigned document launch experience with full control over the launch view's design and first-class support for template document creation — letting users create their first document while maintaining a great browsing experience. (See "Evolve your document launch experience" for details.)

### Updated Tab Bar and Sidebar
`UITabBarController` gains new `UITab` and `UITabGroup` APIs to describe the app's tab structure. Tabs replace the older `viewControllers` array. On iPadOS 18, the tab bar has a new compact visual look, and apps can get a combined sidebar + tab bar experience where the sidebar collapses back into the tab bar when minimized. Users can reorder and hide tabs via drag and drop. The new APIs also adapt natively for Mac Catalyst and visionOS.

### Zoom Transition
A new zoom transition works with both navigation and modal presentations. The transition is continuously interactive — users can grab and drag it at any point, including mid-flight. Where a transition originates from a large cell, the zoom maintains UI continuity by keeping the same elements on screen during the transition.

### SwiftUI Animations in UIKit
`UIView.animate(_:body:)` accepts any SwiftUI `Animation` value — including custom SwiftUI animations. This enables seamless gesture-driven animations using SwiftUI spring physics: animate the view during `.changed` with `.interactiveSpring`, then transition to `.spring` on `.ended`, preserving velocity continuously.

### Coordinated Gesture Recognizers
UIKit and SwiftUI gesture systems are now unified with consistent rules. Gesture failure requirements can be expressed across frameworks by naming SwiftUI gestures (`.gesture(doubleTap, name: "SwiftUIDoubleTap")`) and referring to that name in the UIKit delegate's `shouldRequireFailureOf` check. `UIGestureRecognizerRepresentable` lets UIKit gesture recognizers be added directly to SwiftUI view hierarchies.

### Automatic Trait Tracking
UIKit now automatically tracks which traits a view or view controller reads inside supported update methods (`layoutSubviews`, `drawRect`, `updateConstraints`, etc.). When those traits change, UIKit automatically invalidates and re-calls the appropriate method — no manual `registerForTraitChanges` needed. This is always active and creates dependencies only for traits actually accessed.

### List Improvements
All views in `UICollectionView` list sections and `UITableView` now have the `listEnvironment` trait set, describing the style of the containing list. `UIListContentConfiguration` and `UIBackgroundConfiguration` automatically update their appearance to match that trait. New `UIBackgroundConfiguration` constructors: `listCell()`, `listHeader()`, `listFooter()`. New `UIListContentConfiguration` header and footer configurations. This eliminates the need to manually pass the list appearance when creating cell configurations.

### UIUpdateLink
`UIUpdateLink` is a new, smarter alternative to `CADisplayLink`. It accepts a `UIView` and auto-activates/deactivates based on the view's window visibility, automatically adapts to the display the view is on, and supports low-latency mode for drawing apps. Use `updateInfo.modelTime` for precise animation timing. Set `requiresContinuousUpdates = true` for continuous animation loops.

### SF Symbols Animations in UIKit
Three new animation presets for `UIImageView` and `UIBarButtonItem`:
- `.wiggle` — oscillates a symbol in any direction/angle to draw attention
- `.breathe` — smoothly scales a symbol up and down for ongoing activity indication
- `.rotate` — spins a designated layer around its anchor point

New `.periodic(repeatCount:delay:)` behavior for specifying repeat count and delay; new `.continuous` option for seamless repetition. The default `.replace` effect now prefers Magic Replace (smooth badge/slash transitions) with automatic fallback and a new explicit fallback API.

### Sensory Feedback
`UIFeedbackGenerator` can now be attached to a view as an interaction and accepts a location parameter for where the feedback-triggering action occurred. New `UICanvasFeedbackGenerator` is ideal for large drawing/artboard views — call `alignmentOccurred(at:)` with the gesture location. These work with Apple Pencil Pro haptics on iPad, Magic Keyboard, and audio feedback where applicable.

### Text Improvements
- **Text formatting panel**: `UITextView` gets a new Edit menu action to open a formatting panel providing fonts, sizes, lists, and other attributes. Customize with `textFormattingConfiguration` (a `UITextFormattingViewController.Configuration`). Present with `UITextFormattingViewController` and its delegate.
- **Text highlight**: New `NSAttributedString.Key.textHighlightStyle` and `.textHighlightColorScheme` attributes apply color highlights to text ranges. Five predefined color schemes plus default (tint color).
- **Writing Tools**: Editable `UITextView` instances get full inline proofreading and composition. Non-editable text views get an overlay panel. Additional API for tracking and controlling the experience.

### Menu Actions
`UICommand`, `UIKeyCommand`, and `UIAction` can now be invoked by the system — including when the app runs through iPhone Mirroring and Mac keyboard shortcuts trigger UIKeyCommands.

### Apple Pencil Pro and PencilKit
- `UITouch` and `UIHoverGestureRecognizer` expose `rollAngle` (barrel-roll angle) for expressive drawing tools
- Squeeze gesture support via UIKit
- `PKToolPicker` gains API to define available tools (from custom drawing canvases, `PKCanvasView`, or a combination)

## APIs & Frameworks

**UIKit — Tabs**
- `UITab` **[NEW]** — type-safe tab description
- `UITabGroup` **[NEW]** — group of tabs with sidebar support
- `UITabBarController.tabs` property **[NEW]**
- Combined tab/sidebar mode **[NEW]** — auto-adapts on iPadOS, Mac Catalyst, visionOS

**UIKit — Transitions**
- Zoom transition (navigation + presentation) **[NEW]** — continuously interactive

**UIKit — SwiftUI Interoperability**
- `UIView.animate(_:body:)` with SwiftUI `Animation` **[NEW]**
- Cross-framework gesture failure requirements via gesture name **[NEW]**
- `UIGestureRecognizerRepresentable` **[NEW]** — add UIKit recognizers to SwiftUI hierarchies

**UIKit — Traits**
- Automatic trait tracking in `layoutSubviews`, `drawRect`, `updateConstraints`, etc. **[NEW]**

**UIKit — List / Collection View**
- `listEnvironment` trait **[NEW]** — set automatically in list sections
- `UIBackgroundConfiguration.listCell()` / `listHeader()` / `listFooter()` **[NEW]**
- `UIListContentConfiguration` header/footer configurations **[NEW]**

**UIKit — Animation**
- `UIUpdateLink` **[NEW]** — replaces `CADisplayLink` with automatic view tracking and low-latency mode
  - `requiresContinuousUpdates: Bool` **[NEW]**
  - `UIUpdateInfo.modelTime` **[NEW]**

**UIKit — SF Symbols**
- `.wiggle` / `.breathe` / `.rotate` symbol effects on `UIImageView` / `UIBarButtonItem` **[NEW]**
- `.periodic(repeatCount:delay:)` behavior **[NEW]**
- `.continuous` repeat option **[NEW]**
- Magic Replace with explicit fallback **[NEW]**

**UIKit — Feedback**
- `UIFeedbackGenerator` with view attachment and location **[NEW]**
- `UICanvasFeedbackGenerator` **[NEW]** — `alignmentOccurred(at:)`, `pathCompleted(at:)`, etc.

**UIKit — Text**
- `UITextView.textFormattingConfiguration` **[NEW]**
- `UITextFormattingViewController` + delegate **[NEW]**
- `NSAttributedString.Key.textHighlightStyle` **[NEW]**
- `NSAttributedString.Key.textHighlightColorScheme` **[NEW]**
- Writing Tools integration (automatic for editable `UITextView`) **[NEW]**

**PencilKit**
- `PKToolPicker` — custom tool list API **[NEW]**
- `UITouch.rollAngle` / `UIHoverGestureRecognizer.rollAngle` **[NEW]**

## Code Highlights

SwiftUI animation to drive UIKit gesture transition:
```swift
switch gesture.state {
case .changed:
    UIView.animate(.interactiveSpring) {
        bead.center = gesture.translation
    }
case .ended:
    UIView.animate(.spring) {
        bead.center = endOfBracelet
    }
}
```

Cross-framework gesture failure requirement:
```swift
// SwiftUI side
Circle().gesture(doubleTap, name: "SwiftUIDoubleTap")

// UIKit delegate side
func gestureRecognizer(_ gr: UIGestureRecognizer,
    shouldRequireFailureOf other: UIGestureRecognizer) -> Bool {
    other.name == "SwiftUIDoubleTap"
}
```

Automatic trait tracking (no manual registration needed):
```swift
class MyView: UIView {
    override func layoutSubviews() {
        super.layoutSubviews()
        if traitCollection.horizontalSizeClass == .compact {
            // compact layout — trait access auto-registered
        }
    }
}
```

Simplified list cell configuration:
```swift
var contentConfig = UIListContentConfiguration.cell()
let backgroundConfig = UIBackgroundConfiguration.listCell()
contentConfig.text = location.title
contentConfig.image = location.thumbnailImage
// Appearance automatically matches list style via listEnvironment trait
```

UIUpdateLink for continuous animation:
```swift
let updateLink = UIUpdateLink(view: view, actionTarget: self, selector: #selector(update))
updateLink.requiresContinuousUpdates = true
updateLink.isEnabled = true

@objc func update(updateLink: UIUpdateLink, updateInfo: UIUpdateInfo) {
    view.center.y = sin(updateInfo.modelTime) * 100 + view.bounds.midY
}
```

UICanvasFeedbackGenerator for snapping haptics:
```swift
feedbackGenerator = UICanvasFeedbackGenerator(view: view)
// On snap:
feedbackGenerator.alignmentOccurred(at: sender.location(in: view))
```

Text highlight attribute:
```swift
var attrs = [NSAttributedString.Key: Any]()
attrs[.textHighlightStyle] = NSAttributedString.TextHighlightStyle.default
attrs[.textHighlightColorScheme] = NSAttributedString.TextHighlightColorScheme.default
```

## Takeaways
- Adopt the new `UITab`/`UITabGroup` API to get iPadOS sidebar customization, Mac Catalyst adaptation, and visionOS support with one unified description of your app's tab structure.
- Use `UIView.animate(_:body:)` with SwiftUI animation types — especially `.interactiveSpring` during gestures — to make UIKit gesture-driven animations feel as fluid as SwiftUI ones.
- Delete manual `registerForTraitChanges` code: automatic trait tracking in `layoutSubviews` and other update methods replaces it with zero boilerplate and better performance.
- Adopt `UICanvasFeedbackGenerator` for drawing and alignment feedback — it works seamlessly with Apple Pencil Pro haptics and audio on all supported platforms.

---
_Source: WWDC24 Session 10118 page (abstract, chapter summaries, code samples, and resource links)._
