# What's New in Game Center: Widgets, Friends, and Multiplayer Improvements
**WWDC21 · Session 10066** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10066/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, tvOS 15

## Overview
This session covers all major Game Center improvements for 2021, organized into three areas: game discoverability, Friends API, and multiplayer improvements. Game Center's social network features now extend further into the App Store and home screen via new widgets, requiring no code changes from developers.

The Friends API (introduced in spring 2021) gives games privacy-friendly access to a player's mutual Game Center friends list, enabling in-game social features like progression maps and friend-specific leaderboards. Players grant per-app permission, and the authorization status syncs across devices.

Multiplayer receives a completely redesigned invite flow with a Suggestions Shelf (nearby players, Game Center groups, Message groups), individual player slot management (remove uninvited players, add more after initial send), and a new Fast Start API that lets gameplay begin with the minimum player count while remaining players join progressively.

## Key Topics

### Game Discoverability
- "Friends Are Playing" section on App Store product pages: shows which Game Center friends play a game — automatic for all Game Center-enabled games
- "Recently Played" in App Store: automatic for all Game Center-enabled games
- New "Friends Are Playing" widget **[NEW]**: home screen widget showing what friends are playing
- New "Continue Playing" widget **[NEW]**: resurfaces recently played games for quick re-entry
- Both widgets support larger iPad format
- No code adoption required for any discoverability features

### Friends API
- Privacy-friendly access to a player's bidirectional Game Center friends list **[NEW]**
- Only returns friends who have also granted access to the same game
- System permission prompt shown on first use; authorization syncs across all player's devices
- `NSGKFriendListUsageDescription` key required in `Info.plist`
- `GKLocalPlayer.local.presentFriendRequestCreatorFromViewController(using:)` **[NEW]**: in-game friend request UI presented as a message sheet
- `GKLocalPlayer.local.loadFriendsAuthorizationStatus(completionHandler:)` **[NEW]**: check permission status
- `GKFriendsAuthorizationStatus` **[NEW]**: `.notDetermined`, `.denied`, `.restricted`, `.authorized`
- `GKLocalPlayer.local.loadFriends()` **[NEW]**: Swift async/await; returns `[GKPlayer]` of mutual friends
- If state is `.denied` or `.restricted`, delete any previously collected friend data

### Multiplayer Improvements

**Invite Flow and Suggestions Shelf**
- New Suggestions Shelf **[NEW]**: nearby players, Game Center groups, Message groups shown at top of matchmaker
- Game Center groups **[NEW]**: automatically created from players recently played with in real-time or turn-based matches; one tap selects entire group
- Message groups: one tap selects existing iMessage group; invitation sent via that group chat thread
- Individual player slot removal **[NEW]**: remove uninvited players individually (X button) as long as they haven't accepted; add more players after initial invitations sent
- Controller support for all Game Center UI (Home button on controller presents/dismisses dashboard)

**Fast Start API**
- `GKMatchmakerViewController.canStartWithMinimumPlayers = true` **[NEW]**: allow game to start once `GKMatchRequest.minPlayers` have connected
- Game initiator can tap "Start Game" once minimum players are ready; doesn't have to wait for all players
- Background matchmaking continues after Fast Start: remaining invited players and autoMatched slots connect progressively
- `GKMatch.delegate` receives `match(_:player:didChange:)` with `.connected` state as additional players join
- Invitee side: unchanged — present `GKMatchmakerViewController(invite:)` as before

## APIs & Frameworks

- `GameKit` framework
- `GKLocalPlayer`
- `GKLocalPlayer.local`
- `GKLocalPlayer.presentFriendRequestCreatorFromViewController(using:)` **[NEW]**
- `GKLocalPlayer.loadFriendsAuthorizationStatus(completionHandler:)` **[NEW]**
- `GKLocalPlayer.loadFriends()` async **[NEW]**
- `GKFriendsAuthorizationStatus` **[NEW]** (`.notDetermined`, `.denied`, `.restricted`, `.authorized`)
- `GKPlayer`
- `GKPlayer.displayName`
- `GKPlayer.loadPhoto(for:)` async
- `GKLeaderboard`
- `GKLeaderboard.loadLeaderboards(IDs:)` async
- `GKLeaderboard.loadEntries(for:timeScope:)` async (for friends)
- `GKLeaderboardEntry`
- `GKMatchRequest`
- `GKMatchRequest.minPlayers`
- `GKMatchRequest.maxPlayers`
- `GKMatchRequest.playerGroup`
- `GKMatchmakerViewController`
- `GKMatchmakerViewController.canStartWithMinimumPlayers` **[NEW]**
- `GKMatchmakerViewController(matchRequest:)`
- `GKMatchmakerViewController(invite:)`
- `GKMatchmakerViewControllerDelegate`
- `GKMatchmakerViewControllerDelegate.matchmakerViewController(_:didFind:)`
- `GKMatch`
- `GKMatch.delegate`
- `GKMatchDelegate.match(_:player:didChange:)`
- `GKPlayerConnectionState` (`.connected`)
- `GKInvite`
- `NSGKFriendListUsageDescription` (Info.plist key)

## Code Highlights

Friends API with async/await loading friends onto a progression map:
```swift
func loadFriendsOnProgressionMap() async {
    do {
        let friends = try await GKLocalPlayer.local.loadFriends()
        let leaderboards = try await GKLeaderboard.loadLeaderboards(IDs: ["progress"])
        if let leaderboard = leaderboards.first {
            let entries = try await leaderboard.loadEntries(for: friends, timeScope: .allTime)
            for entry in entries.1 {
                let avatar = try await entry.player.loadPhoto(for: .normal)
                // Display on progression map at entry.score level
            }
        }
    } catch { print("Error: \(error.localizedDescription)") }
}
```

Fast Start matchmaker setup:
```swift
let request = GKMatchRequest()
request.minPlayers = 2
request.maxPlayers = 6
let vc = GKMatchmakerViewController(matchRequest: request)
vc.canStartWithMinimumPlayers = true  // Enable Fast Start
vc.delegate = self
present(vc, animated: true)
```

Handling late-joining players in Fast Start game:
```swift
func match(_ match: GKMatch, player: GKPlayer, didChange state: GKPlayerConnectionState) {
    if state == .connected { addPlayer(player) }
}
```

## Takeaways

- All Game Center discoverability features (Friends Are Playing, Continue Playing widgets, App Store integration) are automatic for any Game Center-enabled game — no code changes required.
- Friends API requires `NSGKFriendListUsageDescription` in `Info.plist` and uses Swift async/await; always check `GKFriendsAuthorizationStatus` and delete friend data if `.denied` or `.restricted`.
- Fast Start (`canStartWithMinimumPlayers = true`) dramatically improves multiplayer game launch speed; implement `GKMatchDelegate.match(_:player:didChange:)` to handle players joining progressively.
- The new Suggestions Shelf and Game Center groups reduce the friction of replaying with the same group of friends, directly improving re-engagement.

---
_Source: WWDC21 Session 10066 page (abstract, chapter summaries, code samples, and resource links)._
