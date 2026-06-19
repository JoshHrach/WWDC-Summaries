# Design for Game Center
**WWDC20 · Session 10145** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10145/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, tvOS 14

## Overview
This session from the Game Center design team walks through the full redesign of the Game Center user experience in 2020, covering every touchpoint a player interacts with: the Access Point, Dashboard, Profile, Achievements, Leaderboards, and Multiplayer Lobby. The session is primarily a design and HIG companion session rather than an API reference — it explains the visual specifications, layout rules, safe areas, artwork requirements, and terminology guidelines that developers must follow to create a polished, on-brand Game Center integration.

The 2020 Game Center redesign is significant: the entire UI is now surfaced as a transparent overlay within the game itself (no longer a separate app), achievements have been redesigned as collectible cards, leaderboards are friend-focused, and Game Center data (friends' avatars, leaderboard positions) is now visible directly on App Store game product pages.

## Key Topics

**Access Point**
The Access Point is the new, standardized way for players to open Game Center from within a game. It is rendered as the player's avatar (optionally with "highlights" such as achievement progress or leaderboard rank). It can be placed in any corner; top-left is recommended when possible. It should appear on the main menu and be hidden during active gameplay. Safe-area specifications are provided for each device class (e.g., iPhone 11 Pro portrait: 62×335 pt; landscape: 62×280 pt).

**Dashboard and Profile**
Tapping the Access Point reveals the Game Center dashboard, rendered as a transparent layer over the game. The dashboard surfaces Profile, Achievements, Leaderboards, and Multiplayer. On tvOS, dashboard artwork can be optionally provided (923×150 px at 1x / 1846×300 px at 2x, PNG or TIFF with transparency support). The profile page shows friends, friend suggestions, and lifetime achievement history. Games can link directly to the Profile page using a provided icon with the exact label "Game Center Profile."

**Achievements — Collectible Card Design**
Achievements are redesigned as collectible cards grouped into Completed and Locked sets. Three types: Standard (immediate unlock), Progressive (shows progress), and Hidden (details revealed only on unlock). Achievement images appear in the card's top portion when earned — they should be eye-catching, unique per achievement, opaque, centered (the system crops to a circle), and free of embedded text. Specs: JPEG/TIFF/PNG, 512×512 at 1x / 1024×1024 at 2x, 72+ DPI, sRGB. Titles and descriptions should be under 30 characters to avoid truncation across devices. Games support up to 100 achievements; be selective and leave room for additions in future releases. In-game achievement notifications should fire at the moment of completion.

**Leaderboards**
Each leaderboard requires unique, recognizable artwork that differentiates it from others in the set. Specs: JPEG/TIFF/PNG, 512×512 at 1x / 1024×1024 at 2x, 72+ DPI, sRGB. tvOS leaderboards use a 16:9 aspect ratio with up to three-layer parallax images (PNG, 659×371 at 1x / 1318×742 at 2x); avoid placing primary content near the edges due to scaling. Leaderboard titles should be under 30 characters, initial caps. The new leaderboard design emphasizes the friend group rather than a global leaderboard. Games can optionally embed leaderboard data directly in their UI (e.g., before a level starts or on a results screen).

**Multiplayer Lobby**
The redesigned lobby opens as an overlay with an Add button to pick players (nearby, friends, recent, contacts). Games should pause distracting animations when the lobby is open. Multiplayer can also be implemented without using the Game Center lobby UI at all.

**App Store Integration**
Game Center now integrates with the App Store: friends' avatars appear on app icons, players' achievements and friends are visible on product pages, and friends' activity is surfaced in the Games tab.

## APIs & Frameworks

### GameKit
- Access Point — in-game UI element displaying player's Game Center avatar (optionally with highlights)
- `GKAccessPoint` — controls position, visibility, and highlights display
- `GKAchievement` — player achievement progress model
- `GKAchievementDescription` — achievement metadata (title, description, image)
- `GKLeaderboard` — leaderboard model for fetching and displaying scores
- `GKLeaderboardEntry` — a player's score entry on a leaderboard
- `GKMatchmaker` — handles real-time multiplayer matchmaking
- `GKTurnBasedMatchmaker` — handles turn-based multiplayer matchmaking
- `GKMatchmakerViewController` — the multiplayer lobby UI

### Human Interface Guidelines (normative references)
- [HIG: Game Center](https://developer.apple.com/design/human-interface-guidelines/game-center) — canonical reference for all safe-area specs, artwork sizes, and terminology rules
- Access Point safe areas per device class (see HIG for complete table)
- tvOS focusable layered image design guidelines

### Terminology Rules
- "Game Center" — never localize this term
- "Game Center Profile" — correct label for profile deep-link (not "profile" or "account" alone)
- "achievements" — correct term (not "trophies" or "awards")
- "leaderboards" — correct term (not "rankings" or "scores")

## Code Highlights

No code samples were provided in this session. The companion technical sessions are "Tap into Game Center: Dashboard, Access Point, and Profile" (10618) and "Tap into Game Center: Leaderboards, Achievements, and Multiplayer" (10619).

## Takeaways
- Place the Access Point on the game's main menu; hide it during active gameplay; always leave the correct safe area (see HIG for per-device specs) — the access point expands to its maximum size when showing highlights.
- Achievement images should be unique per achievement, opaque, centered on a circle crop, and free of text — they are the primary reward signal for players and directly impact engagement.
- On tvOS, leaderboard images must use the 16:9 multi-layer parallax format and avoid placing primary content near edges since parallax scaling can clip them.
- Game Center is now visible directly on App Store product pages (friends' avatars, achievements, activity) — integrating with Game Center has a direct conversion benefit even before someone downloads the game.

---
_Source: WWDC20 Session 10145 page (abstract, transcript, and resource links)._
