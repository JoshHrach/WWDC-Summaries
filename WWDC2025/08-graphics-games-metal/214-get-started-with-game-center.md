# Get started with Game Center

**Session ID:** 214  
**WWDC Year:** 2025  
**Folder:** `08-graphics-games-metal`  
**URL:** https://developer.apple.com/videos/play/wwdc2025/214/

---

## Overview

This session introduces the major Game Center updates in iOS 26 / macOS 26, anchored by two new social features: Challenges and Activities. Challenges allow players to issue head-to-head competitive challenges to friends around a specific achievement or score. Activities are flexible developer-defined game events that can be tied to Challenges, leaderboards, or standalone social feeds. The session also covers party codes for grouping friends into multiplayer sessions, GameKit bundles for sharing assets across a game family, and the new `GKGameActivity` API that underpins Activities and Challenges.

---

## Key Topics

- Game Center Challenges: issuing and receiving head-to-head challenges between friends
- Game Center Activities: developer-defined game events surfaced in the Game Center social feed
- `GKGameActivity` API: creating and completing activity instances
- Party codes: shareable codes for grouping players into a multiplayer lobby
- GameKit bundles: sharing leaderboards, achievements, and assets across a suite of games
- Updated Game Center dashboard and in-game UI components
- Friend presence and activity feeds

---

## APIs & Frameworks

- **GameKit** framework (`import GameKit`) – Core framework for Game Center integration.
- **`GKGameActivity`** – **[NEW]** (iOS 26, macOS 26) Represents a discrete in-game event or accomplishment that can be shared socially and used as the basis for a Challenge. Properties: `activityID: String`, `title: String`, `metadata: [String: String]`.
- **`GKGameActivity.report(_:completionHandler:)`** – **[NEW]** Reports a completed activity instance to Game Center, triggering social feed updates and enabling Challenge resolution.
- **`GKChallenge`** – Existing type; extended in iOS 26 with `GKActivityChallenge` subclass.
- **`GKActivityChallenge`** – **[NEW]** Challenge subtype tied to a `GKGameActivity`; issued between friends; properties include `challenger`, `recipient`, `activityID`, `state`.
- **`GKLocalPlayer.issueChallenge(_:to:message:completionHandler:)`** – **[NEW]** Issues a `GKActivityChallenge` from the local player to a `GKPlayer` friend.
- **`GKMatchmakerViewController` party code support** – **[NEW]** `GKMatchRequest.partyCode` property; display a generated party code in your UI so friends can join a multiplayer session by entering the code.
- **`GKMatchRequest.partyCode`** – **[NEW]** `String?` property; set to a previously generated code to join an existing party, or read the newly generated code after submitting the match request.
- **GameKit Bundle** – **[NEW]** Logical grouping configured in App Store Connect linking multiple games in a family; shared leaderboards and achievements appear across all member apps. No new API required in app code; configured server-side.
- **`GKLeaderboard` / `GKAchievement`** – Existing APIs; now work within GameKit bundles for cross-game data.
- **`GKGameCenterViewController`** – Existing dashboard view controller; updated UI in iOS 26 to surface Activities, Challenges, and party invite flows.

---

## Code Highlights

Reporting a game activity:
```swift
import GameKit

let activity = GKGameActivity(activityID: "com.example.game.level5.complete")
activity.title = "Completed Level 5"
activity.metadata = ["score": "12500", "time": "02:34"]

GKGameActivity.report(activity) { error in
    if let error { print("Activity report failed: \(error)") }
}
```

Issuing a challenge to a friend:
```swift
// After reporting the activity, issue a challenge
let challenge = GKActivityChallenge(activityID: "com.example.game.level5.complete")
GKLocalPlayer.local.issueChallenge(challenge, to: friendPlayer, message: "Beat my time!") { error in
    if let error { print("Challenge failed: \(error)") }
}
```

Generating a party code for multiplayer:
```swift
let request = GKMatchRequest()
request.minPlayers = 2
request.maxPlayers = 4

let matchmakingVC = GKMatchmakerViewController(matchRequest: request)
// After presenting, read the generated code:
if let partyCode = request.partyCode {
    displayPartyCode(partyCode) // Show to user to share with friends
}
```

---

## Takeaways

- `GKGameActivity` is the new social primitive in Game Center; it replaces ad-hoc achievement/leaderboard combinations as the way to surface meaningful in-game moments.
- Activities and Challenges together create a closed social loop: player completes an activity, posts it socially, issues a challenge, friend accepts and tries to beat it.
- Party codes significantly reduce friction for friends to join the same multiplayer session without needing to navigate the full matchmaker UI.
- GameKit bundles are configured entirely in App Store Connect with no new app-side API, making cross-game social features easy to add to an existing game family.
- The updated Game Center dashboard and in-game UI components are system-provided, so apps automatically get the refreshed Liquid Glass-styled UI in iOS 26.
- Implement `GKGameCenterControllerDelegate` to handle dashboard dismiss events and keep your app's state synchronized.
