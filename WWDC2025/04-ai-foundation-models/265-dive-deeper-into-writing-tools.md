# Dive Deeper into Writing Tools
**WWDC25 · Session 265** · [Watch](https://developer.apple.com/videos/play/wwdc2025/265/)

_Platforms:_ iOS 26, iPadOS 26, macOS Tahoe 26, visionOS 26

## Overview
Building on the WWDC24 Writing Tools introduction, this session covers four advanced areas: what's new in Writing Tools (ChatGPT integration, follow-up requests, visionOS support, Shortcuts automation), native text view customization (toolbar buttons, context menu management), rich text / semantic formatting via presentation intents, and a brand-new Writing Tools Coordinator API for apps with fully custom text engines.

The Coordinator API is the centerpiece: it allows custom text engines to offer the complete Writing Tools experience — in-place rewrites, animated text transitions, and inline proofreading marks — rather than the basic panel-only experience available for free.

## Key Topics

### What's New
- **ChatGPT integration** — generate and rewrite content using ChatGPT within Writing Tools.
- **Follow-up requests** (iOS/iPadOS/macOS 26) — after a describe-and-rewrite, users can refine further ("make it warmer," "more conversational"). **[NEW]**
- **visionOS support** — Writing Tools now available on visionOS in Mail, Notes, and third-party apps. **[NEW]**
- **Shortcuts integration** — Proofread, Rewrite, and Summarize available as Shortcuts actions for automation. **[NEW]**
- **Renamed**: "Allowed Input Options" → **"Writing Tools Result Options"** for clarity.

### Native Text View Customization
- `UIBarButtonItem` / `NSToolbarItem` — add a dedicated Writing Tools toolbar button for text-heavy apps.
- `automaticallyInsertsWritingToolsItems = false` + `writingToolsItems` — control exact placement of Writing Tools items in custom context menus.
- The session recommends using `writingToolsItems` to get all future updates automatically.

### Rich Text Formatting with Presentation Intents
- **Plain text** (`plainText` result option) — Writing Tools uses `NSAttributedString` but app can ignore attributes.
- **Rich text** (`richText`) — Writing Tools may add display attributes (bold, italic, concrete font sizes).
- **Presentation intents** (`presentationIntent`) — **[NEW]** Writing Tools uses semantic style intents (heading, subheading, blockquote, table, list, code block) instead of concrete display attributes. App converts intents to its own internal styles.
  - Note: some attributes without a presentation intent representation (underline, subscript, superscript) may still appear as display attributes even in presentation intent mode.
- To give Writing Tools better understanding of the existing document, override `requestContexts` and supply contexts with presentation intents.

### Writing Tools Coordinator for Custom Text Engines
- **`NSWritingToolsCoordinator`** / **`UIWritingToolsCoordinator`** — **[NEW]** attaches to an `NSView` (AppKit) or `UIView` as a `UIInteraction` (UIKit).
- **`NSWritingToolsCoordinator.Delegate`** / **`UIWritingToolsCoordinator.Delegate`** — protocol with async methods:
  - `requestsContextsFor:scope:completion:` — provide `NSAttributedString` + selection range.
  - `replace:in:proposedText:reason:animationParameters:completion:` — incorporate text changes.
  - `select:in:context:completion:` — update selection range.
  - `requestsPreviewFor:textAnimation:of:range:in:completion:` — return `NSTextPreview` (AppKit) or `UITargetedPreview` (UIKit) rendered on clear backgrounds for animations.
  - `prepareFor:textAnimation:for:range:in:completion:` / `finish:textAnimation:` — hide/show text during animations.
  - `requestsUnderlinePathsFor:range:in:completion:` — provide `NSBezierPath` underlines for proofreading marks.
  - `requestsBoundingBezierPathsFor:range:in:completion:` — bounding paths for click/tap hit-testing.
  - `writingToolsCoordinator:willChangeToState:completion:` — optional; handle state changes (pause sync, coalesce undo).
- `updateRange:withText:` — inform coordinator of external text changes.
- `updateForReflowedText` — notify coordinator of layout changes (triggers re-request of previews and proofreading marks).

## APIs & Frameworks

### UIKit
- `UIWritingToolsCoordinator` **[NEW]**
- `UIWritingToolsCoordinator.Delegate` **[NEW]**
- `UITargetedPreview` — used for animation previews
- `UIBarButtonItem` — toolbar button placement
- `UIWritingToolsBehavior` (`.limited`, `.complete`)
- `UIWritingToolsResultOptions` (`.plainText`, `.richText`, `.list`, `.table`, `.presentationIntent`) **[NEW `.presentationIntent`]**
- `automaticallyInsertsWritingToolsItems` / `writingToolsItems`

### AppKit
- `NSWritingToolsCoordinator` **[NEW]**
- `NSWritingToolsCoordinator.Delegate` **[NEW]**
- `NSTextPreview` **[NEW]** — preview type for animation
- `NSBezierPath` — proofreading underline/bounding paths
- `NSToolbarItem` — toolbar button placement
- `NSServicesMenuRequestor` — prerequisite for basic custom engine support

### Foundation
- `NSAttributedString` — all Writing Tools text exchange
- `NSPresentationIntent` — semantic style intents (heading, list, table, code, etc.)

## Code Highlights

```swift
// AppKit — attach coordinator
func configureWritingTools() {
    guard NSWritingToolsCoordinator.isWritingToolsAvailable else { return }
    let coordinator = NSWritingToolsCoordinator(delegate: self)
    coordinator.preferredBehavior = .complete
    coordinator.preferredResultOptions = [.richText, .list]
    writingToolsCoordinator = coordinator
}
```

```swift
// Handle parent/guardian responses — update UI
for await update in CommunicationLimits.current.updates { /* ... */ }
// (See Coordinator delegate methods for text replacement and preview generation)
```

## Takeaways
- Adopt the `writingToolsItems` API for context menus to automatically receive future Writing Tools menu additions.
- If your app supports semantic styles (headings, tables, code blocks), add `.presentationIntent` to result options and supply contexts with intents — this enables Writing Tools to generate semantically appropriate content.
- For custom text engines, implement `NSWritingToolsCoordinator` / `UIWritingToolsCoordinator` to unlock in-place rewriting and inline proofreading; the basic (panel-only) experience is free if you already implement `NSServicesMenuRequestor` or `UITextInteraction`.
- Sample code referenced in the session ("Writing Tools coordinator" sample) is available on developer.apple.com.

---
_Source: WWDC25 Session 265 page (abstract, chapter summaries, code samples, and resource links)._
