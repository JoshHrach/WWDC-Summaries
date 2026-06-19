# Bring Expression to Your App with Genmoji
**WWDC24 · Session 10220** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10220/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia (Apple Intelligence; requires A17 Pro or M-series)

## Overview
Genmoji is Apple Intelligence's new system for generating custom, personalized emoji from natural language descriptions. This session covers the developer APIs needed to support Genmoji in text input views and rich text workflows—centered on the new `NSAdaptiveImageGlyph` type, which represents an image embedded inline within text at the glyph level.

The core challenge is that Genmoji are not standard Unicode characters: they are adaptive, resolution-flexible image glyphs that need to flow with text, scale appropriately, and be preserved across copy/paste, sharing, and persistence. The session walks through what changes are needed in `UITextView`/`NSTextView`, how to render Genmoji in custom drawing code, how to encode/decode them in attributed strings, and what to do if your app uses a text engine other than TextKit.

Adoption is intentionally minimal for standard text views—a single property enables support—but custom text pipelines require more work.

## Key Topics
- **`NSAdaptiveImageGlyph`** — new type representing an inline image glyph; carries a `contentDescription` (required for accessibility), unique `contentIdentifier`, and image data at multiple resolutions
- **Enabling in text views** — `UITextView.supportsAdaptiveImageGlyph = true` / `NSTextView.supportsAdaptiveImageGlyph = true`
- **`NSAttributedString` integration** — Genmoji stored via `.adaptiveImageGlyph` attribute key; `NSAttributedString.Key.adaptiveImageGlyph`
- **Custom rendering** — `NSAdaptiveImageGlyph` provides `image(for:)` with a `CGSize` to get the best-resolution image for drawing
- **Encoding/decoding** — attributed strings with Genmoji encode via `NSAttributedString.DocumentType.rtfd` or via `NSAdaptiveImageGlyph` direct archiving; JSON/plain-text pipelines must handle graceful fallback
- **Accessibility** — `contentDescription` on every `NSAdaptiveImageGlyph` is mandatory; VoiceOver reads it as the glyph's spoken representation
- **Non-TextKit text engines** — apps using custom text layout must implement `UITextInput`/`NSTextInputClient` delegate methods for insertion and deletion of adaptive image glyphs

## APIs & Frameworks
### Apple Intelligence / UIKit / AppKit
- **[NEW] `NSAdaptiveImageGlyph`** — `Foundation`; inline adaptive image for use in attributed strings
  - `contentDescription: String` — required accessibility label
  - `contentIdentifier: String` — unique ID for the glyph (stable across copies)
  - `image(for size: CGSize) -> UIImage` — renders at appropriate resolution
- **[NEW] `NSAttributedString.Key.adaptiveImageGlyph`** — attribute key; value is `NSAdaptiveImageGlyph`
- `UITextView.supportsAdaptiveImageGlyph: Bool` — **[NEW]** opt-in property (iOS 18+)
- `NSTextView.supportsAdaptiveImageGlyph: Bool` — **[NEW]** opt-in property (macOS 15+)
- `UITextViewDelegate` / `NSTextViewDelegate` — no changes needed for standard adoption
- `UITextInput` / `NSTextInputClient` — custom text engines must implement glyph insertion methods
- **`UIImage` / `NSImage`** — `NSAdaptiveImageGlyph.image(for:)` returns a `UIImage`

### Accessibility
- `contentDescription` on `NSAdaptiveImageGlyph` — VoiceOver reads this; must be set for every Genmoji
- `accessibilityAttributedLabel` — works with embedded adaptive image glyph descriptions

### Data / Persistence
- `NSAttributedString(rtfd:documentAttributes:)` — RTFD round-trips Genmoji correctly
- Custom persistence: archive `NSAdaptiveImageGlyph` via `NSKeyedArchiver` or encode/decode `contentIdentifier` + image data

## Code Highlights
```swift
import UIKit

// 1. Enable Genmoji in a UITextView
let textView = UITextView()
textView.supportsAdaptiveImageGlyph = true

// 2. Read Genmoji from an attributed string
let attrString: NSAttributedString = textView.attributedText ?? NSAttributedString()
attrString.enumerateAttribute(.adaptiveImageGlyph,
                               in: NSRange(attrString.string.startIndex..., in: attrString.string)) { value, range, _ in
    if let glyph = value as? NSAdaptiveImageGlyph {
        print("Glyph: \(glyph.contentDescription), ID: \(glyph.contentIdentifier)")
        let image = glyph.image(for: CGSize(width: 20, height: 20))
        // Use image for custom rendering
    }
}

// 3. Insert a Genmoji programmatically (if you have one)
let glyphAttr = NSAttributedString(
    attachment: NSTextAttachment(),  // placeholder if building manually
    // In practice, Genmoji are created by the system picker, not programmatically
)
```

## Takeaways
- Enabling Genmoji support requires just one property (`supportsAdaptiveImageGlyph = true`) for standard `UITextView`/`NSTextView` — no other changes for basic adoption
- `NSAdaptiveImageGlyph` must carry a meaningful `contentDescription` for every instance—this is required for accessibility compliance, not optional
- Apps with custom text engines (non-TextKit) need to implement `UITextInput` delegate methods for glyph insertion; plan for this if you render text in Metal, Canvas, or a third-party layout engine
- Serialize rich text containing Genmoji as RTFD or via `NSKeyedArchiver`—plain-text or JSON pipelines will lose the glyph data

---
_Source: WWDC24 Session 10220 page (abstract, chapter summaries, code samples, and resource links)._
