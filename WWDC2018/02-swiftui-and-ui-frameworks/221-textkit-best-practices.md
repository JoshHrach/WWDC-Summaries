# TextKit Best Practices
**WWDC18 · Session 221** · [Watch](https://developer.apple.com/videos/play/wwdc2018/221/)

_Platforms:_ iOS 12, macOS Mojave 10.14

## Overview
This session provides a comprehensive guide to TextKit — the high-level text system underlying UILabel, UITextField, UITextView, NSTextField, and NSTextView. Organized into three sections, the talk covers key concepts and component architecture, practical examples of progressively complex text customizations, and best practices for correctness, performance, and security.

TextKit is presented as a layered system (model → view → controller) built on Core Text, Core Graphics, and Foundation. The session walks through a real-world Markdown editor built on AppKit to demonstrate delegation, notification observation, and subclassing as progressively more powerful (and more complex) customization approaches.

## Key Topics

### Choosing the Right Control

**UIKit decision tree:**
- Display only, no selection/scrolling → `UILabel`
- Display with selection or scrolling, or large text → `UITextView` (editing disabled)
- Secure text entry (passwords) → `UITextField`
- Single-line text input → `UITextField`
- Multi-line text input → `UITextView`

**AppKit decision tree:**
- Display only → `NSTextField` (editing and selection disabled)
- Secure text entry → `NSSecureTextField`
- Short strings / form input → `NSTextField`
- Large amounts of text → `NSTextView` (optimized for performance with large content)

**String Drawing** (`NSString.draw(in:)` / `draw(at:)`) tips:
- Use only for small amounts of static text; limit draw call frequency
- Strip extra attributes before drawing — only pass attributes needed for visual appearance
- Labels and text fields provide better caching; prefer them when drawing frequently

### TextKit Architecture (Model / View / Controller)
- **Storage (Model)**: `NSTextStorage` — subclass of `NSMutableAttributedString`; holds string data and attributes
- **Display (View)**: UILabel / UITextField / UITextView / NSTextField / NSTextView
- **Layout (Controller)**: `NSLayoutManager` — coordinates all phases; manages glyph generation and glyph layout
- **Geometry**: `NSTextContainer` — defines layout area shape (default: rectangle; customizable for non-rectangular flow)

### Text Layout Process
1. Attribute fixing — resolves inconsistencies (e.g., substitutes fonts for characters not covered by specified font)
2. Glyph generation — maps characters to glyphs (can be many-to-one or one-to-many, e.g., ligatures, combining marks)
3. Glyph layout — positions glyphs within the text container for display

### Component Configurations
- **Standard**: one `NSTextStorage` → one `NSLayoutManager` → one `NSTextContainer` → one text view
- **Multi-page/multi-column**: one storage + one layout manager + multiple text containers/views (text flows between containers)
- **Divergent views**: one storage + multiple layout managers (each with its own container/view; same text, different layouts)

### Customization Approach (Hammer Analogy)
- **Delegation** (standard hammer): most customization hooks; covers the majority of use cases
- **Notifications** (ball-peen hammer): less versatile; useful for specific events
- **Subclassing** (sledgehammer): powerful but complex; use only when delegation/notifications are insufficient

### Live Example: Markdown Journal App
- Word count using `NSTextStorage` notification (`didProcessEditing`)
- Markdown bold (`**text**`): `NSTextStorageDelegate.textStorage(_:didProcessEditing:range:changeInLength:)` — apply bold font for asterisk-delimited ranges
- Markdown code block: subclass `NSTextStorage` + subclass `NSTextBlock` (custom `drawBackground(withFrame:in:characterRange:layoutManager:)`)
- Side-by-side editor with shared text storage but independent layout managers; hide Markdown control characters on right view using `NSLayoutManagerDelegate.layoutManager(_:shouldGenerateGlyphs:properties:characterIndexes:font:forGlyphRange:)` — apply `.null` glyph property to Markdown characters without modifying text storage

### Correctness: Attributed String Attribute Reset
- Initializing `NSMutableAttributedString(string:)` with no attributes applies system default attributes (default font: Helvetica 12pt, default paragraph style)
- Mixing plain text initialization with subsequent attribute application causes unspecified attributes to reset to defaults
- **Fix option 1**: initialize from the original `NSAttributedString` (preserves existing attributes)
- **Fix option 2**: explicitly supply all required attributes in `NSMutableAttributedString(string:attributes:)`
- `NSParagraphStyle` is a "sneaky reset point": conflicting paragraph styles in the same paragraph are fixed by applying the first one to the whole paragraph — always be explicit
- **Dark mode**: use dynamic colors like `NSColor.textColor` explicitly in attributed strings; do not rely on default attributes being dark-mode aware

### Performance: Noncontinuous Layout
- Continuous layout (default on AppKit): glyphs must be generated and laid out from the beginning of the text storage to reach any visible point — expensive for large documents
- Noncontinuous layout: layout manager can generate and lay out glyphs for only the visible portion
- AppKit: `layoutManager.allowsNonContiguousLayout = true`
- UIKit (`UITextView`): enabled by default; requires `isScrollEnabled = true` (disabling scroll forces layout of all text to compute intrinsic content size, defeating the purpose)
- Do not request layout for large ranges or ranges including the end of the text when using noncontinuous layout

### Security: Limiting Text Input
- All text input is potentially untrusted; limit input length where a reasonable maximum is definable
- Pasting extremely long strings (e.g., entire novels) can freeze the app — validate before allowing
- UITextField: use `UITextFieldDelegate.textField(_:shouldChangeCharactersIn:replacementString:)` to validate
- NSTextField: use a custom `NSFormatter` subclass to validate input

## APIs & Frameworks

**UIKit**
- `UILabel` — lightweight display of small/static text
- `UITextField` — single-line input, secure text entry
- `UITextView` — multi-line display and editing; subclass of `UIScrollView`
- `UITextFieldDelegate` — `textField(_:shouldChangeCharactersIn:replacementString:)` for input validation

**AppKit**
- `NSTextField` — display and single/multi-line input; disable editing for label behavior
- `NSSecureTextField` — secure text entry
- `NSTextView` — rich text display and editing; optimized for large documents
- `NSFormatter` — validate/filter `NSTextField` input
- `NSTextBlock` / `NSTextBlock` subclass — paragraph-level background and custom drawing
- `NSParagraphStyle` / `NSMutableParagraphStyle` — `textBlocks` property

**TextKit Core**
- `NSTextStorage` — `NSMutableAttributedString` subclass; model for text + attributes
  - Required override methods: `string`, `attributes(at:effectiveRange:)`, `replaceCharacters(in:with:)`, `setAttributes(_:range:)`
- `NSTextStorageDelegate` — `textStorage(_:didProcessEditing:range:changeInLength:)`
- `NSLayoutManager` — glyph generation, glyph layout, coordinate changes between storage/container/view
  - `allowsNonContiguousLayout` — enable noncontinuous layout for performance
  - `replaceTextStorage(_:)` — swap text storage on an existing layout manager
- `NSLayoutManagerDelegate` — `layoutManager(_:shouldGenerateGlyphs:properties:characterIndexes:font:forGlyphRange:)` — intervene in glyph generation
  - `NSGlyph.nullGlyph` / `.null` property — suppress a glyph from layout without changing text storage
- `NSTextContainer` — layout area geometry

**Foundation**
- `NSAttributedString` / `NSMutableAttributedString` — text + attribute storage
- `NSMutableAttributedString(string:attributes:)` — always supply attributes when mixing with attributed sources
- `NSAttributedString.Key.font`, `.foregroundColor`, `.paragraphStyle`, `.backgroundColor`, etc.
- `NSParagraphStyle.default` — beware of default paragraph style resets

## Code Highlights

Applying bold to a range safely (preserving existing attributes):
```swift
let mutableString = NSMutableAttributedString(attributedString: originalAttributedString)
let boldDescriptor = originalFont.fontDescriptor.withSymbolicTraits(.traitBold)!
let boldFont = UIFont(descriptor: boldDescriptor, size: originalFont.pointSize)
mutableString.addAttribute(.font, value: boldFont, range: boldRange)
textView.attributedText = mutableString
```

NSTextStorage delegate for Markdown bold:
```swift
func textStorage(_ textStorage: NSTextStorage,
                 didProcessEditing editedMask: NSTextStorageEditActions,
                 range editedRange: NSRange, changeInLength delta: Int) {
    // scan for **...** and apply bold font to content range
}
```

Hiding glyphs in glyph generation:
```swift
func layoutManager(_ layoutManager: NSLayoutManager,
                   shouldGenerateGlyphs glyphs: UnsafePointer<CGGlyph>,
                   properties props: UnsafePointer<NSLayoutManager.GlyphProperty>,
                   characterIndexes charIndexes: UnsafePointer<Int>,
                   font aFont: NSFont,
                   forGlyphRange glyphRange: NSRange) -> Int {
    var modifiedProps = [NSLayoutManager.GlyphProperty](UnsafeBufferPointer(start: props, count: glyphRange.length))
    for i in 0..<glyphRange.length where isMarkdownControl(charIndexes[i]) {
        modifiedProps[i] = .null
    }
    layoutManager.setGlyphs(glyphs, properties: &modifiedProps, characterIndexes: charIndexes, font: aFont, forGlyphRange: glyphRange)
    return glyphRange.length
}
```

## Takeaways
- Use `UILabel` / `UITextField` / `UITextView` (UIKit) or their AppKit equivalents as the first tool; drop to delegation before subclassing.
- Always initialize `NSMutableAttributedString` from an existing attributed string or supply explicit attributes — never mix plain-string initialization with partial attribute application.
- Enable noncontinuous layout (`allowsNonContiguousLayout = true`) for any `NSTextView` or `UITextView` with large amounts of text, and keep `isScrollEnabled = true` on UITextView to benefit.
- Validate and limit text input from users for any field where a reasonable maximum is definable — maliciously or accidentally long input can freeze the app.

---
_Source: WWDC18 Session 221 page (abstract, full transcript, chapter summaries, and resource links)._
