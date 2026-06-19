# Game Center Player Identifiers
**WWDC19 · Session 615** · [Watch](https://developer.apple.com/videos/play/wwdc2019/615/)

_Platforms:_ iOS 13, iPadOS 13, macOS 10.15 Catalina, tvOS 13, watchOS 6

## Overview
Game Center introduces two new scoped player identifiers to replace the deprecated static `playerID` property on `GKPlayer`. The change is privacy-driven: rather than giving every game and every development team the same global identifier for a given player, the new identifiers are scoped either to the developer's team (team player ID) or to an individual game (game player ID), preventing cross-game and cross-team tracking.

The session explains the scoping semantics for both identifier types, describes behavior when game ownership is transferred between teams, and specifies when to use each identifier type when migrating existing apps that persist player identifiers in backend systems or local storage.

## Key Topics

**Static Player ID (Deprecated)**
The old `GKPlayer.playerID` property returns the same value regardless of which game the player is playing or which team released it. This enables cross-game tracking and is now deprecated.

**Team Player ID**
- Scoped to the development team (identified by Team ID in App Store Connect)
- A player gets a different team player ID for each team whose games they play
- When a game is transferred from Team A to Team B, the player's team player ID for that game changes to the Team B value
- Use this identifier when storing per-player data in a backend shared across a team's portfolio of games

**Game Player ID**
- Scoped to an individual game, independent of which team owns it
- A player gets a different game player ID for each game they play
- Persists with the game if it is transferred between development teams
- The most privacy-preserving identifier; use when data is tied to a single game

**Non-Local Player Scoping**
- `GKLocalPlayer.teamPlayerID` and `GKLocalPlayer.gamePlayerID` are persistent
- `GKPlayer` instances for non-local players (from leaderboards, multiplayer matches, etc.) have identifiers scoped to that match/leaderboard instance — they are not guaranteed to be persistent across subsequent matches
- Use `GKPlayer.loadPlayersForIdentifiers(_:withCompletionHandler:)` with the new identifiers to load player objects

**Availability Check**
- In rare cases scoped IDs may not be available immediately after authentication
- New `GKPlayer.scopedIDsArePersistent` **[NEW]** property returns `false` in those cases; always check before persisting or using the identifiers

**Migration**
- Apps that do not read `playerID` need no changes
- Apps that use `playerID` for per-player data should migrate to `teamPlayerID` (most cases) or `gamePlayerID`
- Backend migration: convert from old static ID to scoped ID after a successful authentication

## APIs & Frameworks

**GameKit**
- `GKPlayer` (existing)
  - `var playerID: String` — **deprecated** in iOS 13; replaced by scoped identifiers
  - `var teamPlayerID: String` **[NEW]** — persistent identifier scoped to the developer team
  - `var gamePlayerID: String` **[NEW]** — persistent identifier scoped to the individual game
  - `var scopedIDsArePersistent: Bool` **[NEW]** — `false` when scoped IDs are temporarily unavailable
  - `class func loadPlayers(forIdentifiers identifiers: [String], withCompletionHandler completionHandler: (([GKPlayer]?, Error?) -> Void)?)` — updated to accept either `teamPlayerID` or `gamePlayerID` values
  - `var alias: String` — player nickname (unchanged)
  - `func loadPhoto(for size: GKPlayer.PhotoSize, withCompletionHandler completionHandler: ((UIImage?, Error?) -> Void)?)` — avatar image (unchanged)
- `GKLocalPlayer` (existing)
  - Inherits `teamPlayerID`, `gamePlayerID`, `scopedIDsArePersistent` from `GKPlayer`
  - `var authenticateHandler: ((UIViewController?, Error?) -> Void)?` — unchanged; check scoped IDs after successful authentication

## Code Highlights

Checking and using scoped identifiers after authentication:
```swift
GKLocalPlayer.local.authenticateHandler = { viewController, error in
    if let vc = viewController {
        // present vc
        return
    }
    guard error == nil else { return }

    let localPlayer = GKLocalPlayer.local
    guard localPlayer.scopedIDsArePersistent else {
        // scoped IDs not yet available; retry later
        return
    }

    let teamID = localPlayer.teamPlayerID   // use for cross-game backend
    let gameID = localPlayer.gamePlayerID   // use for single-game backend
    // migrate from old playerID if necessary
}
```

Loading players by scoped identifier:
```swift
GKPlayer.loadPlayers(forIdentifiers: [storedTeamPlayerID]) { players, error in
    // use players
}
```

## Takeaways
- Stop using `GKPlayer.playerID`; it is deprecated in iOS 13 and will eventually be removed.
- Use `teamPlayerID` for shared backend systems across a developer's game portfolio; use `gamePlayerID` for single-game data stores.
- Always check `scopedIDsArePersistent` before reading or storing scoped IDs — handle the rare case where they are temporarily unavailable.
- Non-local player scoped IDs from leaderboard or multiplayer contexts are instance-scoped and may not be stable across sessions; use `loadPlayers(forIdentifiers:)` to resolve them.

---
_Source: WWDC19 Session 615 page (abstract, chapter summaries, code samples, and resource links)._
