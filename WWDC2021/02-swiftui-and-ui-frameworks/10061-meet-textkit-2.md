# Meet TextKit 2
**WWDC21 · Session 10061** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10061/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12

## Overview
TextKit 2 is Apple's next-generation text engine, built on three core design principles: correctness, safety, and performance. It has been shipping silently since macOS 11 Big Sur powering system text components; now the public APIs are available for UIKit (iOS 15) and AppKit (macOS 12). TextKit 2 coexists with TextKit 1 (the original NSLayoutManager-based stack) and does not require migrating existing apps immediately.

The three principles drive three concrete architectural changes: abstracting away glyph handling (correctness), emphasizing immutable value-semantic objects (safety), and viewport-based noncontiguous layout (performance).

## Key Topics

**Correctness: No Glyph APIs**
TextKit 1 required working with glyph indices, which are incorrect for complex scripts (Arabic, Devanagari, Kannada use ligatures, reordering, and split vowels that cannot be mapped to simple glyph ranges). TextKit 2 renders all text through Core Text and exposes only character-level objects. New abstractions for selection and navigation replace glyph-index-based APIs.

**Safety: Value-Semantic Objects**
Most TextKit 2 layout objects are immutable classes (value semantics). To change layout, apps create new instances rather than mutating shared objects. The pipeline: text storage → `NSTextContentManager` (elements) → `NSTextLayoutManager` (layout fragments) → `NSTextViewportLayoutController` (viewport display).

Key objects introduced:
- `NSTextElement` — immutable building block representing a portion of content; subclass `NSTextParagraph` is the default.
- `NSTextContentManager` / `NSTextContentStorage` — wraps the backing store, enumerates elements.
- `NSTextLayoutFragment` — immutable layout+positioning info for one or more elements; exposes `textLineFragments`, `layoutFragmentFrame`, `renderingSurfaceBounds`.
- `NSTextLineFragment` — per-line typographic measurement.
- `NSTextLocation` / `NSTextRange` — object-based position types replacing integer indices; no subclassing required.
- `NSTextSelection` — immutable selection state with granularity, affinity, and disjoint ranges.
- `NSTextSelectionNavigation` — produces new `NSTextSelection` instances for tap/mouse/keyboard navigation.

**Performance: Viewport-Based Layout**
Layout is always noncontiguous in TextKit 2 — only the visible viewport and a small over-scroll region are laid out at any time. `NSTextViewportLayoutController` coordinates layout and rendering for the viewport and calls three delegate methods:
- `textViewportLayoutControllerWillLayout(_:)` — clear surfaces, open animation transaction.
- `textViewportLayoutController(_:configureRenderingSurfaceFor:)` — update geometry for each visible fragment.
- `textViewportLayoutControllerDidLayout(_:)` — commit animations, update scroll indicators.

Requests for layout outside the viewport may be inaccurate; use `ensureLayout(for:)` explicitly when needed (can be expensive for large documents).

**NSTextView / UITextField Adoption**
- `UITextField` uses TextKit 2 automatically in iOS 15.
- `UITextView` with TextKit 2 is not yet available in iOS 15 (coming in a future release).
- `NSTextView` does not auto-upgrade; opt in by creating an `NSTextLayoutManager`, associating an `NSTextContainer`, and passing that container to `NSTextView(frame:textContainer:)`.
- Accessing `NSTextView.layoutManager` (the TextKit 1 property) on a TextKit 2 text view triggers automatic fallback to TextKit 1 compatibility mode (permanent for that view).
- `NSTextField` field editors use TextKit 2 by default in macOS 12; accessing `layoutManager` on the field editor in a subclass forces all text fields in that window to TextKit 1.
- Notifications are posted before and after a view falls back to TextKit 1.

## APIs & Frameworks

### TextKit 2 — Core Objects **[NEW]**
- `NSTextElement` — abstract base; immutable, value-semantic content unit **[NEW]**
- `NSTextParagraph: NSTextElement` — default paragraph element **[NEW]**
- `NSTextContentManager` (abstract) — enumerates elements from backing store **[NEW]**
- `NSTextContentStorage: NSTextContentManager` — NSTextStorage-backed default implementation **[NEW]**
  - `performEditingTransaction(_:)` — wrap text storage mutations to notify TextKit 2 **[NEW]**
  - `delegate: NSTextContentStorageDelegate` — `textContentStorage(_:textParagraphWith:)` to inject display attributes; `textContentManager(_:shouldEnumerate:with:)` to filter elements **[NEW]**
- `NSTextLayoutManager` — layout engine; replaces NSLayoutManager (no glyph APIs) **[NEW]**
  - `textContainer: NSTextContainer`
  - `textViewportLayoutController: NSTextViewportLayoutController`
  - `delegate: NSTextLayoutManagerDelegate` — `textLayoutManager(_:textLayoutFragmentFor:in:)` **[NEW]**
- `NSTextLayoutFragment` — immutable layout info for elements **[NEW]**
  - `textLineFragments: [NSTextLineFragment]`
  - `layoutFragmentFrame: CGRect`
  - `renderingSurfaceBounds: CGRect`
  - `draw(at:in:)` — override in subclasses for custom rendering **[NEW]**
- `NSTextLineFragment` — per-line typographic measurement **[NEW]**
- `NSTextViewportLayoutController` — manages viewport layout; access via `textLayoutManager.textViewportLayoutController` **[NEW]**
  - `delegate: NSTextViewportLayoutControllerDelegate` **[NEW]**
- `NSTextLocation` — object-based location type **[NEW]**
- `NSTextRange` — range between two `NSTextLocation` instances **[NEW]**
- `NSTextSelection` — immutable selection with granularity and affinity **[NEW]**
- `NSTextSelectionNavigation` — produces new selections from user interactions **[NEW]**

### AppKit — NSTextView TextKit 2 Opt-In **[NEW]**
- `NSTextView(frame:textContainer:)` — pass TextKit 2 container to opt in **[NEW]**
- `NSTextView.textLayoutManager: NSTextLayoutManager?` — access TextKit 2 layout manager **[NEW]**
- `NSTextView.textContentStorage: NSTextContentStorage?` **[NEW]**
- `NSTextView.willSwitchToNSLayoutManagerNotification` / `didSwitchToNSLayoutManagerNotification` **[NEW]**

## Code Highlights

Opt NSTextView in to TextKit 2 (AppKit):
```swift
let textLayoutManager = NSTextLayoutManager()
let textContainer = NSTextContainer()
textLayoutManager.textContainer = textContainer
textContentStorage.addTextLayoutManager(textLayoutManager)
let textView = NSTextView(frame: bounds, textContainer: textLayoutManager.textContainer)
```

Viewport layout delegate (animate fragment layers):
```swift
func textViewportLayoutControllerWillLayout(_ controller: NSTextViewportLayoutController) {
    contentLayer.sublayers = nil
    CATransaction.begin()
}

func textViewportLayoutController(_ controller: NSTextViewportLayoutController,
                                  configureRenderingSurfaceFor fragment: NSTextLayoutFragment) {
    let layer = findOrCreateLayer(fragment)
    layer.updateGeometry()
    contentLayer.addSublayer(layer)
}

func textViewportLayoutControllerDidLayout(_ controller: NSTextViewportLayoutController) {
    CATransaction.commit()
    updateContentSizeIfNeeded()
}
```

Inject custom display attributes without modifying text storage:
```swift
func textContentStorage(_ storage: NSTextContentStorage,
                        textParagraphWith range: NSRange) -> NSTextParagraph? {
    let original = storage.textStorage!.attributedSubstring(from: range)
    guard original.attribute(.commentDepth, at: 0, effectiveRange: nil) != nil else { return nil }
    let mutable = NSMutableAttributedString(attributedString: original)
    mutable.addAttributes([.font: commentFont, .foregroundColor: commentColor],
                          range: NSRange(location: 0, length: mutable.length - 2))
    return NSTextParagraph(attributedString: mutable)
}
```

Custom NSTextLayoutFragment subclass for bubble drawing:
```swift
override func draw(at renderingOrigin: CGPoint, in ctx: CGContext) {
    ctx.saveGState()
    ctx.addPath(createBubblePath(with: ctx))
    ctx.setFillColor(bubbleColor.cgColor)
    ctx.fillPath()
    ctx.restoreGState()
    super.draw(at: renderingOrigin, in: ctx)  // draw text on top
}
```

## Takeaways
- TextKit 2 eliminates glyph-index APIs entirely — this is the key correctness fix for international scripts and is the most important migration reason for apps that currently use `NSLayoutManager` glyph APIs.
- Viewport-based noncontiguous layout is always on in TextKit 2, providing smooth scrolling through large documents without the opt-in boolean of TextKit 1.
- `NSTextContentStorageDelegate` lets apps inject custom display attributes (font, color) and filter elements (hide comments) without modifying the underlying `NSTextStorage`.
- Calling `NSTextView.layoutManager` on a TextKit 2 text view permanently reverts it to TextKit 1; audit all `layoutManager` accesses before opting in.

---
_Source: WWDC21 Session 10061 page (abstract, chapter summaries, code samples, and resource links)._
