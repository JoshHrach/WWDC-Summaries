# Reach new players with Game Center dashboard
**WWDC22 · Session 10064** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10064/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, tvOS 16

## Overview
This session introduces the redesigned Game Center dashboard with the new Activity feed — a unified social timeline that surfaces achievements earned, leaderboard jumps, and friend-versus-friend score battles across all Game Center-enabled games. The session covers what developers need to do to participate (minimal: just authenticate and enable Game Center), how to display the Access Point to players, and how to use the new Unity GameKit plug-in to implement the same functionality in C#. Activity is available automatically to all existing Game Center games without code changes.

## Key Topics

### Game Center Activity (NEW)
- The redesigned dashboard introduces an **Activity feed** that aggregates Game Center social events for a player's friends across all games.
- Activity events generated automatically by your game:
  - **Achievement unlocked** — when a player earns any achievement; when a player completes all achievements, a special celebratory event is shown.
  - **Leaderboard jump** — when a friend makes a large jump in rank on a leaderboard (e.g., enters the top 25%).
  - **Score beaten** — when a friend's score surpasses the local player's score on a leaderboard; this also generates a Game Center push notification to the local player (no notification permission request required from the developer).
- Activity events appear in the player's personal dashboard and, for leaderboard beats, generate system notifications without opt-in from the app.
- Existing Game Center games get Activity automatically — no code changes required.

### Enabling Game Center (Minimal Integration)
1. Add the **Game Center capability** in Xcode (Signing & Capabilities tab → + Capability → Game Center).
2. Enable Game Center for the app record in **App Store Connect** and configure leaderboards and achievements.
3. Authenticate the local player as early as possible (title screen):
   - Swift: Set `GKLocalPlayer.local.authenticateHandler`; present the returned view controller if non-nil.
   - Unity (C#): `await GKLocalPlayer.Authenticate()`.
4. Show the **Access Point** on the menu screen to give players easy access to the dashboard.

### Game Center Access Point
- `GKAccessPoint.shared` — the system-managed button that launches the Game Center dashboard.
- `GKAccessPoint.shared.location` — set to `.topLeading`, `.topTrailing`, `.bottomLeading`, or `.bottomTrailing`.
- `GKAccessPoint.shared.isActive = true` — shows the access point; set `false` to hide during gameplay.
- When a player taps the Access Point, the Game Center dashboard sheet appears over the game showing Activity, achievements, leaderboards, and the player's profile.

### Leaderboards and Achievements for Maximum Activity
- Every leaderboard score submission creates potential Activity events for the player's friends.
- More leaderboards = more Activity moments = more re-engagement opportunities.
- **Recurring leaderboards** are especially effective: they reset on a schedule (daily/weekly), giving players ongoing reasons to return.
- Achievements provide progress milestones; completing the full set generates a special recognition event.
- Both leaderboards and achievements are configured in App Store Connect, not in code.

### Unity GameKit Plug-in (NEW)
- A new Unity plug-in ships the full GameKit API in C#, developed by Apple.
- Available as part of the Apple Unity plug-ins at `github.com/apple/unityplugins`.
- Requires Apple.Core plug-in as a dependency.
- All code examples in the session are shown in both Swift and C#.
- See companion session "Plug-in and play: Add Apple frameworks to your Unity game projects" (WWDC22-10065) for plug-in setup details.

## APIs & Frameworks

**GameKit**
- `GKLocalPlayer.local.authenticateHandler: ((UIViewController?, Error?) -> Void)?` — set to trigger Game Center authentication; present the view controller if provided
- `GKLocalPlayer` — **[authenticateHandler pattern unchanged; Activity events are new]**
- `GKAccessPoint.shared` — singleton access point UI widget **[NEW dashboard behavior]**
  - `.location: GKAccessPoint.Location` — `.topLeading`, `.topTrailing`, `.bottomLeading`, `.bottomTrailing`
  - `.isActive: Bool` — show/hide the access point button
- `GKLeaderboard` — leaderboard score submission triggers Activity events **[Activity feed is NEW]**
  - `GKLeaderboard.submitScore(_:context:player:leaderboardIDs:completionHandler:)` — submit a score
- `GKAchievement` — achievement reporting triggers Activity events **[Activity feed is NEW]**
  - `GKAchievement(identifier:)` — create achievement progress object
  - `.percentComplete: Double`
  - `GKAchievement.report(_:withCompletionHandler:)` — report progress

**App Store Connect**
- Game Center configuration: leaderboards (classic and recurring), achievements, challenges
- Recurring leaderboards — set reset interval (daily, weekly, etc.)

**Unity GameKit Plug-in (C#)** **[NEW]**
- `GKLocalPlayer.Authenticate()` — `async Task<GKLocalPlayer>`
- `GKAccessPoint.Shared.Location` — `GKAccessPointLocation` enum
- `GKAccessPoint.Shared.IsActive` — `bool`

## Code Highlights

Authenticate the local player (Swift):
```swift
import GameKit

class TitleScreenViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        GKLocalPlayer.local.authenticateHandler = { viewController, error in
            if let viewController = viewController {
                self.present(viewController, animated: true)
            }
        }
    }
}
```

Authenticate the local player (Unity C#):
```csharp
using Apple.GameKit;

public class MyGameManager : MonoBehaviour {
    private GKLocalPlayer _localPlayer;

    private async Task Start() {
        try {
            _localPlayer = await GKLocalPlayer.Authenticate();
        } catch (Exception exception) {
            // Handle exception...
        }
    }
}
```

Show the Access Point (Swift):
```swift
import GameKit

class MenuScreenViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        GKAccessPoint.shared.location = .topLeading
        GKAccessPoint.shared.isActive = true
    }
}
```

Show the Access Point (Unity C#):
```csharp
GKAccessPoint.Shared.Location = GKAccessPoint.GKAccessPointLocation.TopLeading;
GKAccessPoint.Shared.IsActive = true;
```

## Takeaways
- Game Center Activity is free distribution: any Game Center-enabled game automatically generates social Activity events (achievements, leaderboard jumps, score beats) that are visible to friends across the entire Game Center network with no additional code.
- The minimum integration footprint is tiny — add the capability, authenticate once, show the Access Point — and the full dashboard (with Activity feed) is immediately available to your players.
- Leaderboard score beats generate Game Center push notifications to the losing player without requiring the app to request notification permissions.
- Recurring leaderboards are the highest-value Activity driver: they create regular, time-bounded competition cycles that repeatedly surface your game in friends' feeds.
- Unity developers now have full GameKit access in C# via the new Apple Unity GameKit plug-in.

---
_Source: WWDC22 Session 10064 page (abstract, transcript, code samples, and related session links)._
