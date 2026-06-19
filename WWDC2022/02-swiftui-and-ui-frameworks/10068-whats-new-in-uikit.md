# What's new in UIKit
**WWDC22 · Session 10068** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10068/)

_Platforms:_ iOS 16, iPadOS 16, Mac Catalyst

## Overview
UIKit received its most significant productivity and desktop-class update in iOS 16, focused on three areas: powerful new navigation bars with customizable center toolbars and title menus, new and enhanced controls (UICalendarView, UIPageControl improvements, UIPasteControl), and deeper SwiftUI/UIKit integration via `UIHostingConfiguration`. The session also covered SF Symbols improvements, self-sizing cell updates, Swift Concurrency conformances, and privacy/security changes.

The navigation bar overhaul brings iPad apps closer to Mac-quality toolbars. New `UINavigationItem` styles (`.browser`, `.editor`) enable center toolbar item groups with drag-to-customize support. A new title menu supports standard document operations (duplicate, rename, move, export, print) and Mac Catalyst apps automatically adopt this as an NSToolbar integration.

TextKit 2 became the default for `UITextView` in iOS 16, and two new productivity features — Find & Replace and a redesigned Edit Menu via `UIEditMenuInteraction` — round out the desktop-class story.

## Key Topics

### Desktop-Class Navigation Bars
New `UINavigationItem.ItemStyle` values: `.navigator`, `.browser`, `.editor`. New `centerItemGroups` property for toolbar items in the center of the nav bar. Users can drag to customize and reorder items; configuration persists across launches. Overflow menu appears automatically when space is constrained. Mac Catalyst apps get NSToolbar integration automatically.

### Title Menu
New title menu with built-in support for: duplicate, move, rename, export, and print — appears when the corresponding delegate methods are implemented. Supports fully custom items. Accessed via a tap on the navigation title.

### Find and Replace
Setting `isFindInteractionEnabled = true` on `UITextView` or `WKWebView` enables system Find & Replace. Works seamlessly across multiple views and documents. Custom views can adopt it too using the `UIFindInteraction` API.

### Redesigned Edit Menu
New `UIEditMenuInteraction` replaces deprecated `UIMenuController`. Renders as a compact interactive menu on touch, and a full context menu with pointer. New API to insert actions into text view edit menus.

### UICalendarView (New)
Standalone calendar component (previously only available as an inline UIDatePicker style). Uses `NSDateComponents` (not `NSDate`) for date representation. Supports single-date, multi-date (`UICalendarSelectionMultiDate`), and single-required-date (`UICalendarSelectionSingleDate`) selection. Supports disabling individual dates and adding decorations: `.default()`, `.image(_:color:)`, `.customView(_:)`.

### UIPageControl Enhancements
Custom indicator images per page (current and non-current). Full orientation and direction customization (`direction` property: `.topToBottom`, `.bottomToTop`, `.leftToRight`, `.rightToLeft`).

### UIPasteControl (New)
A new `UIControl` subclass styled like a filled `UIButton`. Shows system paste UI to avoid the new alert for programmatic pasteboard access. Replaces custom paste buttons.

### Custom Sheet Detents
`UISheetPresentationController` now supports custom detents via `.custom { context in height }`. Can return a constant or `context.maximumDetentValue` percentage. Can assign identifiers for use with `largestUndimmedDetentIdentifier`.

### SF Symbols — Variable Symbols and New Defaults
`UIImage(systemName:variableValue:)` **[NEW]** for variable symbols (value 0–1). Symbols now default to their preferred rendering mode (e.g., device symbols default to hierarchical). `UIImage.SymbolConfiguration.preferringMonochrome()` **[NEW]** to explicitly request monochrome.

### UIHostingConfiguration — SwiftUI in UIKit Cells
`UIHostingConfiguration` **[NEW]** allows writing SwiftUI views directly as cell content configurations for `UICollectionView` and `UITableView` cells. No extra view controllers or wrapper views needed.

### Self-Sizing Cell Updates
Cells automatically resize when their content changes. `UICollectionView.selfSizingInvalidation` and `UITableView.selfSizingInvalidation` new properties. `UIListContentConfiguration` triggers invalidation automatically. Custom cells call `invalidateIntrinsicContentSize()`. `.enabledIncludingConstraints` option for Auto Layout-driven invalidation.

### Swift Concurrency Conformances
`UIImage`, `UIColor`, `UIFont`, and other immutable UIKit types now conform to `Sendable`.

### Privacy and Security
`UIDevice.name` now returns the device model name, not the user's custom name (requires entitlement for custom name). New alert (replacing banner) for programmatic pasteboard access. Setting `UIDevice.orientation` is no longer supported; use `UIViewController.preferredInterfaceOrientation` instead.

## APIs & Frameworks

**UIKit — Navigation**
- `UINavigationItem.ItemStyle` **[NEW]** — `.navigator`, `.browser`, `.editor`
- `UINavigationItem.centerItemGroups` **[NEW]** — array of customizable toolbar item groups
- `UINavigationItem.titleMenuProvider` **[NEW]** — closure returning custom title menu items
- `UIDocumentProperties` **[NEW]** — metadata for title menu (rename, move, etc.)
- `UINavigationItem.renameDelegate` **[NEW]**
- `UIWindowScene.activationConditions` — for multi-window
- `UIFindInteraction` **[NEW]** — find & replace interaction
- `UIFindInteractionDelegate` **[NEW]**
- `UITextView.isFindInteractionEnabled` **[NEW]**
- `UIEditMenuInteraction` **[NEW]** — replaces `UIMenuController` (deprecated)
- `UIEditMenuInteractionDelegate` **[NEW]**
- `UITextView.editMenuInteraction` **[NEW]**

**UIKit — Controls**
- `UICalendarView` **[NEW]** — standalone calendar component
- `UICalendarView.Decoration` **[NEW]** — `.default()`, `.image(_:color:)`, `.customView(_:)`
- `UICalendarViewDelegate` **[NEW]** — `calendarView(_:decorationFor:)`
- `UICalendarSelectionMultiDate` **[NEW]** — multi-date selection behavior
- `UICalendarSelectionSingleDate` **[NEW]** — single-date selection behavior
- `UICalendarSelectionMultiDateDelegate` **[NEW]** — `multiDateSelection(_:canSelectDate:)`
- `UIPageControl.direction` **[NEW]** — `.topToBottom`, `.bottomToTop`, `.leftToRight`, `.rightToLeft`
- `UIPageControl.preferredIndicatorImage` **[NEW]**
- `UIPageControl.preferredCurrentIndicatorImage` **[NEW]**
- `UIPasteControl` **[NEW]** — privacy-aware paste button
- `UIPasteControl.Configuration` **[NEW]**

**UIKit — Sheet Presentation**
- `UISheetPresentationController.Detent.custom(identifier:resolver:)` **[NEW]**
- `UISheetPresentationController.Detent.Identifier` — custom identifiers
- `UISheetPresentationController.DetentResolutionContext` **[NEW]** — `maximumDetentValue`

**UIKit — SF Symbols**
- `UIImage(systemName:variableValue:)` **[NEW]**
- `UIImage.SymbolConfiguration.preferringMonochrome()` **[NEW]**

**UIKit — SwiftUI Integration**
- `UIHostingConfiguration` **[NEW]** — SwiftUI view as `UIContentConfiguration` for cells
- `UIHostingController` — existing bridge (enhanced)

**UIKit — Collection/Table Views**
- `UICollectionView.selfSizingInvalidation` **[NEW]**
- `UITableView.selfSizingInvalidation` **[NEW]**
- `UICollectionReusableView.invalidateIntrinsicContentSize()` — triggers self-sizing
- `SelfSizingInvalidation.enabledIncludingConstraints` **[NEW]**

**Swift Concurrency**
- `UIImage: Sendable` **[NEW]**
- `UIColor: Sendable` **[NEW]**

## Code Highlights

UICalendarView with multi-date selection and decorations:
```swift
let calendarView = UICalendarView()
calendarView.calendar = Calendar(identifier: .gregorian)
let multiDate = UICalendarSelectionMultiDate(delegate: self)
calendarView.selectionBehavior = multiDate

func calendarView(_ calendarView: UICalendarView,
                  decorationFor dateComponents: DateComponents) -> UICalendarView.Decoration? {
    switch eventType(on: dateComponents) {
    case .none:    return nil
    case .busy:    return .default()
    case .travel:  return .image(planeImage, color: .systemOrange)
    case .party:   return .customView { MyPartyEmojiLabel() }
    }
}
```

Custom sheet detent with identifier:
```swift
sheet.detents = [
    .large(),
    .custom(identifier: .small) { context in
        0.3 * context.maximumDetentValue
    }
]
sheet.largestUndimmedDetentIdentifier = .small
```

SwiftUI cell via UIHostingConfiguration:
```swift
cell.contentConfiguration = UIHostingConfiguration {
    VStack {
        Image(systemName: "wand.and.stars").font(.title)
        Text("Like magic!").font(.title2).bold()
    }
    .foregroundStyle(.purple)
}
```

## Takeaways
- Desktop-class navigation bars with customizable center toolbars, title menus, and automatic Mac Catalyst/NSToolbar integration make iPad apps feel far more capable.
- `UICalendarView` is the new standalone calendar component, using `NSDateComponents` (not `NSDate`) and supporting decorations and flexible selection modes.
- `UIHostingConfiguration` is the easiest way to embed SwiftUI views inside UIKit collection and table view cells — just one line of code.
- Programmatic pasteboard access now triggers a user-facing permission alert; replace custom paste buttons with `UIPasteControl` to avoid the alert.

---
_Source: WWDC22 Session 10068 page (abstract, chapter summaries, code samples, and resource links)._
