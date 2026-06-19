# Get Started with Writing Tools
**WWDC24 · Session 10168** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10168/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia 15

## Overview
Writing Tools is a new Apple Intelligence feature that brings AI-powered text proofreading, rewriting, and transformation to any text view in apps across iOS, iPadOS, and macOS. This session explains how Writing Tools integrates with native text views, how to control and customize its behavior, how to protect specific text ranges (like code blocks or quotes), and how to add Writing Tools support to custom text views.

The key message is that apps using `UITextView`, `NSTextView`, or `WKWebView` get Writing Tools support automatically — no code changes needed. The session then covers the APIs available for fine-tuning the experience: controlling the behavior mode, specifying accepted input formats, protecting ranges, and responding to lifecycle events.

## Key Topics

**Native Text Views (Automatic Support)**
- `UITextView`, `NSTextView`, and `WKWebView` get Writing Tools automatically — it "just works"
- `UITextView`/`NSTextView` must use TextKit 2 for full inline experience; TextKit 1 gets a limited panel-only experience
- Writing Tools may expand the user's selection to full sentences for better model context
- Rich text attributes (styles, links, attachments) are preserved in rewrites; NSTextList/NSTextTable supported for list/table transformations
- During a session, text storage is modified directly; all changes are pushed to the undo stack

**Controlling Behavior**
- `writingToolsBehavior` property on `UITextView`/`NSTextView`: `.inline` (default), `.limited` (panel only), or `.none` (opt out)
- `writingToolsAllowedInputOptions` property: `.plainText`, `.richText`, `.table` — tell Writing Tools what formats your view supports
- `WKWebViewConfiguration.writingToolsBehavior` — defaults to `.limited`; set to `.complete` for full inline experience in web views
- `UITextView.isWritingToolsActive` / `WKWebView.isWritingToolsActive` — observe whether a session is currently active

**Lifecycle Events**
- `textViewWritingToolsWillBegin(_:)` — prepare app state (e.g., pause iCloud sync, disable direct text manipulation)
- `textViewWritingToolsDidEnd(_:)` — restore app state (e.g., resume sync)
- Available on both `UITextViewDelegate` and `NSTextViewDelegate`

**Protecting Ranges**
- `textView(_:writingToolsIgnoredRangesIn:)` — new delegate method; return `[NSRange]` of ranges Writing Tools should leave unchanged (e.g., code blocks, quoted content)
- For `WKWebView`, `<blockquote>` and `<pre>` tags are automatically excluded

**Custom Text Views**
- iOS/iPadOS: adopt `UITextInteraction` to get Writing Tools in callout bar/context menu automatically; alternatively use `UITextSelectionDisplayInteraction` + `UIEditMenuInteraction`
- `UITextInput` protocol used by Writing Tools to read/write text and anchor popovers
- New optional `UITextInput.isEditable` property — indicate whether the custom view supports editing
- macOS: conform to `NSServicesMenuRequestor`; override `validRequestor(forSendType:returnType:)` in `NSResponder`; implement `writeSelection(to:types:)` and `readSelection(from:)` (for editable views)
- Services and Shortcuts also benefit from `NSServicesMenuRequestor` adoption

## APIs & Frameworks

**UIKit**
- `UITextView.writingToolsBehavior` **[NEW]** — `UIWritingToolsBehavior`: `.inline`, `.limited`, `.none`
- `UITextView.writingToolsAllowedInputOptions` **[NEW]** — `UIWritingToolsAllowedInputOptions`: `.plainText`, `.richText`, `.table`
- `UITextView.isWritingToolsActive` **[NEW]** — read-only property
- `UITextViewDelegate.textViewWritingToolsWillBegin(_:)` **[NEW]** — lifecycle callback
- `UITextViewDelegate.textViewWritingToolsDidEnd(_:)` **[NEW]** — lifecycle callback
- `UITextViewDelegate.textView(_:writingToolsIgnoredRangesIn:) -> [NSRange]` **[NEW]** — protect ranges
- `UITextInput.isEditable` **[NEW]** — optional property for custom text views
- `UITextInteraction` — adopt for automatic Writing Tools support in custom views
- `UITextSelectionDisplayInteraction`, `UIEditMenuInteraction` — alternative for custom view support

**AppKit**
- `NSTextView.writingToolsBehavior` **[NEW]** — `NSWritingToolsBehavior`
- `NSTextView.writingToolsAllowedInputOptions` **[NEW]**
- `NSTextView.isWritingToolsActive` **[NEW]**
- `NSTextViewDelegate.textViewWritingToolsWillBegin(_:)` **[NEW]**
- `NSTextViewDelegate.textViewWritingToolsDidEnd(_:)` **[NEW]**
- `NSTextViewDelegate.textView(_:writingToolsIgnoredRangesIn:)` **[NEW]**
- `NSServicesMenuRequestor` — protocol for macOS custom views; enables Writing Tools in context menu and Edit menu
- `NSResponder.validRequestor(forSendType:returnType:)` — override to declare supported pasteboard types
- `writeSelection(to:types:)` — send text to Writing Tools
- `readSelection(from:)` — receive rewritten text from Writing Tools

**WebKit**
- `WKWebViewConfiguration.writingToolsBehavior` **[NEW]** — `UIWritingToolsBehavior` (iOS) / `NSWritingToolsBehavior` (macOS); default `.limited`
- `WKWebView.isWritingToolsActive` **[NEW]**

**TextKit**
- TextKit 2 (`NSTextLayoutManager`) required for full inline Writing Tools experience
- `NSTextList`, `NSTextTable` — used by Writing Tools for list/table transformations

## Code Highlights

Opt out of Writing Tools for a specific text view:
```swift
textView.writingToolsBehavior = .none
textView.writingToolsAllowedInputOptions = [.plainText]
```

Pause sync during Writing Tools session (UIKit delegate):
```swift
func textViewWritingToolsWillBegin(_ textView: UITextView) {
    // pause iCloud sync
}
func textViewWritingToolsDidEnd(_ textView: UITextView) {
    // resume sync
}
```

Protect code block ranges from rewrites:
```swift
func textView(_ textView: UITextView, writingToolsIgnoredRangesIn enclosingRange: NSRange) -> [NSRange] {
    let text = textView.textStorage.attributedSubstring(from: enclosingRange)
    return rangesInappropriateForWritingTools(in: text)
}
```

## Takeaways
- Writing Tools works automatically in `UITextView`, `NSTextView`, and `WKWebView` — test your app and confirm it looks correct with no code changes.
- Opt out or limit Writing Tools for views with specialized content formats using `writingToolsBehavior` and `writingToolsAllowedInputOptions`.
- Use `textViewWritingToolsWillBegin` / `textViewWritingToolsDidEnd` to prevent sync conflicts or unintended edits during active sessions.
- For custom text views, adopt `UITextInteraction` (iOS) or `NSServicesMenuRequestor` (macOS) to get Writing Tools for free.

---
_Source: WWDC24 Session 10168 page (abstract, chapter summaries, code samples, and resource links)._
