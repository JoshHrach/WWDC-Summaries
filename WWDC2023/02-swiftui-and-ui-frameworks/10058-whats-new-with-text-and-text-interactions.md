# What's new with text and text interactions
**WWDC23 · Session 10058** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10058/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14

## Overview
Text is at the heart of every app, and iOS 17 / macOS Sonoma 14 bring a wave of improvements to how text looks, feels, and interacts. Key additions include a redesigned selection cursor with a new loupe, two new UIKit APIs for adopting system selection UI in custom text views without requiring `UITextInteraction`, a new AppKit API for bringing the system insertion indicator and dictation UI to custom macOS text views, richer interactivity for `UITextView` through customizable text item actions and menus, list and bullet support in TextKit 2, and important internationalization improvements for line-height clipping and line-break quality in CJK and other languages.

## Key Topics

**Selection UI Redesign**
- Completely redesigned text cursor across all platforms
- New inline, interactive language-switcher when changing input languages mid-composition
- More ergonomic selection handles for range selection
- All-new loupe for precise cursor placement in large documents
- Standard `UITextView` / `UITextField` get all of this automatically

**`UITextSelectionDisplayInteraction` (NEW)**
- Separates selection UI from gesture handling — unlike `UITextInteraction`, which bundles both
- Install on any `UIView` alongside an object implementing `UITextInput`
- Provides cursor view + cursor accessories, range highlight, and selection handles — all replaceable for custom styling
- Call `setNeedsSelectionUpdate()` whenever selection state changes

**`UITextLoupeSession` (NEW)**
- Standalone loupe API, independent of `UITextSelectionDisplayInteraction` / `UITextInput`
- Drive with a `UIPanGestureRecognizer`: `begin(at:in:for:)` → `move(to:in:)` → `invalidate()`

**Text Item Actions and Menus**
- `UITextView` text items now fully customizable via new `UITextViewDelegate` methods
- Text items: `NSTextAttachment` (attachments), `NSLinkAttributeName` (links), and now custom ranges via `UITextItemTagAttributeName` **[NEW]**
- Two new delegate methods: `textView(_:primaryActionFor:defaultAction:)` and `textView(_:menuConfigurationFor:textRange:)` **[NEW]**
- Return `nil` to suppress default interaction; return a `UIMenu` / `UIMenuConfiguration` to replace it
- Enables redirect of links to in-app views, custom context menus on tagged text ranges

**TextKit 2 — Lists and Bullets**
- Ordered list types: decimal, alphabetical, and roman numeral **[NEW in TextKit 2]**
- Automatic localization based on device/app locale
- Set via `NSParagraphStyle.textLists` on attributed strings
- Numbering is automatic based on newline characters; `UITextView` propagates paragraph style on typing

**Dictation on macOS Sonoma**
- New dictation indicator: trailing glow while speaking, microphone indicator at rest, scroll-away affordance when cursor leaves viewport
- `NSTextView` gets this automatically
- Custom text views: adopt `NSTextInsertionIndicator` **[NEW]** — a customizable `NSView` subclass that handles dictation effects, glow animations, and language switcher placement
- `NSTextInputClient.preferredTextAccessoryPlacement` **[NEW]** — override language switcher placement
- Notify system of scrolling: `textInputClientWillStartScrollingOrZooming()` / `willEndScrollingOrZooming()` for scroll-away indicator support
- Implement `selectionRect` and `documentVisibleRect` in `NSTextInputClient` for accurate indicator positioning

**Internationalization Improvements**
- Automatic line-height adjustment for languages with variable line heights (Thai, Hindi, etc.) in `UILabel` and `UITextField`
- Adjustment applies only when text styles (`UIFont.TextStyle`) are used, not custom fonts
- Adjustment is device-language-dependent and affects all text elements for visual consistency
- Line-breaking improvements for Chinese, German, Japanese, Korean: title text styles no longer split words mid-line
- `clipsToBounds` on text elements removed from more system defaults to prevent ascender/descender clipping

## APIs & Frameworks

**UIKit**
- `UITextSelectionDisplayInteraction` **[NEW]** — system selection UI without gesture handling
- `UITextSelectionDisplayInteraction.setNeedsSelectionUpdate()` **[NEW]**
- `UITextSelectionDisplayInteraction.cursorView` **[NEW]** — replaceable
- `UITextSelectionDisplayInteraction.highlightView` **[NEW]** — replaceable
- `UITextSelectionDisplayInteraction.handleViews` **[NEW]** — replaceable
- `UITextLoupeSession` **[NEW]**
- `UITextLoupeSession.begin(at:in:for:)` **[NEW]**
- `UITextLoupeSession.move(to:in:)` **[NEW]**
- `UITextLoupeSession.invalidate()` **[NEW]**
- `UITextItemTagAttributeName` **[NEW]** — attribute key for tagging custom text ranges as interactive
- `UITextViewDelegate.textView(_:primaryActionFor:defaultAction:)` **[NEW]**
- `UITextViewDelegate.textView(_:menuConfigurationFor:textRange:)` **[NEW]**
- `UIFont.preferredFont(forTextStyle:)` — use text styles to opt into automatic line-height adjustments (existing, now more dynamic in iOS 17)

**TextKit 2 (NSTextKit)**
- `NSParagraphStyle.textLists` — ordered and unordered list support **[NEW in TextKit 2]**
- `NSTextList` — existing class, now fully supported in TextKit 2 rendering

**AppKit**
- `NSTextInsertionIndicator` **[NEW]** — NSView subclass for system-consistent cursor and dictation effects
- `NSTextInsertionIndicator.effectsViewInserter` **[NEW]** — block for inserting system effect views into the hierarchy
- `NSTextInsertionIndicator.displayMode` **[NEW]**: `.automatic`, `.hidden`
- `NSTextInsertionIndicator.automaticModeOptions` **[NEW]**: `.showEffectsView`
- `NSTextInputClient.preferredTextAccessoryPlacement` **[NEW]** — placement hint for language switcher
- `NSTextInputClient.textInputClientWillStartScrollingOrZooming()` **[NEW]**
- `NSTextInputClient.textInputClientWillEndScrollingOrZooming()` **[NEW]**
- `NSTextInputClient.selectionRect` (existing, now required for scroll-away indicator)
- `NSTextInputClient.documentVisibleRect` (existing, now required for scroll-away indicator)

## Code Highlights

Installing `UITextSelectionDisplayInteraction`:
```swift
let selectionInteraction = UITextSelectionDisplayInteraction(textInput: myDocument, delegate: self)
myDocumentView.addInteraction(selectionInteraction)

// After selection changes:
selectionInteraction.setNeedsSelectionUpdate()
```

Managing a loupe session with a pan gesture:
```swift
var loupeSession: UITextLoupeSession?

@objc func handlePan(_ pan: UIPanGestureRecognizer) {
    let location = pan.location(in: view)
    switch pan.state {
    case .began:
        loupeSession = UITextLoupeSession.begin(at: location, in: view, for: cursorView)
    case .changed:
        loupeSession?.move(to: location, in: view)
    default:
        loupeSession?.invalidate(); loupeSession = nil
    }
}
```

Custom menu for a link tap in `UITextView`:
```swift
func textView(_ textView: UITextView,
              menuConfigurationFor textItem: UITextItem,
              defaultMenu: UIMenu) -> UITextItem.MenuConfiguration? {
    guard case .link(let url) = textItem.content else { return nil }
    let openAction = UIAction(title: "Open in App") { _ in self.openURL(url) }
    return UITextItem.MenuConfiguration(menu: UIMenu(children: [openAction]))
}
```

`NSTextInsertionIndicator` setup (macOS):
```swift
let indicator = NSTextInsertionIndicator(frame: cursorFrame)
documentView.addSubview(indicator)
indicator.effectsViewInserter = { view in
    documentView.addSubview(view)
}
```

## Takeaways
- Custom text views on iOS should adopt `UITextSelectionDisplayInteraction` to get the new cursor, loupe, and selection handles automatically — avoids implementing these independently and keeps pace with future system changes.
- Use the new `UITextViewDelegate` methods to intercept link taps and show custom context menus — replacing the old `shouldInteractWith` boolean with full menu control.
- On macOS Sonoma, replace custom cursor drawing with `NSTextInsertionIndicator` to get the dictation glow, language switcher, and scroll-away indicator for free.
- Adopt text styles (`UIFont.preferredFont(forTextStyle:)`) in all labels and text fields to automatically benefit from dynamic line-height corrections and improved CJK line-breaking in iOS 17.

---
_Source: WWDC23 Session 10058 page (abstract, chapter summaries, transcript, and resource links)._
