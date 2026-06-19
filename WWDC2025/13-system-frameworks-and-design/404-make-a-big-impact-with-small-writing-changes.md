# Make a big impact with small writing changes
**WWDC25 · Session 404** · [Watch](https://developer.apple.com/videos/play/wwdc2025/404/)

_Platforms:_ All Apple platforms

## Overview
Good writing inside an app is an invisible feature: when it works, users simply understand and act. When it fails, confusion and support burden follow. This session distills Apple's content design principles into a set of actionable micro-edits — small wording changes that measurably reduce cognitive load and increase task completion without requiring a design overhaul.

The session is aimed at engineers and designers who write their own UI copy, not professional writers. It demonstrates that even minor rewrites — changing passive constructions to active ones, eliminating filler words, and reordering information to lead with what matters — make a tangible difference in how quickly users understand what to do next.

The Human Interface Guidelines Writing chapter and the Apple Style Guide are the two canonical references cited throughout; both are freely available on developer.apple.com and should be treated as living standards that evolve with each platform release.

## Key Topics

### Lead with the Action
UI strings should front-load the verb or outcome, not the condition. Instead of "If you would like to continue, tap OK," write "Tap OK to continue." Users scan rather than read; putting the call-to-action first matches scanning behavior.

### Cut Filler Words
Words like "simply," "easily," "just," and "please" add length without information. Every filler word delays comprehension. The session provides before/after pairs of alert bodies, button labels, and onboarding strings to illustrate the reduction.

### Avoid Passive Voice in Errors
Error messages in passive voice obscure who is responsible and what to do. "Your request could not be completed" gives the user nowhere to go. "Something went wrong — try again" is shorter and actionable.

### Match Platform Conventions
Each platform has terminology norms: macOS says "Quit," iOS says nothing (the concept doesn't apply the same way); macOS uses "window," not "screen." Using cross-platform generic terms ("the app," "the screen") produces copy that feels generic on all platforms and native on none.

### Write for Accessibility
Screen reader users hear every word. Redundant phrases like "button" in a button label ("Submit button") cause VoiceOver to say the type twice. Acronyms should be spelled out on first use. Date and time formats should be unambiguous across locales.

### Test with Real Users
Even a two-person hallway test surfaces misread labels. The session recommends a "5-second test" for any critical string: show the string for five seconds and ask users what they think they should do next.

## APIs & Frameworks

- **Human Interface Guidelines: Writing** — canonical style and tone guidance for all Apple platforms
- **Apple Style Guide** — terminology, capitalization, and punctuation reference
- _(No code-level APIs; this is a design/content session)_

## Code Highlights

_This session contains no code samples. The guidance applies to string literals, localized string files, and asset catalog display names across all Apple platforms._

### Before / After Examples (from the session)

| Before | After |
|--------|--------|
| "Your account could not be verified at this time." | "Couldn't verify your account. Try again." |
| "Simply tap the button below to get started." | "Tap Get Started." |
| "If you would like to enable notifications, you can do so in Settings." | "To get notifications, go to Settings." |

## Takeaways

- Lead every UI string with the action or outcome; users scan, not read.
- Delete filler words ("simply," "just," "please") without replacement — they add zero information.
- Use active voice in errors and pair every problem statement with a resolution step.
- Match platform-specific terminology; generic cross-platform language sounds foreign everywhere.

---
_Source: WWDC25 Session 404 page (abstract, chapter summaries, code samples, and resource links)._
