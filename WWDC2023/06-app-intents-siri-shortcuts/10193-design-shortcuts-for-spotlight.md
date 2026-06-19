# Design Shortcuts for Spotlight
**WWDC23 · Session 10193** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10193/)

_Platforms:_ iOS 17, iPadOS 17

## Overview
In iOS 17, App Shortcuts surface directly inside Spotlight's top hit, giving users immediate access to key app actions without opening the app. This session by Apple HI designer Cameron covers the full design process for creating effective App Shortcuts for Spotlight: ideation principles, visual appearance (icons, colors, titles), behavior selection (app launch vs. Live Activity vs. snippet), and discoverability through phrase synonyms.

The session is the design companion to "Spotlight your app with App Shortcuts" (10102) and "Explore enhancements to App Intents" (10103). It teaches developers how to think about what shortcuts to create and how to present them in a way that is recognizable, personal, and instantly useful — not just a reflection of the app's tab bar.

## Key Topics

### Principle 1: Focus on the Essentials
Identify the handful of habitual, essential actions or content items people reach for most frequently. Do not expose every feature. It is fine to have just one or two shortcuts. Avoid complex multi-step workflows that are better done inside the app.

### Principle 2: Prefer Actionable Shortcuts
Go beyond mirroring the app's tab bar navigation. Design shortcuts that perform the app's primary purpose directly — e.g., placing a call to a favorite contact for Phone, rather than opening the Recents tab.

### Principle 3: Predictable and Personal Shortcuts
Surface content and actions that are personalized to the user's history inside the app (recently listened podcasts, pinned notes, recent contacts). Never suggest new or trending content the user has not engaged with. Reflect the ordering logic from inside the app (e.g., reverse chronology for notes, pinned items first) to build muscle memory.

### Visual Design: Icons
- Every shortcut is either an **action** (verb, rendered as SF Symbol inside a circle) or an **entity** (noun, can use a custom icon for better recognizability).
- Entity icons should match the shapes used inside the app (circle for Reminders lists, square for Photos albums).
- Carry visual details from the app's UI into the icon (e.g., a heart badge on the Favorites album icon).
- Keep icon shapes as recognizable as possible at small sizes.

### Visual Design: Titles and Subtitles
- Titles should be concise and pithy; never include the app name (it is already displayed).
- Titles may wrap to a second line but should be kept short to avoid truncation.
- Entities can provide a subtitle with additional detail (e.g., Photos showing the count of favorited photos).

### Visual Design: Color
- By default, shortcuts use a neutral appearance suitable for multicolor or neutral app icons.
- Apps with a strong brand color (e.g., Notes yellow, Maps green) can infuse that color into Spotlight top hits and Shortcuts app platters.
- Derive the complementary color by conceptually peeling the graphics off the app icon to isolate the background gradient or solid color.
- For apps with dark complementing colors, provide a secondary tint color to create a two-tone symbol appearance that maintains brand identity.

### Shortcut Behavior
Three options for what happens when a shortcut is triggered:
1. **App Launch** — opens the app to a specific screen. Simple, reliable, and often the right choice.
2. **Live Activity** — starts a Live Activity or background audio session. Best for simple actions requiring no further in-app attention (e.g., starting a timer, placing a call).
3. **Snippet** — shows a compact UI to ask a question or display a small piece of information. After the snippet, optionally launch a Live Activity or open the app. Use only for simple tasks; prefer app launch for complex flows.

### Discoverability: Phrase Synonyms
- Each shortcut has a **phrase** (natural language description including the app name; also the Siri invocation phrase).
- Provide multiple **synonyms** to catch different ways users might search (e.g., "File Scanner" as a synonym for "Scan Document").
- If the app is known by multiple names (e.g., Music Recognition app also known as "Shazam"), provide **app name synonyms** to cover alternative searches.

## APIs & Frameworks

### App Intents / App Shortcuts **[NEW in iOS 16, expanded in iOS 17]**
- `AppShortcut` — defines an App Shortcut with a phrase and action **[NEW expanded surface in iOS 17]**
- `AppShortcutsProvider` — protocol for providing an app's shortcut list **[NEW]**
- `ShortcutTile` — Spotlight top hit tile representation (design concept; surfaced via AppShortcutsProvider)
- `AppShortcut.phrase` — the primary Siri/Spotlight phrase, must include app name **[NEW]**
- `AppShortcut.phraseAlternatives` / phrase synonyms — additional search terms **[NEW]**
- App name synonyms — alternative app name strings for Spotlight search **[NEW]**
- **Action** (verb shortcut) — backed by `AppIntent` conformance
- **Entity** shortcut — backed by `AppEntity` conformance with custom icon support
- Entity subtitle — custom string shown in Spotlight result detail **[NEW]**
- App Shortcut icon (SF Symbol inside circle for actions; custom image for entities)
- App Shortcut color / complementary color tint — brand color infusion **[NEW]**
- Secondary tint color — two-tone symbol appearance for dark brand colors **[NEW]**
- Live Activity behavior — shortcut triggers `ActivityKit` session start
- Snippet behavior — shortcut shows compact SwiftUI view inline in Spotlight
- App launch behavior — shortcut opens app to a specific `Scene` or deep link URL

### Resources
- [Human Interface Guidelines: App Shortcuts](https://developer.apple.com/design/human-interface-guidelines/app-shortcuts)
- [App Shortcuts documentation](https://developer.apple.com/documentation/AppIntents/app-shortcuts)

## Code Highlights
No code samples were shown in this design-focused session. See "Spotlight your app with App Shortcuts" (10102) and "Explore enhancements to App Intents" (10103) for implementation.

## Takeaways
- Limit shortcuts to a handful of habitual, high-value actions or content items — do not mirror the entire app nav.
- Match shortcut icon shapes and visual details to what already exists inside the app to maximize immediate recognizability.
- Choose the simplest behavior that achieves the goal: app launch is often best; Live Activity for fire-and-forget actions; snippets only for truly simple queries.
- Provide phrase synonyms (and app name synonyms if applicable) to maximize Spotlight discoverability across different search terms.

---
_Source: WWDC23 Session 10193 page (abstract, chapter summaries, code samples, and resource links)._
