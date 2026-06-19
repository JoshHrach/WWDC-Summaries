# Tap into Game Center: Dashboard, Access Point, and Profile
**WWDC20 · Session 10618** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10618/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, tvOS 14

## Overview
Game Center received a comprehensive visual and functional refresh in iOS 14. The centerpiece is a new in-game dashboard—a single, unified destination giving players access to leaderboards, achievements, challenges, and the all-new player profile without leaving the game. Two new surface areas accompany it: the Access Point (a persistent avatar button that can appear in any screen corner and display highlights like achievement count or leaderboard rank), and expanded player cards visible from leaderboards, friend lists, and even the App Store.

Authentication remains the prerequisite for all Game Center functionality. Setting `GKLocalPlayer.local.authenticateHandler` early in the app lifecycle starts the flow automatically. The handler is called on sign-in, sign-out, and account switches (particularly relevant on tvOS with its multi-user support). New in iOS 14, `isPersonalizedCommunicationRestricted` joins the existing `isUnderage` and `isMultiplayerGamingRestricted` flags, helping games honor parental controls over voice and messaging.

The Challenges feature is now opt-in via App Store Connect rather than always-on. Games that want to keep the challenges section visible simply toggle a checkbox in App Store Connect; otherwise the section is hidden from the dashboard by default.

## Key Topics
- **In-game Dashboard** — single UI for leaderboards, achievements, challenges, and player profile via `GKGameCenterViewController(state:)`
- **GKGameCenterViewControllerState** — `.dashboard`, `.leaderboards`, `.achievements`, `.challenges`, `.localPlayerProfile` **[NEW states]**
- **Deep-linking to a leaderboard** — `GKGameCenterViewController(leaderboardID:playerScope:timeScope:)` still works
- **Access Point** — `GKAccessPoint.shared`; configure location, highlights, activate with `isActive`; observe `isPresentingGameCenter` and `frameInScreenCoordinates` **[NEW]**
- **Player types** — `GKLocalPlayer` (signed-in user, persistent ID) vs. `GKPlayer` (others, per-session scoped ID)
- **Authentication flow** — set `authenticateHandler` early; handle VC-present, error, and success cases
- **Player restrictions** — `isUnderage`, `isMultiplayerGamingRestricted`, `isPersonalizedCommunicationRestricted` **[NEW]**
- **Player profiles and privacy** — three privacy levels (everyone / friends-only / no one) handled automatically
- **tvOS multi-user support** — new User Management capability; `GKLocalPlayer` changes as accounts switch
- **App Store integration** — friend activity and player cards surfaced in Arcade/Games tabs and game product pages

## APIs & Frameworks

**GameKit**

_Dashboard & View Controllers_
- `GKGameCenterViewController` — presents Game Center UI; `gameCenterDelegate: GKGameCenterControllerDelegate`
- `GKGameCenterViewController.init(state:)` — initialize to a specific dashboard section **[NEW state values]**
- `GKGameCenterViewControllerState` enum **[EXTENDED]**:
  - `.default`, `.leaderboards`, `.achievements`, `.challenges` — existing
  - `.localPlayerProfile` **[NEW]** — shows the signed-in player's full profile
  - `.dashboard` **[NEW]** — shows the top-level in-game dashboard
- `GKGameCenterViewController.init(leaderboardID:playerScope:timeScope:)` — deep-link to specific leaderboard

_Access Point_ **[ALL NEW]**
- `GKAccessPoint` — singleton managing the in-game access point button
- `GKAccessPoint.shared` — shared singleton
- `GKAccessPoint.shared.location` — `GKAccessPoint.Location`; `.topLeading`, `.topTrailing`, `.bottomLeading`, `.bottomTrailing`
- `GKAccessPoint.shared.showHighlights` — Bool; display achievement count / leaderboard rank callouts
- `GKAccessPoint.shared.isActive` — Bool; show or hide the access point
- `GKAccessPoint.shared.isPresentingGameCenter` — observable Bool; true while dashboard is presented
- `GKAccessPoint.shared.frameInScreenCoordinates` — observable `CGRect`; screen-coordinate frame for layout adjustment
- `GKAccessPoint.shared.triggerAccessPoint(completionHandler:)` — programmatically open the dashboard (for controller / Apple TV focus)

_Players & Authentication_
- `GKLocalPlayer` — represents the signed-in user
- `GKLocalPlayer.local` — shared instance
- `GKLocalPlayer.local.authenticateHandler` — closure; set early; called on sign-in, sign-out, account switch
- `GKLocalPlayer.local.isUnderage` — Bool; restrict explicit content
- `GKLocalPlayer.local.isMultiplayerGamingRestricted` — Bool; disable multiplayer
- `GKLocalPlayer.local.isPersonalizedCommunicationRestricted` **[NEW]** — Bool; disable in-game voice/messaging
- `GKPlayer` — represents other players; scoped IDs per game instantiation
- `GKPlayer.scopedIDsArePersistent()` — persisted for local player, session-scoped for others

## Code Highlights

Present the top-level dashboard:
```swift
let vc = GKGameCenterViewController(state: .dashboard)
vc.gameCenterDelegate = self
present(vc, animated: true, completion: nil)
```

Deep-link to a specific leaderboard:
```swift
let vc = GKGameCenterViewController(
    leaderboardID: "grp.xyz.laketahoe",
    playerScope: .global,
    timeScope: .allTime)
vc.gameCenterDelegate = self
present(vc, animated: true, completion: nil)
```

Configure and activate the Access Point:
```swift
GKAccessPoint.shared.location = .topLeading
GKAccessPoint.shared.showHighlights = true
GKAccessPoint.shared.isActive = true
```

Observe the dashboard presentation state:
```swift
let observation = GKAccessPoint.shared.observe(\.isPresentingGameCenter) { [weak self] _, _ in
    self?.paused = GKAccessPoint.shared.isPresentingGameCenter
}
```

Check player restrictions including the new communication flag:
```swift
GKLocalPlayer.local.authenticateHandler = { viewController, error in
    guard viewController == nil, error == nil else { return }
    if GKLocalPlayer.local.isUnderage { /* hide explicit content */ }
    if GKLocalPlayer.local.isMultiplayerGamingRestricted { /* disable multiplayer */ }
    if GKLocalPlayer.local.isPersonalizedCommunicationRestricted { /* disable voice/chat */ }
}
```

## Takeaways
- The new `GKAccessPoint` API provides a persistent, corner-anchored gateway into Game Center; use `isPresentingGameCenter` and `frameInScreenCoordinates` to adapt your game's UI dynamically.
- `GKGameCenterViewController(state: .dashboard)` is now the recommended entry point for Game Center, replacing per-feature VC instantiation; individual sections remain accessible via specific states.
- The new `isPersonalizedCommunicationRestricted` property on `GKLocalPlayer` must be checked and honored before enabling any in-game voice or messaging feature.
- The Challenges feature is now off by default in the dashboard and must be opted in via App Store Connect.

---
_Source: WWDC20 Session 10618 page (abstract, chapter summaries, code samples, and resource links)._
