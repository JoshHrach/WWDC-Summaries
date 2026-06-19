# Unlock the Power of Grammatical Agreement
**WWDC23 · Session 10153** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10153/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, tvOS 17, watchOS 10

## Overview
Foundation's grammatical agreement APIs, introduced in 2021, receive significant new capabilities in iOS 17. The session covers two major additions: dependency agreement (making words agree with other words in or out of the same string) and inclusive language support (personalizing pronouns based on a person's preferred term of address).

New locale support was added for European Portuguese and German, joining the existing Spanish, French, Italian, and Brazilian Portuguese. The `inflect: true` Markdown attribute now covers more languages, and two new Markdown attributes — `agreeWithArgument` and `agreeWithConcept` — solve a common class of agreement problems where adjectives must match nouns that are separate in the UI.

A new `TermOfAddress` type and `referentConcept` Markdown attribute make it straightforward to substitute gendered or neutral pronouns at runtime based on a user's or contact's stored preferences, without requiring separate localized strings for each variant.

## Key Topics

### Grammatical Agreement Recap
- The `inflect: true` attribute in Markdown-attributed strings inflects words to match their grammatical context (e.g., based on the user's preferred term of address).
- Supported languages expanded: Spanish, French, Italian, Brazilian Portuguese (prior years), and now European Portuguese and German.

### Dependency Agreement
- **`agreeWithArgument`**: New Markdown attribute. Inflects a word to agree with a numbered argument elsewhere in the same string. No code changes required — only string catalog annotation. Index is 1-based.
- **`agreeWithConcept`**: New Markdown attribute. Inflects a word to agree with a concept object passed in `LocalizationOptions.concepts`. Useful when the governing noun is not part of the string being formatted (e.g., a food item name affecting a food size adjective in a separate UI label).
- Both attributes are ignored on older OS versions (safe to use without version guards).

### Inclusive Language
- **`TermOfAddress`**: New type with three built-in options — `.masculine`, `.feminine`, `.neutral`.
- **`Morphology.Pronoun`**: New type for specifying exact pronoun forms (nominative, accusative, genitive, etc.) for a given language; enables fully custom pronoun sets.
- **`referentConcept` Markdown attribute**: Marks a third-person personal pronoun so the engine replaces it with the appropriate form from the `TermOfAddress` concept at index N.
- Pass a `TermOfAddress` as a concept via `LocalizationOptions.concepts` using the `.termsOfAddress(_:)` concept type.
- Works in English and can be combined with other agreement concepts.

### String Catalog Integration
- All new Markdown attributes (`agreeWithArgument`, `agreeWithConcept`, `referentConcept`) are editable directly inside Xcode String Catalogs.
- Demo used String Catalogs to annotate Spanish strings for food-ordering UI.

## APIs & Frameworks

- `AttributedString.LocalizationOptions` — existing type, extended with new `concepts` property **[NEW]**
- `AttributedString.LocalizationOptions.concepts` **[NEW]** — array of `LocalizationOptions.Concept` values affecting inflection
- `LocalizationOptions.Concept.localizedPhrase(_:)` **[NEW]** — concept that agrees with an arbitrary localized string value
- `LocalizationOptions.Concept.termsOfAddress(_:)` **[NEW]** — concept that provides term-of-address data for pronoun substitution
- `TermOfAddress` **[NEW]** — type representing masculine, feminine, or neutral term of address
- `TermOfAddress.masculine` **[NEW]**
- `TermOfAddress.feminine` **[NEW]**
- `TermOfAddress.neutral` **[NEW]**
- `TermOfAddress` (localized) **[NEW]** — localized variant specifying language and explicit `Morphology.Pronoun` list
- `Morphology.Pronoun` **[NEW]** — specifies a single pronoun form and its morphological context
- Markdown attribute `inflect: true` (existing) — inflects enclosed words
- Markdown attribute `agreeWithArgument: N` **[NEW]** — inflects word to agree with Nth argument in the same string
- Markdown attribute `agreeWithConcept: N` **[NEW]** — inflects word to agree with Nth concept in `LocalizationOptions.concepts`
- Markdown attribute `referentConcept: N` **[NEW]** — substitutes a third-person pronoun based on Nth concept's `TermOfAddress`
- `NSMorphology` — underlying morphology descriptor (Objective-C)
- `NSInflectionRule` — underlying inflection rule (Objective-C)
- `AttributedString(localized:options:)` — existing initializer; now accepts concepts via `options`
- String Catalogs (`.xcstrings`) — Xcode tooling for managing localized strings with attribute annotations

## Code Highlights

```swift
// Dependency agreement: agree food size adjective with a concept (the food name)
var options = AttributedString.LocalizationOptions()
options.concepts = [.localizedPhrase(food.localizedName)]
let size = AttributedString(localized: "small", options: options)
// String catalog annotation: ^[small](agreeWithConcept: 1)

// Inclusive language: substitute pronouns based on preferred term of address
var options = AttributedString.LocalizationOptions()
options.concepts = [.termsOfAddress(person.preferredTermsOfAddress)]
let message = AttributedString(
    localized: "\(person.name) is on ^[their](referentConcept: 1) way.",
    options: options
)
```

## Takeaways
- `agreeWithConcept` and `agreeWithArgument` solve the long-standing problem of adjective–noun agreement when the two words are in separate UI elements or separate parts of a string, with minimal code changes.
- `TermOfAddress` and `referentConcept` enable apps to display grammatically personalized language (he/she/they pronouns) based on stored user or contact preferences — a single localized string handles all variants.
- All new Markdown attributes degrade gracefully on older OS versions, so adoption is safe without version guards.
- New locale support for European Portuguese and German expands the reach of the inflection engine.

---
_Source: WWDC23 Session 10153 page (abstract, chapter summaries, code samples, and resource links)._
