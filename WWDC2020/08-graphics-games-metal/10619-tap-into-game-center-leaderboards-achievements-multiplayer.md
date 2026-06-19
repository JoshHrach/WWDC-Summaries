# Tap into Game Center: Leaderboards, Achievements, and Multiplayer
**WWDC20 · Session 10619** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10619/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, tvOS 14

## Overview
This session is the second part of the WWDC20 Game Center deep-dive. It covers the new recurring leaderboards feature, achievement improvements, and updates to multiplayer matchmaking. The classic "never-expiring" leaderboard is unchanged; the new recurring leaderboard introduces time-bounded competitions (15-minute, hourly, daily, weekly, etc.) where scores reset on a configured schedule, giving every player a fresh chance at the top spot.

Achievements receive a design refresh and land in the new in-game dashboard, with an updated badge style and a progress ring for in-progress items. The achievement limit is raised to 100 per game (was 50) with a cumulative point budget of 1,000. Multiplayer matchmaking gains a streamlined UI with improved player suggestions, a new auto-match-only mode, and a nearby-only mode. The `personalizedCommunicationRestricted` flag introduced in session 10618 extends to multiplayer: when true, in-game voice chat and custom invite messages must be disabled.

## Key Topics
- **Recurring leaderboards** — time-bounded competitions with configurable start date, frequency (restart period), and duration; occurrences never overlap; up to one previous occurrence accessible per player **[NEW]**
- **Classic leaderboards** — unchanged; retain best or most-recent score forever
- **Score submission API** — new `GKLeaderboard.submitScore(_:context:player:leaderboardIDs:)` class method replaces deprecated instance-based submission **[NEW]**
- **Occurrence navigation** — `GKLeaderboard.loadLeaderboards(IDs:)` loads current occurrence; `loadPreviousOccurrence` navigates history (up to 30 days retained)
- **Achievement limit increase** — 100 achievements, 1,000 total points **[UP from 50/1,000]**
- **Achievement states** — locked, in-progress (new progress ring UI), completed, hidden
- **Achievement reporting** — `GKAchievement(identifier:)` + `percentComplete` + `GKAchievement.report(_:)`
- **Achievement descriptions** — `GKAchievementDescription` for custom UI; localized title, `achievedDescription`, `unachievedDescription`, images
- **Multiplayer matchmaking** — `GKMatchRequest`, `GKMatchmaker`, `GKMatchmakerViewController`; new auto-match-only and nearby-only options **[NEW]**
- **Real-time vs. turn-based** — `GKMatch` + `GKInviteEventListener` for real-time; `GKTurnBasedMatch` + `GKTurnBasedEventListener` for turn-based
- **Privacy restriction** — `GKLocalPlayer.local.personalizedCommunicationRestricted` disables voice chat and custom invite messages

## APIs & Frameworks

**GameKit — Leaderboards**
- `GKLeaderboard` — represents a classic or recurring leaderboard / occurrence
- `GKLeaderboard.submitScore(_:context:player:leaderboardIDs:completionHandler:)` **[NEW]** — class method to submit score to one or more leaderboard IDs at once
- `GKLeaderboard.loadLeaderboards(IDs:completionHandler:)` **[NEW]** — loads GKLeaderboard instances for given IDs (current occurrence for recurring)
- `GKLeaderboard.submitScore(_:context:player:completionHandler:)` — instance method for submitting to a specific occurrence
- `GKLeaderboard.loadPreviousOccurrence(completionHandler:)` **[NEW]** — loads the previous occurrence of a recurring leaderboard
- `GKLeaderboard.startDate` — start of the occurrence window
- `GKLeaderboard.duration` — duration of the occurrence window
- `GKLeaderboard.nextStartDate` — when the next occurrence begins
- `GKLeaderboard.loadEntries(for:timeScope:range:completionHandler:)` — load scores for display
- `GKLeaderboard.baseLeaderboardID` — the ID shared across all occurrences of a recurring leaderboard

**GameKit — Achievements**
- `GKAchievement` — represents a player's progress on a single achievement
- `GKAchievement.init(identifier:)` — initialize with the App Store Connect achievement ID
- `GKAchievement.percentComplete` — Double (0–100)
- `GKAchievement.report(_:withCompletionHandler:)` — class method to submit progress
- `GKAchievementDescription` — metadata class: `identifier`, `title`, `achievedDescription`, `unachievedDescription`, `maximumPoints`, `isHidden`, `isReplayable`
- `GKAchievementDescription.loadAchievementDescriptions(completionHandler:)` — loads all descriptions
- `GKAchievementDescription.loadImage(completionHandler:)` — loads achieved image
- `GKGameCenterViewController(state: .achievements)` — present the in-game achievements page

**GameKit — Matchmaking**
- `GKMatchRequest` — configure player count, invited players, player group/attributes
- `GKMatchRequest.minPlayers`, `GKMatchRequest.maxPlayers`
- `GKMatchRequest.inviteMessage` — custom invite message (disabled when `personalizedCommunicationRestricted`)
- `GKMatchmakerViewController` — native matchmaking UI; new configuration options
- `GKMatchmakerViewControllerDelegate`
- `GKMatchmaker.shared` — singleton for programmatic matchmaking
- `GKMatchmaker.shared.findMatch(for:withCompletionHandler:)` — auto-match
- `GKMatch` — real-time match object; `send(_:to:dataMode:error:)`, `sendData(toAllPlayers:with:error:)`
- `GKInviteEventListener` — protocol; register for incoming real-time invites
- `GKTurnBasedMatch` — turn-based match; `findMatch(for:withCompletionHandler:)`, `endTurn(withNextParticipants:turnTimeout:match:completionHandler:)`
- `GKTurnBasedEventListener` — protocol; register for turn events
- `GKMatchmakerViewController` new options **[NEW]**:
  - Auto-match only mode — skip player picker, go straight to auto-matching
  - Nearby only mode — restrict matchmaking to nearby players (e.g., ARKit co-located games)
- `GKLocalPlayer.local.personalizedCommunicationRestricted` **[NEW]** — disable voice chat and custom messaging when `true`

## Code Highlights

Submit a score to one or more leaderboards (new class method):
```swift
GKLeaderboard.submitScore(self.points, context: 0, player: GKLocalPlayer.local,
    leaderboardIDs: ["my.leaderboard.id"]) { error in
    if let error = error { print(error) }
}
```

Load and navigate recurring leaderboard occurrences:
```swift
GKLeaderboard.loadLeaderboards(IDs: ["my.recurring.leaderboard.id"]) { fetchedLBs, error in
    if let current = fetchedLBs?.first {
        current.loadPreviousOccurrence { prevOccurrence, error in
            // use prevOccurrence
        }
    }
}
```

Report achievement progress:
```swift
if let achievement = GKAchievement(identifier: identifier) {
    achievement.percentComplete = percentComplete
    GKAchievement.report([achievement]) { error in
        if let error = error { print("Error: \(error)") }
    }
}
```

Present the achievements page:
```swift
let vc = GKGameCenterViewController(state: .achievements)
vc.gameCenterDelegate = self
present(vc, animated: true)
```

Disable voice chat when communication is restricted:
```swift
if GKLocalPlayer.local.personalizedCommunicationRestricted {
    voiceChatButton.isEnabled = false
}
```

## Takeaways
- Recurring leaderboards enable time-bounded competitions configured entirely in App Store Connect (start date, frequency/restart period, duration); the new `GKLeaderboard.submitScore` class method is the preferred submission path for both classic and recurring leaderboards.
- Each player can access at most the current occurrence and one previous occurrence; expired occurrences are purged after 30 days.
- Achievements cap at 100 items and 1,000 total points—plan headroom for future updates; use `GKAchievement.report` with `percentComplete` and let the system handle the new in-progress badge styling.
- Check `personalizedCommunicationRestricted` before enabling any in-game voice or custom messaging feature in multiplayer; the new nearby-only matchmaking mode simplifies co-located game setups.

---
_Source: WWDC20 Session 10619 page (abstract, chapter summaries, code samples, and resource links)._
