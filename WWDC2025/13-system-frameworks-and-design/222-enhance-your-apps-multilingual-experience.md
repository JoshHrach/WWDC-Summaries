# Enhance Your App's Multilingual Experience
**WWDC25 · Session 222** · [Watch](https://developer.apple.com/videos/play/wwdc2025/222/)

_Platforms:_ iOS 26, iPadOS 26, macOS Tahoe 26, watchOS 26, visionOS 26

## Overview
This session covers three major advances for multilingual apps in iOS 26: Language Discovery (a new Siri-powered mechanism for detecting a user's preferred languages and surfacing them in apps), new alternate calendar identifiers in Foundation, and significant improvements to bidirectional text handling — including Natural Selection (supporting non-contiguous text ranges) and dynamic writing-direction determination.

The session provides practical guidance on replacing the deprecated `Locale.preferredLanguages` string API with the richer `Locale.preferredLocales` API, and explains how to update custom text engines to support multi-range selection in bidirectional text.

## Key Topics

### Language Discovery
- Siri uses on-device intelligence to detect that a user types, listens, or browses in multiple languages even when the device is set to a single UI language.
- When detected, Siri proactively suggests enabling additional languages, keyboards, and content recommendations.
- **`Locale.preferredLocales`** — **[NEW]** returns an array of `Locale` objects (language + region) superseding the string-based `Locale.preferredLanguages`. Contains numbering system, localized names for languages and regions, and all BCP-47/ICU/CLDR identifier formats.
- Apps should use `preferredLocales` to prioritize content (e.g., Translate shows the user's languages at the top of its list), recommend localized content (e.g., Apple Music surfaces tracks in preferred languages), and adapt formatting (calendar, currency, date).
- `Locale.preferredLanguages` is slated for deprecation — migrate now.
- Use `locale.language.isEquivalent(to:)` or `hasCommonParent` to match a user's preferred locale against your app's available locales.

### Alternate Calendars
- **11 new `Calendar.Identifier` values** — **[NEW]** including Gujarati, Marathi, and Korean, bringing the total to 27. Available on all platforms.
- Foundation's `Calendar` and `DateFormatter` APIs automatically adapt to the selected alternate calendar.

### Bidirectional Text
- **Natural Selection** — **[NEW on iOS/iPadOS 26]** text selection in bidirectional (LTR+RTL mixed) text now follows the cursor naturally, producing multiple non-contiguous ranges rather than a single gapped range.
- **`UITextView.selectedRanges`** — **[NEW]** an array of `NSRange` values; the singular `selectedRange` property is deprecated in a future release.
- **`UITextViewDelegate.shouldChangeTextInRanges(_:replacementText:)`** — **[NEW]** updated delegate method accepting an array of ranges.
- **`editMenuForTextIn(ranges:suggestedActions:)`** — **[NEW]** multi-range version of the edit menu delegate method.
- Requires TextKit2: accessing `textView.layoutManager` reverts to TextKit1 and disables Natural Selection. Use `textView.textLayoutManager` instead.
- **Dynamic writing direction** — **[NEW]** the writing direction of a paragraph automatically changes to RTL when RTL text becomes dominant, rather than being fixed by the first character typed. Apps using standard UIKit/AppKit text views receive this for free.
- Custom text engines should refer to the Language Introspector sample code for the new writing-direction determination APIs.

## APIs & Frameworks

### Foundation
- `Locale.preferredLocales` — **[NEW]** `[Locale]` array replacing `preferredLanguages`
- `Locale` properties: `numberingSystem`, `localizedString(forLanguageCode:)`, etc.
- `Calendar.Identifier` — 11 new values **[NEW]**
- `locale.language.isEquivalent(to:)` / `locale.language.hasCommonParent(with:)` — locale matching

### UIKit
- `UITextView.selectedRanges` — **[NEW]** `[NSRange]` multi-range selection
- `UITextViewDelegate.shouldChangeTextInRanges(_:replacementText:)` — **[NEW]**
- `UITextViewDelegate.editMenuForTextIn(ranges:suggestedActions:)` — **[NEW]**
- `textView.textLayoutManager` — TextKit2 layout manager (use instead of `layoutManager`)

### SwiftUI (via related session)
- SwiftUI Rich Text Editor supports Natural Selection via `AttributedString.Index` range sets.

## Code Highlights

```swift
// Match user's preferred locales against app's available locales
let preferredLocales = Locale.preferredLocales
let availableLocales = getAvailableLocalesForTranslatingFrom()
var matchedLocales: [Locale] = []

for locale in availableLocales {
    for preferredLocale in preferredLocales {
        if locale.language.isEquivalent(to: preferredLocale.language) {
            matchedLocales.append(locale)
            break
        }
    }
}
```

```swift
// Delete text in multiple non-contiguous ranges (Natural Selection)
let ranges = textView.selectedRanges.reversed()
for range in ranges {
    textView.textStorage.deleteCharacters(in: range)
}
```

## Takeaways
- Replace `Locale.preferredLanguages` with `Locale.preferredLocales` throughout your app — it provides richer locale metadata and is future-proof.
- Use `preferredLocales` to personalize language pickers, content recommendations, and format choices rather than asking users to manually select their language.
- Update any code that reads `textView.selectedRange` to read `textView.selectedRanges` (an array) to handle bidirectional text selection correctly.
- Ensure your custom text view code uses `textView.textLayoutManager` (TextKit2) rather than `textView.layoutManager` (TextKit1) to benefit from Natural Selection and other iOS 26 text improvements.

---
_Source: WWDC25 Session 222 page (abstract, chapters, full transcript, and code samples)._
