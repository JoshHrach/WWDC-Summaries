# Designing for a Global Audience
**WWDC17 · Session 819** · [Watch](https://developer.apple.com/videos/play/wwdc2017/819/)

_Platforms:_ iOS, macOS, watchOS, tvOS (design/localization principles; platform-agnostic)

## Overview
This session makes the case that designing for a global audience is not an all-or-nothing exercise — smart, targeted changes can significantly improve usefulness and appeal to international users even before a full localization is undertaken. The App Store reaches every country enabled in App Store Connect, so every app already has a potential global audience.

The session covers three primary design considerations for international audiences: language choices (both translated copy and the risks of informal language and idioms), symbology (how icons and gestures carry different meanings across cultures), and associations (how imagery, animals, and colors carry culturally specific connotations). It also provides practical research strategies for teams that cannot do international user research in person.

The Apple Maps post office icon set is used as a concrete example of the spectrum from hyper-local iconography (Japan Post logo used only in Japan) to globally recognizable symbols (a simple letter outline used everywhere else), illustrating the trade-off between local resonance and universal clarity.

## Key Topics

- **Strategic planning for localization** — identify target countries and languages on the roadmap; review App Store Connect app analytics for surprising market data before prioritizing; batch translation and artwork work together to save time.
- **Language — word choice across markets** — when English words are interchangeable ("picture" vs. "photo"), prefer the one with closer cognates in other languages ("photo" wins internationally); prioritize translating headings, key instructions, important terms, and error messages.
- **Informal language and idioms** — slang, figures of speech, and non-literal phrases add character but can confuse or offend global users; check for unintended meanings and provide contextual clues.
- **Gestures and non-verbal communication** — counting gestures vary widely by culture (finger ordering for 1-2-3 differs between Western and East Asian conventions); test gestural UI against target markets.
- **Iconography: local to global spectrum** — the post office example illustrates three levels: market-specific logos (Japan Post), regional symbols (postal horn in 30+ European/Middle Eastern countries), and universal glyphs (envelope outline everywhere else); deeper localization creates stronger resonance but requires more assets.
- **Depicting people in icons** — detailed human icons may exclude users who don't identify with them; simple silhouettes (available in UIKit) are more universally neutral and require no per-market customization.
- **Cultural associations** — symbols carry culturally specific meanings (owls = wisdom in English-speaking cultures; bad luck or death in parts of the Arabic-speaking world); research associations in target markets before using imagery.
- **Research strategies without international travel** — formal or informal focus groups segmented by language/cultural background; personal and professional contacts; language-learning and translation apps as language guides; web/image searches for symbol usage; intercultural communication books; libraries.

## APIs & Frameworks

This is a design and localization principles session with no new API introductions. Relevant Apple platform features referenced:

- **App Store Connect Analytics** — check downloads and engagement by country to identify unexpected international markets
- **App Store Connect / iTunes Connect territory selection** — controls which countries can purchase the app
- **NSLocalizedString / String Catalogs** — Foundation localization infrastructure for translated copy
- **NSLocale** — locale-aware formatting for dates, numbers, and currencies
- **UIKit system glyphs** — built-in person silhouette and other neutral glyphs usable as globally safe iconography without custom artwork
- **RTL (Right-to-Left) layout** — `UIView.semanticContentAttribute`, auto-mirroring for Arabic/Hebrew
- **Dynamic Type** — `UIFont.preferredFont(forTextStyle:)` — important for languages with longer translated strings that need flexible layout
- **Xcode Localization Catalog** — `.xcloc` export/import format for sending strings to translators

## Code Highlights

No code samples presented. The session is a design and UX research lecture.

## Takeaways

- Analyze App Store Connect analytics before localizing — markets may already be downloading your app in surprising countries, revealing where localization effort will have the most impact.
- When choosing between synonymous English words ("picture" vs. "photo"), prefer the word with closer cognates across other target languages to ease translation consistency.
- Avoid idioms, slang, and non-literal figures of speech unless you can verify they don't offend in target markets and you provide enough context for users who may not recognize them.
- Test icons and symbols in target markets; a globally recognizable neutral glyph (UIKit person silhouette, envelope outline) is often safer than a locally resonant but regionally confusing custom symbol.

---
_Source: WWDC17 Session 819 page (abstract, transcript, and resource links)._
