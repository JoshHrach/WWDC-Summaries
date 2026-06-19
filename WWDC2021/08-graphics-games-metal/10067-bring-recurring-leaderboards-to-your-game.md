# Bring Recurring Leaderboards to Your Game
**WWDC21 · Session 10067** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10067/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, tvOS 15

## Overview
This code-along session walks through configuring a recurring leaderboard in App Store Connect and then integrating it into a SpriteKit game ("The Coast") using three progressively richer patterns: linking to Game Center's built-in leaderboard UI, displaying live global rankings during gameplay, and comparing a player's current occurrence rank against their rank in the previous occurrence.

Recurring leaderboards differ from classic leaderboards in that each occurrence has a start time and a finite duration; after it expires, a new occurrence begins automatically on the configured schedule. This makes them ideal for timed daily or weekly challenges where fresh top spots are available to every player.

## Key Topics

### Recurring vs. Classic Leaderboards
- **Classic**: always active, no end date; rolling daily/weekly views plus all-time; suited for cumulative/all-time stats
- **Recurring**: developer-defined start date, duration, and restart interval; occurrences never overlap; access to current active occurrence and most recently finished occurrence (up to 30 days after end time)
- Recurring leaderboards always use `.allTime` as the `timeScope` in all GameKit calls — they have their own occurrence-based scoping

### App Store Connect Configuration
Required fields for recurring leaderboards (beyond classic fields):
- **Start Date and Time** (UTC): first occurrence start — must be in the future
- **Duration**: how long scores can be submitted to one occurrence (e.g., 1 day)
- **Restarts Every**: interval between occurrence starts; must be ≥ duration (no gaps = set equal to duration)
- Score format type, submission type (best/most recent), sort order, and optional score range apply to all occurrences

### Pattern 1: Link to Game Center Leaderboard UI
- Authenticate `GKLocalPlayer` in `viewDidLoad` or `viewWillAppear`
- Present `GKGameCenterViewController(leaderboardID:playerScope:timeScope:)` with `.allTime` time scope
- Submit scores using `GKLeaderboard.submitScore(_:context:player:leaderboardIDs:completionHandler:)` (class method)

### Pattern 2: Live Global Rankings During Gameplay
- Load the leaderboard instance with `GKLeaderboard.loadLeaderboards(IDs:completionHandler:)`
- Use the instance method `GKLeaderboard.loadEntries(for:timeScope:range:completionHandler:)` to get top-N global entries
- Call periodically (e.g., on each score submission) to keep an in-scene leaderboard node up to date
- `timeScope` for recurring leaderboards must be `.allTime`

### Pattern 3: Current vs. Previous Occurrence Rank
- Use GameKit's new `async`/`await` APIs (iOS 15) to load both occurrences concurrently with `async let`
- Load previous occurrence via `GKLeaderboard.loadPreviousOccurrence(completionHandler:)` then call `loadEntries` on it
- Display the rank delta to motivate replays

### Class Submit vs. Instance Submit
- **Class method** (`GKLeaderboard.submitScore`): submits to whatever occurrence is currently active — always succeeds if an occurrence is running; appropriate when session boundaries don't need to align with occurrence boundaries
- **Instance method** (on loaded `GKLeaderboard`): submits to the specific occurrence loaded at the time; fails if that occurrence has since ended — use when score integrity per occurrence is critical (e.g., hourly challenge games)

## APIs & Frameworks

### GameKit
- `GKLocalPlayer.local.authenticateHandler: ((UIViewController?, Error?) -> Void)?` — set in view controller to trigger sign-in flow
- `GKLeaderboard.submitScore(_:context:player:leaderboardIDs:completionHandler:)` **[class method]** — submit to currently active occurrence
- `GKLeaderboard.loadLeaderboards(IDs:completionHandler:)` — load `GKLeaderboard` instances by ID
  - Completion: `([GKLeaderboard]?, Error?) -> Void`
- `GKLeaderboard.loadEntries(for:timeScope:range:completionHandler:)` — load top entries or local player entry
  - `for: [GKPlayer]` — pass empty `[]` for local player only
  - `range: NSRange` — e.g., `NSRange(location: 1, length: 5)` for top 5
  - `timeScope: GKLeaderboard.TimeScope` — must be `.allTime` for recurring leaderboards
  - Completion: `(GKLeaderboard.Entry?, [GKLeaderboard.Entry]?, Int, Error?) -> Void`
- `GKLeaderboard.loadPreviousOccurrence(completionHandler:)` **[NEW]** — fetch the most recently completed occurrence (up to 30 days after end)
- `GKLeaderboard.Entry.rank: Int` — player rank in that occurrence
- `GKLeaderboard.Entry.score: Int` — player score
- `GKLeaderboard.Entry.player: GKPlayer`
- `GKGameCenterViewController(leaderboardID:playerScope:timeScope:)` — present Game Center leaderboard UI
  - `playerScope: GKLeaderboard.PlayerScope` — `.global` or `.friendsOnly`
  - `timeScope: GKLeaderboard.TimeScope` — use `.allTime` for recurring

### Swift Concurrency (GameKit async APIs — iOS 15)
- `GKLeaderboard.loadLeaderboards(IDs:)` **async** overload **[NEW]**
- `GKLeaderboard.loadEntries(for:timeScope:range:)` **async** overload **[NEW]**
- `GKLeaderboard.loadPreviousOccurrence()` **async** overload **[NEW]**

## Code Highlights

Submit score (class method — current occurrence):
```swift
GKLeaderboard.submitScore(
    score,
    context: 0,
    player: GKLocalPlayer.local,
    leaderboardIDs: ["DailyHighScore"]
) { error in
    if let error { print(error) }
    self.updateLeaderboardNode()
}
```

Load and display live global top-5 entries:
```swift
func updateLeaderboardNode() {
    GKLeaderboard.loadLeaderboards(IDs: ["DailyHighScore"]) { leaderboards, error in
        guard let leaderboard = leaderboards?.first else { return }
        leaderboard.loadEntries(
            for: [],
            timeScope: .allTime,
            range: NSRange(location: 1, length: 5)
        ) { _, entries, _, error in
            guard let entries else { return }
            let rows = entries.map { LeaderboardEntry(name: $0.player.displayName, score: $0.score) }
            DispatchQueue.main.async { self.leaderboardNode?.updateEntries(rows) }
        }
    }
}
```

Previous vs. current occurrence rank (async/await):
```swift
func addRankToGameMenu() async {
    do {
        let leaderboards = try await GKLeaderboard.loadLeaderboards(IDs: ["DailyHighScore"])
        guard let leaderboard = leaderboards.first else { return }
        async let currentEntry = leaderboard.loadEntries(for: [], timeScope: .allTime, range: NSRange(location: 1, length: 1))
        async let previousLeaderboard = leaderboard.loadPreviousOccurrence()
        let (current, prev) = try await (currentEntry, previousLeaderboard)
        let prevEntry = try await prev?.loadEntries(for: [], timeScope: .allTime, range: NSRange(location: 1, length: 1))
        gameMenuNode.addRankNode(current: current.0?.rank, previous: prevEntry?.0?.rank)
    } catch { print(error) }
}
```

## Takeaways
- Configure recurring leaderboards entirely in App Store Connect (start date, duration, restart interval) — no code required for scheduling; the system handles occurrence lifecycle automatically.
- Always pass `.allTime` as the `timeScope` in all `GKLeaderboard` API calls when working with recurring leaderboards — occurrence scoping is handled by the occurrence model, not the time scope.
- Use the class submit method for most games; switch to instance submit only when you need to guarantee that a score belongs to a specific occurrence (e.g., an hourly challenge).
- GameKit's new `async`/`await` APIs (iOS 15) make it straightforward to load multiple leaderboard occurrences concurrently with `async let`, enabling rank-change comparisons without callback nesting.

---
_Source: WWDC21 Session 10067 page (abstract, transcript, and resource links)._
