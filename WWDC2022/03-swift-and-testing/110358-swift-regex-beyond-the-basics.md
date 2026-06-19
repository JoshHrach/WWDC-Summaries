# Swift Regex: Beyond the Basics
**WWDC22 · Session 110358** · [Watch](https://developer.apple.com/videos/play/wwdc2022/110358/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, watchOS 9, tvOS 16

## Overview
Swift 5.7 introduces the `Regex` type, Regex literals, and the `RegexBuilder` DSL as first-class string-processing primitives. This session goes beyond the introductory "Meet Swift Regex" talk to explore how the Regex engine executes patterns, how Foundation's date/number/URL parsers integrate directly into `RegexBuilder`, and how to extract typed values from matches using captures and transforming captures.

The session walks through a real-world example: parsing XCTest log output step-by-step, progressively improving the Regex with captures, reluctant quantifiers, transforming captures (including a custom `TestStatus` enum), Foundation's ISO 8601 date parser, and a custom `CustomConsumingRegexComponent` that calls a C standard-library parser (`strtod`).

A key theme is balancing concision and readability: Regex literals (for compact familiar syntax) and `RegexBuilder` (for readable Swift-style composition) can be freely mixed inside the same Regex.

## Key Topics

### Regex Type and Execution Model
`Regex` is a new Swift Standard Library type. At runtime, the Regex engine walks the input string matching patterns left to right; quantifiers (like `OneOrMore`, `ZeroOrMore`) are eager by default, consuming as many characters as possible.

### Regex-Powered String Algorithms
`String` gains collection-based methods: `firstMatch(of:)`, `wholeMatch(of:)`, `prefixMatch(of:)`, `starts(with:)`, `replacing(_:with:)`, `trimmingPrefix(_:)`, `split(separator:)`. Regex can now appear in Swift `switch` pattern matching.

### RegexBuilder DSL
`import RegexBuilder` gives access to the result-builder syntax. Components include `Regex { }`, `ChoiceOf { }`, `Capture { }`, `TryCapture { }`, `OneOrMore`, `ZeroOrMore`, `Optionally`, `Repeat`, character classes (`.digit`, `.whitespace`, `.word`, `.any`).

### Regex Literals and Extended Literals
`/pattern/` for inline Regex literals with compiler syntax checking and Xcode syntax highlighting. `#/pattern/#` (extended literal) allows non-semantic whitespace and multi-line patterns.

### Captures and Typed Output
`Capture { }` appends a `Substring` to the output tuple. Named captures in literals (e.g., `/(?<year>\d{4})/`) append a strongly typed `Substring`. `Capture(transform:)` converts to a custom type; `TryCapture(transform:)` removes optionality and backtracks on `nil`.

### Repetition Behaviors
Eager (default) vs. reluctant: pass `.reluctant` to any repetition (e.g., `OneOrMore(.any, .reluctant)`) or use `.repetitionBehavior(_:)` modifier to override all repetitions globally.

### Foundation Integration
Foundation date, number, currency, and URL parsers conform to `RegexComponent` and can be embedded directly in `RegexBuilder`. Examples: `.iso8601(...)`, `.date(format:)`, `.currency(code:)`, `.localizedDouble`, `.localizedInteger`.

### Custom Parsers via CustomConsumingRegexComponent
Any type conforming to `CustomConsumingRegexComponent` can plug into the Regex engine, providing an `upperBound` and typed output from a `consuming(_:startingAt:in:)` method. Demonstrated with a `CDoubleParser` wrapping the C `strtod` function.

## APIs & Frameworks

**Swift Standard Library / RegexBuilder** (all **[NEW]** in Swift 5.7)

_Core types_
- `Regex<Output>` **[NEW]** — parameterized regex type; `Output` is an inferred tuple of captures
- `Regex.Match` **[NEW]** — result of a match; `.output` property returns the typed tuple
- `RegexBuilder` module **[NEW]** — DSL for building Regex values

_RegexBuilder components_
- `Regex { }` **[NEW]** — top-level builder
- `ChoiceOf { }` **[NEW]** — alternation (matches one of several sub-patterns)
- `Capture { }` / `Capture(transform:)` **[NEW]** — appends matched substring (or transformed value) to output tuple
- `TryCapture { } transform:` **[NEW]** — like `Capture` but backtracks on nil transform result
- `OneOrMore(_:)` / `OneOrMore(_:_:)` **[NEW]** — one-or-more quantifier with optional repetition behavior
- `ZeroOrMore(_:)` **[NEW]** — zero-or-more quantifier
- `Optionally(_:)` **[NEW]** — optional match
- `Repeat(_:count:)` **[NEW]** — fixed-count repetition
- `.repetitionBehavior(_:)` modifier **[NEW]** — overrides eager/reluctant for all quantifiers in scope
- `RepetitionBehavior.eager` / `.reluctant` / `.possessive` **[NEW]** — repetition behavior enum cases

_Character classes_
- `.digit`, `.whitespace`, `.word`, `.any`, `.letter`, `.newlineSequence` **[NEW]**
- Custom character classes via `CharacterClass` literals in builder or `/[a-zA-Z0-9]/` syntax

_Regex literals_
- `/pattern/` **[NEW]** — compile-time-checked Regex literal
- `#/pattern/#` **[NEW]** — extended Regex literal (non-semantic whitespace, multi-line)

_String collection algorithms_
- `String.firstMatch(of:)` **[NEW]**
- `String.wholeMatch(of:)` **[NEW]**
- `String.prefixMatch(of:)` **[NEW]**
- `String.starts(with:)` with Regex **[NEW]**
- `String.replacing(_:with:)` with Regex **[NEW]**
- `String.trimmingPrefix(_:)` with Regex **[NEW]**
- `String.split(separator:)` with Regex **[NEW]**
- Pattern matching in `switch` with Regex via `~=` **[NEW]**

_Protocols_
- `RegexComponent` **[NEW]** — protocol that Foundation parsers and custom parsers conform to
- `CustomConsumingRegexComponent` **[NEW]** — protocol for plug-in parsers; `consuming(_:startingAt:in:)` method

_Foundation Regex components_ (all **[NEW]**)
- `.iso8601(timeZone:includingFractionalSeconds:dateTimeSeparator:)` — parses `Date`
- `.date(format:)` — `Date` with custom format string
- `.currency(code:).sign(strategy:)` — `Decimal` currency parser
- `.localizedDouble` — `Double` via Foundation localized parser
- `.localizedInteger` — `Int`
- `.url` — `URL` (new in 2022)

## Code Highlights

Using `TryCapture` with a custom enum and Foundation ISO 8601 date parser:
```swift
import RegexBuilder

enum TestStatus: String {
    case started, passed, failed
}

let regex = Regex {
    "Test Suite '"
    Capture(/[a-zA-Z][a-zA-Z0-9]*/)
    "' "
    TryCapture {
        ChoiceOf { "started"; "passed"; "failed" }
    } transform: { TestStatus(rawValue: String($0)) }
    " at "
    Capture(.iso8601(timeZone: .current, includingFractionalSeconds: true, dateTimeSeparator: .space))
    Optionally(".")
} // Regex<(Substring, Substring, TestStatus, Date)>
```

Custom `CustomConsumingRegexComponent` wrapping C's `strtod`:
```swift
import Darwin

struct CDoubleParser: CustomConsumingRegexComponent {
    typealias RegexOutput = Double
    func consuming(_ input: String, startingAt index: String.Index,
                   in bounds: Range<String.Index>) throws -> (upperBound: String.Index, output: Double)? {
        input[index...].withCString { startAddress in
            var endAddress: UnsafeMutablePointer<CChar>!
            let output = strtod(startAddress, &endAddress)
            guard endAddress > startAddress else { return nil }
            let parsedLength = startAddress.distance(to: endAddress)
            let upperBound = input.utf8.index(index, offsetBy: parsedLength)
            return (upperBound, output)
        }
    }
}
```

Regex-powered string algorithms:
```swift
let input = "name:  John Appleseed,  user_id:  100"
input.split(separator: /\s*,\s*/)         // ["name:  John Appleseed", "user_id:  100"]
input.replacing(/user_id:\s*(\d+)/, with: "456")
switch "abc" {
case /\w+/: print("It's a word!")
}
```

## Takeaways
- Regex literals (`/pattern/`) give compile-time syntax checking; `RegexBuilder` gives readable, composable Swift-style string processing; both can be freely mixed.
- `Capture` / `TryCapture` with transform closures produce strongly typed output tuples, eliminating manual substring parsing.
- Foundation's parsers (dates, numbers, currencies, URLs) plug directly into `RegexBuilder` as first-class `RegexComponent` values.
- `CustomConsumingRegexComponent` allows any existing parser — including C library functions — to participate in the Regex engine, enabling re-use of battle-tested parsing logic.

---
_Source: WWDC22 Session 110358 page (abstract, chapter summaries, code samples, and resource links)._
