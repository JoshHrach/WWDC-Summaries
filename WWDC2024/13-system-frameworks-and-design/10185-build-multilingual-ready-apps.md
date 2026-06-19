# Build Multilingual-Ready Apps
**WWDC24 · Session 10185** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10185/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia (localization APIs; String Catalogs available from iOS 16+/Xcode 15+)

## Overview
This session covers how to build apps that work well for users across multiple languages and scripts, with a focus on new iOS 18 features that support multilingual input and display. Apple's platforms increasingly support users who type in multiple languages simultaneously (e.g., switching between English and Chinese mid-sentence), and this session explains what developers need to do to handle that gracefully.

Key topics include updates to the multilingual keyboard and text engine, the new `localizedStandardRange(of:)` string search API, String Catalog improvements for better localization workflow, and how TextKit 2 handles complex script rendering. The session also covers how to correctly handle text marked by the system during input method editing (IME) and how to avoid common bugs in custom text input implementations.

## Key Topics
- **Multilingual keyboard** — iOS 18 improvements allowing seamless switching between languages mid-typing; developers should not intercept keyboard language state
- **`localizedStandardRange(of:)`** — new `StringProtocol` method for locale-aware, diacritic-insensitive, case-insensitive string search; replaces manual `options:` combinations
- **String Catalogs (`.xcstrings`)** — Xcode 15+ format; new in WWDC24: **"Don't Translate"** marking, **format specifier validation**, **number formatting** in format strings
- **TextKit 2** — fully adopted in iOS 18 for all `UITextView`; improvements to complex script rendering (Arabic, Hebrew, Devanagari, CJK)
- **Marked text / IME** — `UITextInput.markedTextRange`; custom text inputs must preserve marked text during composition; do not process or tokenize marked text
- **Text input traits** — `UITextInputTraits.inlinePredictionType` — control whether system shows inline predictions (e.g., for languages with character suggestions)
- **Attributed string localization** — `AttributedString` with `LocalizedStringResource` for richer localized text with formatting

## APIs & Frameworks
### String / StringProtocol (Foundation)
- **[NEW] `localizedStandardRange(of:)`** — returns `Range<String.Index>?` using locale-sensitive, case- and diacritic-insensitive search; equivalent to `range(of:options: [.caseInsensitive, .diacriticInsensitive], locale: .current)` but simpler and locale-correct
- `String.localizedStandardContains(_:)` — returns Bool; same locale-aware semantics as above
- `String.localizedCompare(_:)` — locale-sensitive sort comparison

### String Catalogs (`.xcstrings`) — Xcode 15+
- **[NEW] "Don't Translate" marking** — mark a string as intentionally untranslated (e.g., app name, brand names); prevents localization tools from flagging it as missing
- **[NEW] Format specifier validation** — Xcode validates `%@`, `%d`, `%lld` etc. in plural rules and format strings at build time
- **[NEW] Number formatting in strings** — `^[%lld](inflect: true)` — automatic grammatical inflection for numbers in supported languages (English, Spanish, others)
- `NSLocalizedString` / `LocalizedStringKey` / `LocalizedStringResource` — unchanged; String Catalog is the backing store

### UIKit — Text Input
- `UITextInput` protocol — `markedTextRange`, `setMarkedText(_:selectedRange:)`, `unmarkText()` — must be implemented correctly in custom text inputs
- `UITextInputTraits.inlinePredictionType: UITextInlinePredictionType` — `.default`, `.no`, `.yes`; controls system inline predictions shown above keyboard
- `UITextView` — fully on TextKit 2 in iOS 18; no migration needed for standard text views

### TextKit 2
- `NSTextLayoutManager` — TextKit 2's layout engine; handles complex scripts, bidirectional text, variable fonts
- `NSTextContentStorage` — backing store for TextKit 2; replaces `NSLayoutManager`
- `NSTextContainer` — defines layout area; unchanged API from TextKit 1

### AttributedString / LocalizedStringResource
- `AttributedString(localized:)` — create localized attributed strings with interpolation
- `LocalizedStringResource` — type-safe localized string reference; used in SwiftUI `Text` and App Intents

## Code Highlights
```swift
// localizedStandardRange — locale-aware search
let haystack = "Héllo Wörld"
let needle = "hello"
if let range = haystack.localizedStandardRange(of: needle) {
    // Finds "Héllo" despite case and diacritics — correct for search UIs
    print("Found at: \(range)")
}

// String Catalogs: "Don't Translate" is set in the .xcstrings file UI
// Format specifier with grammatical inflection (iOS 18)
let count = 3
let message = String(localized: "^[\(count) item](inflect: true) selected")
// En: "3 items selected"; Es: "3 elementos seleccionados"

// Custom UITextInput: preserve marked text during IME composition
class MyTextInput: UIView, UITextInput {
    func setMarkedText(_ markedText: String?, selectedRange: NSRange) {
        // Store marked text separately — do NOT process it as final input
        self.markedText = markedText
        updateDisplay()
    }

    func unmarkText() {
        // Commit marked text to final content
        if let marked = markedText {
            commitText(marked)
            markedText = nil
        }
    }
}

// Inline prediction control
textView.inlinePredictionType = .no  // Disable for code editors, password fields
```

## Takeaways
- Use `localizedStandardRange(of:)` for any user-facing text search—it correctly handles diacritics and case sensitivity for the user's locale, replacing manual `NSString.CompareOptions` combinations
- String Catalogs' "Don't Translate" feature prevents localization tools and translators from touching brand names and intentionally English strings; use it proactively
- Custom `UITextInput` implementations must correctly implement the marked text lifecycle (`setMarkedText`, `unmarkText`)—failing to do so breaks IME input for CJK, Arabic, and other composition-based scripts
- Grammatical inflection with `^[\(count) item](inflect: true)` syntax in String Catalogs automatically handles plural/grammatical agreement in supported languages at zero extra code cost

---
_Source: WWDC24 Session 10185 page (abstract, chapter summaries, code samples, and resource links)._
