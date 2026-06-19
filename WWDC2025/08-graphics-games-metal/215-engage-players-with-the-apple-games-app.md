# Engage Players with the Apple Games App
**WWDC25 · Session 215** · [Watch](https://developer.apple.com/videos/play/wwdc2025/215/)

_Platforms:_ iOS 26, iPadOS 26, macOS Tahoe 26, tvOS 26

## Overview
The Apple Games app is a new first-party destination for discovering, playing, and sharing games — available system-wide on iOS, iPadOS, macOS, and tvOS. This session introduces two new GameKit features that power the social layer of the Games app — Challenges and Activities — and explains how developers can configure their games for maximum visibility and engagement in this new surface.

The session also covers the new Xcode 26 Game Center configuration workflow and the continuing role of In-App Events in the Games app's editorial and promotional surfaces.

## Key Topics

### Apple Games App Overview
- A dedicated app (and system integration) for discovering, managing, and playing games.
- Surfaces personalized recommendations, friends' activity, trending titles, and App Store editorial content.
- Games already on the App Store appear automatically; richer integration requires adopting the new GameKit features.

### Challenges (New GameKit Feature)
- **Challenges** let players create and send gameplay challenges directly to friends from within a game. **[NEW]**
- Challenge types include score-based and achievement-based goals.
- Challenges drive re-engagement: recipients are notified when a friend challenges them and again when results are available.
- Configure challenge eligibility per leaderboard and per achievement in App Store Connect.
- New **`GKChallenge`** API additions allow games to present challenge UI and respond to incoming challenges programmatically.
- Challenges appear in the Games app's Friends tab and trigger push notifications.

### Activities (New GameKit Feature)
- **Activities** are structured milestones within a game that players can track and share. **[NEW]**
- Each activity has a name, description, and optional visual assets (artwork, icon); configured in App Store Connect.
- Players complete activities; completion is reported to GameKit and surfaces in the Games app.
- Activities differ from achievements: they are designed to be repeatable and ongoing rather than one-time unlock events.
- New `GKActivity` API for reporting completion and querying status.

### Leaderboard Improvements
- **Description field** — **[NEW]** leaderboards now support a free-text description displayed in the Games app leaderboard view; helps players understand the context and scoring rules.
- Configure via App Store Connect (Leaderboard metadata section).

### Game Center Configuration in Xcode 26
- **[NEW]** Xcode 26 provides a native Game Center configuration editor integrated into the project settings.
- Configure leaderboards, achievements, challenges, activities, and their metadata directly in Xcode without leaving the IDE.
- Changes sync with App Store Connect; no need to context-switch to a browser.

### In-App Events
- In-App Events (launched WWDC21) surface in the Games app's events and editorial surfaces.
- Keep events up to date with current gameplay offers, seasonal content, and tournaments.
- Events shown to players who have the game installed and to potential new players in discovery surfaces.

## APIs & Frameworks

### GameKit
- `GKChallenge` — existing class; new methods for challenge creation/presentation. **[NEW challenge authoring APIs]**
- `GKActivity` — **[NEW]** new class for declaring and reporting activities.
- `GKLeaderboard` — existing class; leaderboard description field now available. **[NEW description field]**
- App Store Connect API — configure challenges, activities, and leaderboard descriptions; Xcode 26 editor wraps this.
- Game Center configuration editor in Xcode 26. **[NEW]**

## Code Highlights
This session was demo- and UI-focused. Code details for `GKActivity` and `GKChallenge` authoring APIs are in the GameKit documentation updates for iOS 26.

## Takeaways
- Adopt Challenges to drive re-engagement: score-based and achievement-based challenge flows are handled almost entirely by GameKit; your game only needs to provide leaderboard/achievement context.
- Configure Activities for ongoing, repeatable milestones — they are the primary social sharing mechanism in the Games app beyond leaderboards.
- Fill in the new leaderboard Description field in App Store Connect so players understand scoring without leaving the Games app.
- Use the Xcode 26 Game Center configuration editor to streamline the setup of all these features without switching to a browser.

---
_Source: WWDC25 Session 215 page (abstract, chapter summaries, and full transcript)._
