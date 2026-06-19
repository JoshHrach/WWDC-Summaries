# What's new in TextKit and text views
**WWDC22 · Session 10090** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10090/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13

## Overview
iOS 16 and macOS Ventura complete Apple's transition to TextKit 2 as the universal text engine for all text controls in UIKit and AppKit. In iOS 16, `UITextView` now defaults to TextKit 2 (following `UITextField` in iOS 15). In macOS Ventura, `NSTextView` also defaults to TextKit 2. TextEdit uses TextKit 2 in both plain text and rich text modes. Most apps get this upgrade automatically with zero code changes required.

TextKit 2 brings significant improvements: viewport-based layout for high-performance rendering of large documents, correct handling of complex international scripts without glyph-level assumptions, support for modern font technologies (OpenType, Variable Fonts), and non-simple text containers with exclusion paths. New additions in this release include text list support on all platforms, even line breaking for justified paragraphs, and tree-structured text elements.

For apps that currently use TextKit 1 APIs (particularly `NSLayoutManager`), a compatibility mode automatically falls back when TextKit 1 APIs are accessed, but this is expensive and one-way. The session provides detailed migration strategies to avoid the fallback and modernize code to TextKit 2.

## Key Topics

### TextKit 2 Now Default for All Text Controls
`UITextView` (iOS 16) and `NSTextView` (macOS Ventura) now use `NSTextLayoutManager` by default. New convenience constructors (`init(usingTextLayoutManager:)`) let developers explicitly choose the text engine at initialization time. Interface Builder gains a new "Text Layout" option per text view instance.

### Non-Simple Text Containers
`NSTextContainer.exclusionPaths` now works with TextKit 2, enabling text to wrap around images or inline content with holes/gaps in the layout area.

### Even Line Breaking for Justified Text
TextKit 2's new line breaking engine distributes line lengths more evenly in justified paragraphs, reducing stretched-out lines with large inter-word spacing. No adoption required.

### Text List Support (iOS 16 New)
`NSTextList` is now available in UIKit (previously AppKit only). Used with `NSMutableParagraphStyle`, it drives numbered or bulleted list rendering in text views. New TextKit 2 types: `NSTextListElement` (a subclass of `NSTextElement`) and tree structure support on `NSTextElement` with `parent` and `childElements` properties.

### TextKit 1 Compatibility Mode
Accessing `textView.layoutManager` (or the layout manager via `textContainer`) triggers an automatic, one-way, irreversible switch from TextKit 2 to TextKit 1. Debug with breakpoint on `_UITextViewEnablingCompatibilityMode` (UIKit) or `willSwitchToNSLayoutManagerNotification` / `didSwitchToNSLayoutManagerNotification` (AppKit).

### Modernization Strategies
- Audit code for `layoutManager` property access and replace with TextKit 2 equivalents.
- Gate TextKit 1 code behind a nil check of `textView.textLayoutManager`.
- Replace glyph-based APIs (no equivalents in TextKit 2) by enumerating `NSTextLayoutFragment` and `NSTextLineFragment`.
- Convert between `NSRange` ↔ `NSTextRange` using `NSTextContentManager.location(_:offsetBy:)` and `offset(from:to:)`.

## APIs & Frameworks

**TextKit 2 (NSTextKit / UIKit / AppKit)**
- `NSTextLayoutManager` — TextKit 2 layout manager
- `NSTextContentManager` — content/backing store abstraction
- `NSTextContainer.exclusionPaths` — non-simple container support **[NEW in TK2]**
- `NSTextLayoutFragment` — layout unit for a paragraph
- `NSTextLineFragment` — single wrapped line within a layout fragment
- `NSTextElement` — base tree-structured content element **[NEW tree support]**
  - `NSTextElement.parent` **[NEW]**
  - `NSTextElement.childElements` **[NEW]**
- `NSTextListElement` **[NEW]** — element subclass for list items
- `NSTextList` — numbered or bulleted list definition (now available in UIKit **[NEW]**)
- `NSMutableParagraphStyle` — used with `NSTextList` for list formatting
- `NSTextLocation` protocol — single content location (replaces NSRange integer index)
- `NSTextRange` — start/end `NSTextLocation` pair (replaces `NSRange`)
- `NSTextContentManager.location(_:offsetBy:)` — range conversion utility
- `NSTextContentManager.offset(from:to:)` — range conversion utility
- `NSTextLayoutManager.usageBoundsForTextContainer` — replaces `NSLayoutManager.usedRect(for:)`
- `NSTextLayoutManager.renderingAttributes(for:at:)` — replaces TextKit 1 "temporary attributes"
- `NSTextLayoutManager.enumerateTextLayoutFragments(from:options:using:)` — replaces glyph enumeration
- `UITextView.init(usingTextLayoutManager:)` **[NEW]** — explicit engine selection
- `NSTextView.init(usingTextLayoutManager:)` **[NEW]** — explicit engine selection
- `NSTextAttachmentViewProvider` — UIView/NSView as text attachments (TextKit 2 only)

**Debugging**
- `_UITextViewEnablingCompatibilityMode` — breakpoint symbol for TK1 fallback in UIKit
- `NSTextView.willSwitchToNSLayoutManagerNotification` — AppKit TK1 fallback notification
- `NSTextView.didSwitchToNSLayoutManagerNotification` — AppKit TK1 fallback notification

## Code Highlights

Checking for TextKit 2 before accessing layout manager:
```swift
if let textLayoutManager = textView.textLayoutManager {
    // TextKit 2 code
} else {
    let layoutManager = textView.layoutManager  // TextKit 1 only
}
```

Counting wrapped lines using TextKit 2 layout fragment enumeration:
```swift
var numberOfLines = 0
textLayoutManager.enumerateTextLayoutFragments(
    from: textLayoutManager.documentRange.location,
    options: [.ensuresLayout]) { layoutFragment in
    numberOfLines += layoutFragment.textLineFragments.count
    return true
}
```

Converting NSRange to NSTextRange:
```swift
let start = contentManager.location(contentManager.documentRange.location,
                                     offsetBy: nsRange.location)!
let end = contentManager.location(start, offsetBy: nsRange.length)
let textRange = NSTextRange(location: start, end: end)
```

## Takeaways
- All UIKit and AppKit text controls default to TextKit 2 as of iOS 16 / macOS Ventura; most apps upgrade automatically with zero code changes.
- Accessing `textView.layoutManager` triggers a permanent one-way fallback to TextKit 1 — audit and remove or gate all such accesses.
- `NSTextList` is now available in UIKit (iOS 16), enabling programmatic bulleted and numbered lists in `UITextView`.
- Glyph-based TextKit 1 APIs have no direct equivalents in TextKit 2; replace them by enumerating `NSTextLayoutFragment` and `NSTextLineFragment` objects.

---
_Source: WWDC22 Session 10090 page (abstract, chapter summaries, code samples, and resource links)._
