# Adopt Desktop-Class Editing Interactions
**WWDC22 · Session 10071** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10071/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13 (Mac Catalyst)

## Overview
iOS 16 brings a major overhaul to edit menus and introduces a new system-level find-and-replace UI, moving iPadOS closer to a true desktop-class editing experience. The edit menu gets alternate presentations based on input method (compact touch menu vs. context menu for pointer/keyboard) and gains powerful new capabilities: data detector integration with inline unit/currency conversion and smart lookup, UIMenu-based architecture for richer customization (submenus, images, size variants), and the `UIEditMenuInteraction` API replacing the deprecated `UIMenuController`.

The system find panel (`UIFindInteraction`) is available for `UITextView`, `WKWebView`, and `PDFView` with a single line of code, and can be adopted for fully custom views by conforming to `UITextSearching`. The panel adapts automatically across input methods, screen sizes, and Mac Catalyst.

## Key Topics

### New Edit Menu (UIEditMenuInteraction)
- Replaces `UIMenuController` (deprecated in iOS 16)
- Touch input: familiar compact horizontal menu with improved paging for discoverability
- Pointer/keyboard: context menu presented on secondary/right click
- Mac Catalyst: native macOS context menus
- Text menus gain data detectors, inline unit/currency conversion, and smart lookup automatically — no adoption required
- Customize text view menus via new `UITextViewDelegate`, `UITextFieldDelegate`, and `UITextInput` methods
- `UIEditMenuInteraction` can be added to any view; presentations are driven by `UIEditMenuConfiguration`
- Delegate methods: `targetRectFor` (controls menu placement), `menuFor` (injects custom actions)

### UIMenu Enhancements
- `UIMenu.preferredElementSize` — `.small`, `.medium` (side-by-side), `.large` (default full-width) **[NEW]**
- `UIMenuElement.Attributes.keepsMenuPresented` — keeps menu open after action fires **[NEW]**
- Edit menu uses `UIMenuSystem.context` for menu building

### System Find and Replace (UIFindInteraction)
- New in iOS 16; standard find panel that adapts to hardware keyboard, software keyboard, and Mac
- One-line enablement for built-in views: `textView.isFindInteractionEnabled = true`
- For custom views: install `UIFindInteraction`, conform document to `UITextSearching`, vend `UITextSearchingFindSession`
- `UIFindSession` abstract base class for custom state management
- `UITextSearching` protocol: implement `performTextSearch(queryString:options:resultAggregator:)` and `decorate(foundTextRange:document:usingStyle:)`
- `UITextSearchAggregator` — thread-safe result collector; uses `UITextRange` to represent results
- Supports multiplexing across multiple visible documents (e.g., Mail conversation view)
- `UIFindInteraction.optionsMenuProvider` — customize the search options menu

## APIs & Frameworks

**UIKit — Edit Menu** **[NEW/updated]**
- `UIEditMenuInteraction` **[NEW]** — `UIInteraction` subclass replacing UIMenuController
  - `init(delegate: UIEditMenuInteractionDelegate)`
  - `presentEditMenu(with: UIEditMenuConfiguration)`
- `UIEditMenuConfiguration` **[NEW]**
  - `init(identifier:sourcePoint:)`
  - `.sourcePoint: CGPoint`
- `UIEditMenuInteractionDelegate` protocol **[NEW]**
  - `editMenuInteraction(_:menuFor:suggestedActions:) -> UIMenu?`
  - `editMenuInteraction(_:targetRectFor:) -> CGRect`
- `UITextViewDelegate` updates:
  - `textView(_:editMenuForTextIn:suggestedActions:) -> UIMenu?` **[NEW]**
- `UITextFieldDelegate` updates:
  - `textField(_:editMenuForCharactersIn:suggestedActions:) -> UIMenu?` **[NEW]**
- `UITextInput` protocol:
  - `editMenu(for:suggestedActions:) -> UIMenu?` **[NEW]**
- `UIMenuController` — **deprecated in iOS 16**
- `UIMenu.preferredElementSize: UIMenu.ElementSize` **[NEW]** — `.small`, `.medium`, `.large`
- `UIMenuElement.Attributes.keepsMenuPresented` **[NEW]**

**UIKit — Find Interaction** **[NEW]**
- `UIFindInteraction` **[NEW]**
  - `init(sessionDelegate: UIFindInteractionDelegate)`
  - `isFindInteractionEnabled: Bool` (on UITextView, WKWebView, PDFView)
  - `presentFindNavigator(showingReplace:)` — programmatic presentation
  - `optionsMenuProvider: ((UIMenu) -> UIMenu?)?`
  - `findSession: UIFindSession?`
- `UIFindInteractionDelegate` protocol **[NEW]**
  - `findInteraction(_:sessionFor:) -> UIFindSession?`
  - `findInteractionWillBeginFind(_:)`
  - `findInteractionDidEndFind(_:)`
- `UIFindSession` (abstract) **[NEW]** — manages result state; subclass for custom implementation
- `UITextSearchingFindSession` **[NEW]** — concrete session backed by `UITextSearching`
  - `init(searchableObject: UITextSearching)`
- `UITextSearching` protocol **[NEW]**
  - `performTextSearch(queryString:options:resultAggregator:)`
  - `decorate(foundTextRange:document:usingStyle:)`
  - `clearAllDecoratedFoundText()`
  - `shouldReplace(foundTextRange:document:with:)` — optional, for replace support
  - `replace(foundTextRange:document:with:)` — optional
  - `replaceAll(queryString:options:)` — optional
- `UITextSearchAggregator` **[NEW]** — thread-safe result aggregator
- `UITextRange` — abstract base class for representing text ranges in results

## Code Highlights

Customizing a text view's edit menu:
```swift
func textView(_ textView: UITextView, editMenuForTextIn range: NSRange,
              suggestedActions: [UIMenuElement]) -> UIMenu? {
    var additionalActions: [UIMenuElement] = []
    if range.length > 0 {
        additionalActions.append(UIAction(title: "Highlight") { _ in /* ... */ })
    }
    additionalActions.append(UIAction(title: "Insert Photo") { _ in /* ... */ })
    return UIMenu(children: suggestedActions + additionalActions)
}
```

Presenting an edit menu from a gesture:
```swift
let editMenuInteraction = UIEditMenuInteraction(delegate: self)
view.addInteraction(editMenuInteraction)
// In gesture handler:
let config = UIEditMenuConfiguration(identifier: nil, sourcePoint: location)
editMenuInteraction.presentEditMenu(with: config)
```

Enabling find interaction on a custom view:
```swift
let findInteraction = UIFindInteraction(sessionDelegate: self)
customView.addInteraction(findInteraction)

func findInteraction(_ interaction: UIFindInteraction, sessionFor view: UIView) -> UIFindSession? {
    return UITextSearchingFindSession(searchableObject: customDocument)
}
```

## Takeaways
- `UIMenuController` is deprecated in iOS 16 — migrate to `UIEditMenuInteraction` and `UIMenu`-based APIs for richer, cross-platform-consistent edit menus.
- Data detectors in text menus (unit conversion, address lookup, etc.) are free — no adoption required for standard text views.
- Enable `isFindInteractionEnabled = true` on `UITextView`, `WKWebView`, or `PDFView` for an instant system find panel; custom views need only implement `UITextSearching`.
- Use `UIMenu.preferredElementSize = .medium` for side-by-side compact layouts and `.keepsMenuPresented` for repeat-action menus like indent/outdent controls.

---
_Source: WWDC22 Session 10071 page (abstract, chapter summaries, code samples, and resource links)._
