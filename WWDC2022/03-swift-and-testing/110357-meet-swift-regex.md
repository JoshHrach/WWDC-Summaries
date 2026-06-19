# Meet Swift Regex
**WWDC22 · Session 110357** · [Watch](https://developer.apple.com/videos/play/wwdc2022/110357/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, watchOS 9, tvOS 16

## Overview
Swift Regex is a new first-class feature in Swift 5.7 that brings regular expressions into the language with compile-time syntax checking, strongly-typed captures, and deep integration with Foundation parsers. It introduces two complementary ways to write patterns: concise regex literals using `/…/` delimiters (compatible with Perl, Python, Ruby, Java, and NSRegularExpression syntax), and declarative **RegexBuilder** DSL code that offers readability, structure, and reusability.

The session uses a running financial-transaction parsing example to show how Swift Regex handles common real-world challenges: ambiguous field separators, locale-sensitive date and currency parsing, Unicode canonical equivalence, and controlling backtracking behavior for performance. Swift Regex is "obsessively Unicode correct by default," matching at the Swift `Character` (extended grapheme cluster) level and respecting canonical equivalence, without compromising expressivity.

Key advances over traditional regex: interoperability with Foundation's industrial-strength parsers as regex components, named captures surfacing as Swift tuple labels, Unicode property escapes, extended delimiter syntax for whitespace-insensitive patterns, and explicit backtracking controls (`NegativeLookahead`, `Local`) that make execution predictable.

## Key Topics

### Regex Literals and Run-time Construction
- `/pattern/` literal syntax — compile-time syntax checking, syntax highlighting, strongly-typed captures
- Extended delimiters `#/…/#` — allows unescaped slashes; enables extended syntax mode (whitespace ignored for readability)
- `try Regex(runtimeString)` — run-time construction from a string; output type is `Regex<AnyRegexOutput>`

### RegexBuilder DSL
- Import `RegexBuilder` to use the declarative builder syntax
- Composable components: `OneOrMore`, `ZeroOrMore`, `CharacterClass`, `Capture`, `TryCapture`, `NegativeLookahead`, `Local`
- Regex literals can be embedded inside builders for a hybrid approach

### Foundation Parser Integration
- `One(.date(.numeric, locale:timeZone:))` — parse a `Date` directly within a regex
- `One(.localizedCurrency(code:).locale(_:))` — parse a `Decimal` currency value
- `Date.ParseStrategy` — pick parsing strategy dynamically (e.g., based on currency symbol)
- Any `CustomConsumingRegexComponent`-conforming parser can participate

### Captures and Named Captures
- `Capture { … }` — extracts matched substring or strongly-typed parsed value
- Named captures via `(?<name>…)` in literals → surface as labeled tuple elements in the output type
- `TryCapture(field) { … }` — closure actively participates in matching; returning `nil` signals failure and triggers backtracking

### Unicode Correctness
- Default matching semantics operate at Swift `Character` (extended grapheme cluster) level
- Respects Unicode canonical equivalence: "café" == "cafe\u{301}"
- `.matchingSemantics(.unicodeScalar)` — switches to scalar-level matching for sub-grapheme precision
- `\p{currencySymbol}` / `\P{currencySymbol}` — Unicode property escapes
- `\N{SPARKLING HEART}` — named Unicode scalar matching
- `.ignoresCase()` — case-insensitive matching

### Backtracking Controls
- `NegativeLookahead { … }` — peeks ahead without consuming; prevents over-matching
- `Local { … }` — creates a local backtracking scope (atomic non-capturing group); once matched, alternatives are discarded; prevents exponential backtracking on well-specified tokens

### String Algorithms
- `String.split(separator: regex)` — split using a regex pattern
- `String.replacing(_:with:)` / `.replace(_:) { match in … }` — find-and-replace with a closure
- `String.firstMatch(of:)` — find first match
- `String.matches(of:)` — find all matches

## APIs & Frameworks

**Swift Standard Library** **[NEW]**
- `Regex<Output>` struct — **[NEW]** generic over match output (including captures)
- `Regex<AnyRegexOutput>` — **[NEW]** run-time constructed regex with existential output
- `AnyRegexOutput` — **[NEW]**
- `/pattern/` regex literal syntax — **[NEW]**
- `#/pattern/#` extended delimiter syntax — **[NEW]**
- `Regex.init(_ pattern: String) throws` — **[NEW]**
- `String.split(separator: some RegexComponent)` — **[NEW]** overload
- `String.replacing(_:with:)` — **[NEW]** overload
- `String.replace(_:) { match in … }` — **[NEW]** overload
- `String.firstMatch(of:)` — **[NEW]**
- `String.matches(of:)` — **[NEW]**
- `.matchingSemantics(.unicodeScalar)` — **[NEW]**
- `.ignoresCase()` — **[NEW]**
- `RegexComponent` protocol — **[NEW]**

**RegexBuilder module** **[NEW]**
- `Regex { … }` builder — **[NEW]**
- `OneOrMore { … }` — **[NEW]**
- `ZeroOrMore { … }` — **[NEW]**
- `One(_:)` — **[NEW]**
- `Capture { … }` — **[NEW]**
- `TryCapture(_:) { … }` — **[NEW]**
- `NegativeLookahead { … }` — **[NEW]**
- `Local { … }` — **[NEW]** local backtracking scope (atomic non-capturing group)
- `CharacterClass.any` — **[NEW]**
- `CharacterClass.digit` — **[NEW]**

**Foundation** (extended for Regex)
- `Date.FormatStyle` / `Date.ParseStrategy` — extended as `RegexComponent` **[NEW]**
- `Decimal.FormatStyle.Currency` — extended as `RegexComponent` **[NEW]**
- `.date(.numeric, locale:timeZone:)` — **[NEW]** regex-compatible date parser
- `.localizedCurrency(code:).locale(_:)` — **[NEW]** regex-compatible currency parser

## Code Highlights

Split on a multi-character field separator using a regex literal:
```swift
let fragments = transaction.split(separator: /\s{2,}|\t/)
```

Full Regex builder with Foundation parsers and captures:
```swift
import RegexBuilder
let transactionMatcher = Regex {
    Capture { /CREDIT|DEBIT/ }
    fieldSeparator
    Capture { One(.date(.numeric, locale: Locale(identifier: "en_US"), timeZone: .gmt)) }
    fieldSeparator
    Capture {
        OneOrMore {
            NegativeLookahead { fieldSeparator }
            CharacterClass.any
        }
    }
    fieldSeparator
    Capture { One(.localizedCurrency(code: "USD").locale(Locale(identifier: "en_US"))) }
}
// Regex<(Substring, Substring, Date, Substring, Decimal)>
```

Named captures with extended delimiter syntax and Unicode property escape:
```swift
let regex = #/
  (?<date>     \d{2} / \d{2} / \d{4})
  (?<middle>   \P{currencySymbol}+)
  (?<currency> \p{currencySymbol})
/#
```

Local backtracking scope to prevent exponential backtracking:
```swift
let fieldSeparator = Local { /\s{2,}|\t/ }
```

## Takeaways
- Swift Regex provides compile-time–verified, Unicode-correct pattern matching integrated directly into the language; captures are strongly typed, including Foundation-parsed values like `Date` and `Decimal`.
- The RegexBuilder DSL replaces cryptic syntax with readable, composable Swift code while still allowing embedded regex literals for conciseness.
- `NegativeLookahead` and `Local` (atomic non-capturing group) are essential for preventing over-matching and exponential backtracking in production patterns.
- Always prefer real parsers (Foundation `Date.ParseStrategy`, `Decimal.FormatStyle`, etc.) over hand-rolled patterns for dates, numbers, and currencies.

---
_Source: WWDC22 Session 110357 page (abstract, chapter summaries, code samples, and resource links)._
