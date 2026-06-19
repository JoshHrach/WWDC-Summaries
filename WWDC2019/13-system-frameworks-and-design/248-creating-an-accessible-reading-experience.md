# Creating an Accessible Reading Experience
**WWDC19 · Session 248** · [Watch](https://developer.apple.com/videos/play/wwdc2019/248/)

_Platforms:_ iOS 13, iPadOS 13

## Overview
Apps that implement custom text layout using CoreText or TextKit often miss VoiceOver compatibility because standard UIKit elements are bypassed. This session teaches the three techniques required to create a first-class VoiceOver reading experience: adopting the `UIAccessibilityReadingContent` protocol (line-by-line navigation), implementing automatic page turning (so VoiceOver's "Read All" command works across pages), and customizing speech output with `NSAttributedString` accessibility attributes (language selection and IPA pronunciation).

The session uses a page-based reader app as the running example, walking through the complete implementation of all three techniques from scratch.

## Key Topics

**Auditing with VoiceOver First**
- Add VoiceOver to the Accessibility Shortcut (Settings → Accessibility → Accessibility Shortcut → VoiceOver) for rapid enable/disable during development
- Without any accessibility implementation, dragging a finger over a custom CoreText view produces only sound effects — no speech
- This confirms VoiceOver sees no accessibility elements; the reading content protocol fixes this

**`UIAccessibilityReadingContent` Protocol**
- Adopted on a custom view that represents a single page of content
- Must first mark the view as an accessibility element: `isAccessibilityElement = true`
- Four required methods:
  1. `accessibilityLineNumber(for point: CGPoint) -> Int` — map a touch point to a line number (use `hitTest` to identify which sub-view was touched, then return a stable integer identifier)
  2. `accessibilityContent(forLineNumber lineNumber: Int) -> String?` — return the text for the given line number
  3. `accessibilityFrame(forLineNumber lineNumber: Int) -> CGRect` — return the bounding rect for the given line in screen coordinates
  4. `accessibilityPageContent() -> String?` — return the full page's text as a single string (used by Read All)
- With these implemented, dragging a finger across the view causes VoiceOver to speak and highlight the text under the finger

**Automatic Page Turning**
- VoiceOver's "Read All" command reads continuously; when it reaches the end of a page it needs to turn to the next page
- Two steps required:
  1. Add the `.causesPageTurn` accessibility trait to the page view
  2. Implement `accessibilityScroll(_ direction: UIAccessibilityScrollDirection) -> Bool`:
     - `.previous` / `.left` → ask delegate to go to previous page; on success post `UIAccessibility.pageScrolledNotification`
     - `.next` / `.right` → ask delegate to go to next page; on success post `UIAccessibility.pageScrolledNotification`
     - Return `true` if the scroll was handled, `false` if at the boundary (no more pages)
- With these in place, VoiceOver automatically turns pages and reads the entire book without user intervention

**Customizing Speech Output with NSAttributedString**
- The reading content protocol has alternate `NSAttributedString`-returning versions of the line content and page content methods:
  - `accessibilityAttributedContent(forLineNumber:) -> NSAttributedString?`
  - `accessibilityAttributedPageContent() -> NSAttributedString?`
- Accessibility string attributes allow fine-grained speech control:

  **Language selection** — use `UIAccessibility.speechAttributeLanguage` (key: `.accessibilitySpeechLanguage`) with a BCP-47 language tag (e.g., `"fr-FR"`, `"ja-JP"`); VoiceOver selects the best available voice for the specified language
  
  **IPA pronunciation** — use `UIAccessibility.speechAttributeIPANotation` (key: `.accessibilitySpeechIPANotation`) with the International Phonetic Alphabet string for a word or phrase; VoiceOver uses the IPA transcription exactly as provided

**When to Use These Techniques**
- Any custom view that renders text with CoreText, TextKit, or a canvas drawing approach bypasses standard UIKit accessibility
- WebKit-based content generally gets reading content for free via Safari/WKWebView
- PDF viewers, e-book readers, magazine apps, and document viewers are the primary use cases

## APIs & Frameworks

### UIAccessibility — Reading Content Protocol
- `UIAccessibilityReadingContent` protocol
  - `accessibilityLineNumber(for point: CGPoint) -> Int`
  - `accessibilityContent(forLineNumber lineNumber: Int) -> String?`
  - `accessibilityFrame(forLineNumber lineNumber: Int) -> CGRect`
  - `accessibilityPageContent() -> String?`
  - `accessibilityAttributedContent(forLineNumber lineNumber: Int) -> NSAttributedString?` — NSAttributedString variant
  - `accessibilityAttributedPageContent() -> NSAttributedString?` — NSAttributedString variant

### UIAccessibility — Traits and Scroll
- `UIAccessibilityTraits.causesPageTurn` — trait that signals VoiceOver this view participates in page-turning
- `UIView.accessibilityScroll(_ direction: UIAccessibilityScrollDirection) -> Bool` — implement to handle VoiceOver-initiated page turns
- `UIAccessibilityScrollDirection` — `.right`, `.left`, `.up`, `.down`, `.next`, `.previous`
- `UIAccessibility.post(notification: .pageScrolled, argument: nil)` — notify VoiceOver that a page turn has completed

### NSAttributedString Accessibility Attributes
- `NSAttributedString.Key.accessibilitySpeechLanguage` — BCP-47 language tag string; selects TTS voice
- `NSAttributedString.Key.accessibilitySpeechIPANotation` — IPA string; overrides pronunciation exactly
- `NSAttributedString.Key.accessibilitySpeechPitch` — Double (0.0–2.0); adjusts pitch for a range of text
- `NSAttributedString.Key.accessibilitySpeechPunctuation` — Bool; whether to speak punctuation marks aloud
- `NSAttributedString.Key.accessibilitySpeechQueueAnnouncement` — Bool; queue announcement after current speech

### UIKit / Accessibility Infrastructure
- `UIView.isAccessibilityElement = true` — makes a view an accessibility element (required for custom page views)
- `UIView.accessibilityTraits` — set of `UIAccessibilityTraits`
- `UIView.accessibilityFrame` — CGRect in screen coordinates (use `UIAccessibility.convertToScreenCoordinates`)
- `UIView.accessibilityLabel` — readable string for a view
- `UIAccessibility.accessibilityShortcutTriggered` / Triple-click shortcut for VoiceOver toggle during dev

## Code Highlights

Adopting `UIAccessibilityReadingContent` on a custom page view:
```swift
class PageView: UIView, UIAccessibilityReadingContent {
    enum Layout: Int {
        case image = 0, title = 1, subtitle = 2, detail = 3
    }

    override init(frame: CGRect) {
        super.init(frame: frame)
        isAccessibilityElement = true
        accessibilityTraits = [.causesPageTurn]
    }

    func accessibilityLineNumber(for point: CGPoint) -> Int {
        guard let hit = hitTest(point, with: nil) else { return 0 }
        switch hit {
        case titleLabel:   return Layout.title.rawValue
        case subtitleLabel: return Layout.subtitle.rawValue
        case detailView:   return Layout.detail.rawValue
        default:           return Layout.image.rawValue
        }
    }

    func accessibilityContent(forLineNumber lineNumber: Int) -> String? {
        switch Layout(rawValue: lineNumber) {
        case .title:    return titleLabel.accessibilityLabel
        case .subtitle: return subtitleLabel.accessibilityLabel
        case .detail:   return detailView.accessibilityLabel
        default:        return nil
        }
    }

    func accessibilityFrame(forLineNumber lineNumber: Int) -> CGRect {
        switch Layout(rawValue: lineNumber) {
        case .title:    return titleLabel.accessibilityFrame
        case .subtitle: return subtitleLabel.accessibilityFrame
        case .detail:   return detailView.accessibilityFrame
        default:        return imageView.accessibilityFrame
        }
    }

    func accessibilityPageContent() -> String? {
        return [titleLabel, subtitleLabel, detailView]
            .compactMap { $0.accessibilityLabel }
            .joined(separator: ". ")
    }
}
```

Implementing automatic page turning:
```swift
override func accessibilityScroll(_ direction: UIAccessibilityScrollDirection) -> Bool {
    switch direction {
    case .previous, .left:
        guard delegate?.goToPreviousPage() == true else { return false }
        UIAccessibility.post(notification: .pageScrolled, argument: nil)
        return true
    case .next, .right:
        guard delegate?.goToNextPage() == true else { return false }
        UIAccessibility.post(notification: .pageScrolled, argument: nil)
        return true
    default:
        return false
    }
}
```

Customizing speech with `NSAttributedString` attributes:
```swift
func accessibilityAttributedPageContent() -> NSAttributedString? {
    let text = NSMutableAttributedString(string: "Arc de Triomphe est magnifique.")
    // Speak in French
    let range = NSRange(text.string.startIndex..., in: text.string)
    text.addAttribute(.accessibilitySpeechLanguage, value: "fr-FR", range: range)
    return text
}

func ipaAttributedString(word: String, ipa: String) -> NSAttributedString {
    let str = NSMutableAttributedString(string: word)
    let range = NSRange(str.string.startIndex..., in: str.string)
    str.addAttribute(.accessibilitySpeechIPANotation, value: ipa, range: range)
    return str
}
// e.g.: ipaAttributedString(word: "Yosemite", ipa: "joʊˈsɛmɪti")
```

## Takeaways
- Any custom view that renders text without standard UILabel/UITextView needs `UIAccessibilityReadingContent` — VoiceOver cannot infer line positions or content from CoreText or canvas drawing.
- Implement all four protocol methods: line number mapping, text content per line, frame per line, and full-page content — each serves a different VoiceOver interaction mode (drag-to-read, swipe navigation, Read All).
- The `.causesPageTurn` trait + `accessibilityScroll` implementation is the minimum contract required for VoiceOver's "Read All" to work across multi-page content; missing either breaks continuous reading.
- Use `accessibilitySpeechLanguage` on NSAttributedString ranges to trigger the correct localized TTS voice for inline foreign-language passages; use `accessibilitySpeechIPANotation` for precise pronunciation of proper nouns or technical terms that generic TTS handles poorly.

---
_Source: WWDC19 Session 248 page (abstract and full transcript)._
