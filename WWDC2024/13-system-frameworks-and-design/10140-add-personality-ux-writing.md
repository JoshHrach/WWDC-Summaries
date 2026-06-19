# Add Personality to Your App through UX Writing
**WWDC24 · Session 10140** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10140/)

_Platforms:_ iOS, iPadOS, macOS, watchOS, tvOS, visionOS (design/writing practice; platform-agnostic)

## Overview
This design-focused session teaches developers and designers how to use voice and tone to give an app a distinct personality through its written text—labels, error messages, onboarding copy, empty states, and notifications. The session argues that UX writing is not just about clarity but about character: the words you choose tell users who you are and build (or erode) trust.

The session introduces a practical framework for establishing voice (who you are, consistent) and tone (how you speak in a given moment, context-dependent). Through before-and-after rewrites of real UI copy, the presenters demonstrate how to avoid common writing pitfalls—jargon, passive voice, false enthusiasm, overly apologetic errors—and instead write copy that feels human, helpful, and on-brand.

The advice directly applies to String Catalogs and any localized string in an Xcode project, making it a practical companion to the localization-focused sessions.

## Key Topics
- **Voice vs. Tone** — voice is your consistent brand identity in words; tone shifts based on emotional context (success vs. error vs. warning)
- **Personality traits** — defining 2–4 adjectives that describe how your app "sounds" (e.g., warm, direct, playful, expert)
- **Error messages** — the most important UX writing surface; should explain what happened, why, and what to do next—without blaming the user
- **Empty states** — opportunity to set expectations and provide a call to action, not just say "Nothing here yet"
- **Onboarding and permissions** — explain the value exchange clearly before asking for sensitive permissions
- **Things to avoid** — exclamation marks overuse, "Oops!", fake apologies ("We're sorry but…"), jargon, marketing speak in UI
- **Inclusive language** — avoid idioms that don't translate; write for localization from the start

## APIs & Frameworks
UX writing is a design practice, not a code API. However, it directly intersects with:
- **String Catalogs** (`.xcstrings`) — centralized localization strings in Xcode 15+; where your voice and tone live in code
- **`NSLocalizedString` / `LocalizedStringKey`** — Swift/SwiftUI localized string APIs
- **SwiftUI `Text` views** — rendered output of your UX copy; support markdown for emphasis
- **Accessibility labels** — `accessibilityLabel(_:)`, `accessibilityHint(_:)` — UX writing extends to VoiceOver strings
- **Push notification payloads** — `alert.title`, `alert.body` — high-stakes writing surface with strict character limits
- **`UIAlertController` / `.alert` modifier** — title, message, and button label copy

## Code Highlights
```swift
// String Catalog usage — good UX copy in LocalizedStringKey
// Before: "Error occurred."
// After: "Couldn't save your changes. Check your connection and try again."

Text("Couldn't save your changes. Check your connection and try again.",
     comment: "Error shown when a network save fails; instructs user to retry")

// Accessibility hint — voice and tone matter here too
Image(systemName: "heart")
    .accessibilityLabel("Add to favorites")
    // Not: "Heart icon" or "Button"

// Permission request — lead with value
// Before: "Allow access to your location?"
// After (custom pre-permission screen body):
Text("See restaurants near you and get accurate delivery estimates.")
```

## Takeaways
- Establish your app's voice in a brief style guide (even a one-pager) before writing any UI copy; it keeps strings consistent across a team
- Error messages are your most important UX writing surface—tell users what happened, why, and what to do next in plain language
- Tone is contextual: be warm and encouraging in onboarding, direct and calm in error states, celebratory (but not excessive) in success moments
- Write for localization from the start: avoid idioms, keep strings short, and never concatenate strings that would break in other languages

---
_Source: WWDC24 Session 10140 page (abstract, chapter summaries, code samples, and resource links)._
