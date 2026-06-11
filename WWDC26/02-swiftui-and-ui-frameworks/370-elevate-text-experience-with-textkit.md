# Elevate Your App's Text Experience with TextKit
**WWDC26 · Session 370** · [Watch](https://developer.apple.com/videos/play/wwdc2026/370/)

_Platforms:_ iOS, iPadOS, macOS, visionOS

## Overview
This session bridges the gap between the convenience of `UITextView`/`NSTextView` and the power of the underlying TextKit 2 engine. New APIs in iOS/macOS 27 allow subclassing the framework text views and overriding viewport layout delegate methods to extend their behavior with custom UI — demonstrated with two complete examples: a code editor with line numbers and a recipe app with collapsible sections.

The session first establishes a clear mental model of TextKit's four-layer architecture (content storage → layout manager → viewport controller → view), then introduces the new `NSTextViewportRenderingSurface` protocol as the abstraction that ties custom rendering surfaces into this pipeline. It closes by covering new text attachment view provider reuse policies that prevent expensive provider re-creation during editing and scroll.

Prerequisites: "Meet TextKit 2" (WWDC21) and "What's New in TextKit and Text Views" (WWDC22).

## Key Topics

### TextKit Architecture (3:09)
Four layers:
1. **`NSTextContentStorage`** — stores the attributed string and breaks it into `NSTextParagraph` elements
2. **`NSTextLayoutManager`** — produces immutable `NSTextLayoutFragment` objects (one per paragraph)
3. **`NSTextViewportLayoutController`** — coordinates with the view to render only the visible subset of fragments efficiently
4. **View / rendering surface** — the `UITextView`/`NSTextView` or custom view that draws fragments

### What's New in TextKit (9:17)
- **[NEW]** `NSTextViewportRenderingSurface` protocol — a common abstraction for any view or layer that draws a layout fragment; replaces ad-hoc per-subclass approaches
- **[NEW]** `NSTextViewportRenderingSurfaceKey` — uniquely identifies a rendering surface across layout cycles; used as a key in `NSMapTable<NSTextLayoutFragment, MyView>`
- New viewport controller delegate methods allow querying and assigning rendering surfaces during the layout pass

### Extending Framework Text Views (11:27)
`UITextView` and `NSTextView` now publicly conform to `NSTextViewportLayoutControllerDelegate`, so you can override three key delegate methods in a subclass:
- `textViewportLayoutControllerWillLayout(_:)` — set up before the layout pass
- `textViewportLayoutController(_:configureRenderingSurfaceFor:)` — assign or configure a surface per fragment; call `super` first
- `textViewportLayoutControllerDidLayout(_:)` — act on the accumulated layout results

Use these views in SwiftUI via `UIViewRepresentable` / `NSViewRepresentable`.

### Example: Code Editor with Line Numbers (12:58)
A `UITextView` subclass overrides the three delegate methods:
- `WillLayout`: resets a `lines: [CGRect]` accumulator and records the starting line number by walking `NSTextContentStorage.enumerateTextElements(from:)` to count paragraphs before the viewport
- `ConfigureRenderingSurface`: captures each fragment's bounds in the accumulator
- `DidLayout`: transforms bounds to viewport-relative coordinates and fires an `onDidLayout` closure that a container `UIView` uses to draw line number strings with `NSString.draw(at:withAttributes:)`

### Example: Collapsible Recipe Sections (17:52)
The `UITextView` subclass also conforms to `NSTextContentStorageDelegate`. The `textContentManager(_:shouldEnumerate:options:)` method returns `false` for paragraphs belonging to a collapsed section (tracked by a `Set<Int>` of header offsets). Toggling a section calls `textViewportLayoutController.delegate?.textViewportLayoutControllerReceivedSetNeedsLayout?(_:)` to force re-layout.

### Text Attachments and View Provider Reuse (19:56)
Text attachment views use the same four-layer architecture. Previously, `NSTextAttachmentViewProvider` instances were discarded and recreated on every edit or scroll. New in iOS/macOS 27: register a reuse policy on `UITextView` with `register(_:forTextAttachmentViewProviderType:)` and supply one of:
- `.onEditingInlineParagraphs` — preserve providers across edits
- `.onScrollingOutOfViewport` — cache providers when they scroll off screen

## APIs & Frameworks

**TextKit / UIKit / AppKit**
- `NSTextContentStorage` — attributed string storage with paragraph enumeration
- `NSTextParagraph` / `NSTextElement` — paragraph element type
- `NSTextContentStorage.enumerateTextElements(from:using:)` — iterate elements for line counting
- `NSTextLayoutManager` — produces `NSTextLayoutFragment` per paragraph
- `NSTextLayoutFragment` — immutable fragment with `.layoutFragmentFrame`
- `NSTextViewportLayoutController` — coordinates visible layout
- `NSTextViewportLayoutController.delegate` (now public on `UITextView`/`NSTextView`)
- **[NEW]** `NSTextViewportRenderingSurface` protocol — conformance for UIView/CALayer
- **[NEW]** `NSTextViewportRenderingSurfaceKey` — unique key per surface
- `NSMapTable<NSTextLayoutFragment, SurfaceType>` — fragment → surface cache

**`NSTextViewportLayoutControllerDelegate` methods (now public)**
- `textViewportLayoutControllerWillLayout(_:)` — **[NEW as public override point]**
- `textViewportLayoutController(_:configureRenderingSurfaceFor:)` — **[NEW as public override point]**
- `textViewportLayoutControllerDidLayout(_:)` — **[NEW as public override point]**
- `textViewportLayoutControllerReceivedSetNeedsLayout?(_:)` — trigger forced re-layout

**`NSTextContentStorageDelegate`**
- `textContentManager(_:shouldEnumerate:options:) -> Bool` — skip layout for collapsed sections

**Text Attachment APIs**
- `NSTextAttachmentViewProvider` — supplies views for inline attachments
- **[NEW]** `UITextView.register(_:forTextAttachmentViewProviderType:)` — register reuse policy
- **[NEW]** `UITextViewAttachmentReusePolicy` — `.onEditingInlineParagraphs`, `.onScrollingOutOfViewport`

**SwiftUI**
- `UIViewRepresentable` / `NSViewRepresentable` — bridge text views into SwiftUI
- `TextEditor` — built-in SwiftUI text editor (uses `UITextView`/`NSTextView` internally)

**UIKit / AppKit text views**
- `UITextView` — public `NSTextViewportLayoutControllerDelegate` conformance in iOS 27
- `NSTextView` — same on macOS 27
- `UIFont.monospacedSystemFont(ofSize:weight:)` — used for code editor

## Code Highlights

Conforming to the viewport rendering surface protocol:
```swift
class MyView: UIView, NSTextViewportRenderingSurface {}
```

Override `configureRenderingSurface` in a `UITextView` subclass:
```swift
override func textViewportLayoutController(
    _ controller: NSTextViewportLayoutController,
    configureRenderingSurfaceFor fragment: NSTextLayoutFragment
) {
    super.textViewportLayoutController(controller, configureRenderingSurfaceFor: fragment)
    lines.append(fragment.layoutFragmentFrame)
}
```

Register attachment view provider reuse:
```swift
textView.register(
    [.onEditingInlineParagraphs],
    forTextAttachmentViewProviderType: AnimatedAttachmentViewProvider.self
)
```

## Takeaways
- Start with `UITextView` / `NSTextView` as the foundation — the new public delegate hooks mean you rarely need to drop down to a fully custom TextKit implementation.
- Override the three `NSTextViewportLayoutControllerDelegate` methods (will/configure/did layout) to inject custom UI (line numbers, fold decorations, margin annotations) alongside standard text rendering.
- Use `NSTextContentStorageDelegate.textContentManager(_:shouldEnumerate:)` to hide/collapse paragraphs without altering the underlying attributed string.
- Register a reuse policy for attachment view providers to eliminate jitter and allocation spikes in documents with many inline attachments.

---
_Source: WWDC26 Session 370 page (abstract, chapter summaries, code samples, and resource links)._
