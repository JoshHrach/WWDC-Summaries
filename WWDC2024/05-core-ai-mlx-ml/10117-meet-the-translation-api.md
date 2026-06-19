# Meet the Translation API
**WWDC24 · Session 10117** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10117/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia 15

## Overview
Apple's new Translation framework brings on-device machine translation directly into third-party apps. The same ML models that power the Translate app, system-wide translation, and camera translation are now available to developers through two APIs: a one-line SwiftUI overlay for quick single-string translation, and a flexible `TranslationSession` API for batch translation and in-app UI integration.

Translation runs entirely on-device; language models are shared system-wide and downloaded on demand. Hindi is newly supported in iOS/macOS 2024, bringing the total supported languages in line with the Translate app.

## Key Topics

**Simple Overlay Translation (translationPresentation)**
- `.translationPresentation(isPresented:text:)` — SwiftUI view modifier; one line of code shows the system translation sheet over the app
- Identical to the system-wide translation feature accessible via text selection throughout iOS
- The user can change the target language in the sheet
- Attach the modifier to the content being translated (not to a button) so the popover points to the right element on iPad and Mac

**Flexible Translation (TranslationSession)**
- `TranslationSession` — the core class for programmatic translation; never instantiated directly
- Created and provided via the `.translationTask(configuration:)` view modifier; the closure runs whenever the configuration becomes non-nil or changes
- `TranslationSession.Configuration` — controls source/target language; `nil` source = automatic language detection; `nil` target = best language for the user; call `.invalidate()` to re-trigger translation of new content with the same session
- `TranslationSession.translate(_:)` — translate a single string; returns a `TranslationSession.Response`
- `TranslationSession.translations(from:)` — batch translate an array of `TranslationSession.Request`; returns all results at once in original order
- `TranslationSession.translate(batch:)` — batch translate as an async sequence; streams results as they complete; use `clientIdentifier` on each request to match responses
- Prefer batch APIs when translating multiple strings of the same language — more efficient than multiple single-string calls
- Keep all source text in a single batch to the same language; mixing languages in one batch produces poor results
- Each `TranslationSession` is anchored to a view — don't store it outside that view's lifetime

**Language Detection and Language Availability**
- `LanguageAvailability` — new class to query supported languages and translation pair status
- `LanguageAvailability.supportedLanguages` — list of supported `Locale.Language` values; use these when specifying source/target languages
- `LanguageAvailability.status(from:to:)` — returns `.supported`, `.installed`, or `.unsupported` for a language pair
- `LanguageAvailability.status(for:to:)` — variant that accepts source text instead of a source language
- Same-language pairs and same-variant pairs always return `.unsupported`
- `NLLanguageRecognizer` — use when you need to identify text language programmatically; call `processString(_:)` then `dominantLanguage`, then convert to `Locale.Language`

**Language Download Flow**
- Models are downloaded on demand; the framework shows download progress UI automatically
- Downloads continue in the background even if the user leaves the app
- `TranslationSession.prepareTranslation()` — proactively trigger the download prompt without translating; useful for offline scenarios

**Best Practices**
- These APIs do not work in the Simulator — use a real device for development
- Attach `.translationPresentation` and `.translationTask` to the content view, not a button
- Keep all strings in a single batch from the same source language; use separate batch calls per language within the same session
- Use `nil` source language to let the framework auto-detect and handle multi-language content
- A new SF Symbol is available for translation-related UI affordances

## APIs & Frameworks

**Translation** **[NEW]**
- `TranslationSession` **[NEW]** — performs on-device machine translation
- `TranslationSession.Configuration` **[NEW]** — specifies source and target `Locale.Language`; `nil` = automatic
- `TranslationSession.Configuration.invalidate()` **[NEW]** — marks configuration changed to re-trigger `.translationTask`
- `TranslationSession.Request` **[NEW]** — wraps a source string with an optional `clientIdentifier`
- `TranslationSession.Response` **[NEW]** — contains translated string and `clientIdentifier`
- `TranslationSession.translate(_:)` **[NEW]** — translate a single `String`; async, throws
- `TranslationSession.translations(from:)` **[NEW]** — batch translate; returns ordered array of responses
- `TranslationSession.translate(batch:)` **[NEW]** — streaming batch translate; async sequence of responses
- `TranslationSession.prepareTranslation()` **[NEW]** — trigger download prompt without translating
- `LanguageAvailability` **[NEW]** — query supported languages and translation pair status
- `LanguageAvailability.supportedLanguages` **[NEW]** — `[Locale.Language]` of all supported languages
- `LanguageAvailability.status(from:to:)` **[NEW]** — check if a language pair is `.supported`, `.installed`, or `.unsupported`
- `LanguageAvailability.status(for:to:)` **[NEW]** — check pair status using source text instead of source language

**SwiftUI (Translation extensions)**
- `.translationPresentation(isPresented:text:)` **[NEW]** — show system translation overlay sheet
- `.translationTask(configuration:)` **[NEW]** — view modifier that provides a `TranslationSession` when configuration is non-nil

**Natural Language**
- `NLLanguageRecognizer` — identify language of arbitrary text; `processString(_:)` + `dominantLanguage`

## Code Highlights

Simple overlay (one line):
```swift
@State var showsTranslation = false

someTextView
    .translationPresentation(isPresented: $showsTranslation, text: review.body)
```

Flexible in-app batch translation:
```swift
@State var configuration: TranslationSession.Configuration?

List { /* reviews */ }
.translationTask(configuration) { session in
    let requests = reviews.map { TranslationSession.Request(sourceText: $0.body) }
    for try await response in session.translate(batch: requests) {
        updateUI(with: response)
    }
}

// Trigger translation:
func translateButtonTapped() {
    if configuration == nil {
        configuration = TranslationSession.Configuration() // auto language detection
    } else {
        configuration?.invalidate() // re-translate new content
    }
}
```

Check language pair support:
```swift
let availability = LanguageAvailability()
let status = await availability.status(from: Locale.Language(identifier: "ja"),
                                       to: Locale.Language(identifier: "en"))
// .supported, .installed, or .unsupported
```

## Takeaways
- Use `.translationPresentation` for a zero-effort single-translation flow; adopt `TranslationSession` when you need translations displayed inline or in bulk.
- Always pass `nil` as the source language unless you are certain of the content's language — auto-detection handles mixed-language content gracefully.
- Keep all text in a batch from the same language; make separate `translate(batch:)` calls per language using a single session.
- Call `TranslationSession.prepareTranslation()` ahead of anticipated offline usage to trigger model downloads before they are needed.

---
_Source: WWDC24 Session 10117 page (abstract, chapter list, full transcript, and resource links)._
