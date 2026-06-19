# Window Management in Your Multitasking App
**WWDC19 · Session 246** · [Watch](https://developer.apple.com/videos/play/wwdc2019/246/)

_Platforms:_ iPadOS 13

## Overview
iPadOS 13 introduces the ability for apps to have multiple simultaneous windows (scenes), and this session dives into the three new programmatic scene session management APIs: requesting activation of a new or existing session, requesting a session refresh (including snapshot update for the App Switcher), and requesting session destruction with an appropriate animation.

The session is built around a demo app called "Clown Town" that illustrates all three APIs in a realistic context, showing how an app can open, track, refresh, and close auxiliary content windows while the user browses a primary map view — with the system handling all presentation, animation, and App Switcher representation automatically.

## Key Topics

### Scene Sessions Overview (Context)
- Each window on iPadOS is backed by a `UISceneSession`.
- An app can have multiple scene sessions simultaneously, each running the same or a different scene configuration.
- The new APIs allow apps to programmatically control the lifecycle of those sessions from code.

### Activating a Session **[NEW]**
- Call `UIApplication.shared.requestSceneSessionActivation(_:userActivity:options:errorHandler:)`.
- **Only call in response to direct, local user interaction** (the user must have touched the screen).
- Pass `nil` as the first parameter to request a brand-new session; pass an existing `UISceneSession` to bring it to the foreground (or reconnect it if disconnected).
- Pass a `NSUserActivity` that encodes what the new window should display (e.g., a specific item ID).
- Use `UISceneActivationRequestOptions` to specify a `requestingScene`: the system uses this to avoid replacing the source window with the activated one, and for layout decisions.
- Enforce "no duplicates" policy by checking for an existing session before calling the API.
- The delegate chain:
  1. `application(_:configurationForConnecting:options:)` on `UIApplicationDelegate` — inspect the user activity and return the appropriate `UISceneConfiguration` (pick the storyboard / class).
  2. `scene(_:willConnectTo:options:)` on `UISceneDelegate` — configure the window and view controller hierarchy for the user activity.
  3. If an existing session reconnects: `scene(_:continue:)` on the scene delegate (if scene still connected) or `scene(_:willConnectTo:options:)` (if it was disconnected and reconnected).

### Refreshing a Session **[NEW]**
- Call `UIApplication.shared.requestSceneSessionRefresh(_:)` at any time.
- Triggers: model changes from another window or device; new data fetched from a server; need to update scene/session metadata (title, user activity, activation conditions).
- What it updates:
  - State restoration `NSUserActivity` for the session.
  - `UISceneActivationConditions` (which user activities or predicates should activate this session).
  - UI snapshot captured for display in the App Switcher.
- The session is "background-connected" if needed: the scene is connected in the background, the view controller hierarchy updates itself (listening to the same notifications it would in the foreground), then a snapshot is captured.
- **Architectural pattern**: listen for model change notifications both in the view controller and in a long-lived object (e.g., app delegate or a singleton model). If the scene is disconnected, the long-lived object calls `requestSceneSessionRefresh` on its behalf.
- Make layout fast: the system needs to capture a snapshot quickly; keep your layout/rendering time minimal.
- Do not assume the refresh executes immediately; the system may defer it.

### Destroying a Session **[NEW]**
- Call `UIApplication.shared.requestSceneSessionDestruction(_:options:errorHandler:)` at any time.
- The session is permanently destroyed and will not be restored.
- Use `UISceneDestructionRequestOptions` to specify a `UIWindowSceneDismissalAnimation`:
  - `.standard` — the user explicitly requested destruction (e.g., cancelled a draft without saving); the window is dismissed as if discarded.
  - `.commit` — the session's final purpose was fulfilled (e.g., email sent, booking confirmed); the window commits its result.
  - `.decline` — the session's purpose was not fulfilled but the user is not asking for destruction (e.g., saved draft to return later); the window declines/defers.
- The dismissal animation only applies if the scene is in the foreground at the time of destruction.

## APIs & Frameworks

### UIKit — UIApplication
- `UIApplication.shared.requestSceneSessionActivation(_:userActivity:options:errorHandler:)` **[NEW]**
  - First parameter: `UISceneSession?` (nil = new session, non-nil = existing session)
  - `userActivity: NSUserActivity?` — activity to pass to the new/reconnected scene
  - `options: UISceneActivationRequestOptions?`
  - `errorHandler: ((Error) -> Void)?`
- `UIApplication.shared.requestSceneSessionRefresh(_:)` **[NEW]**
  - Parameter: `UISceneSession` — the session to refresh
- `UIApplication.shared.requestSceneSessionDestruction(_:options:errorHandler:)` **[NEW]**
  - First parameter: `UISceneSession`
  - `options: UISceneDestructionRequestOptions?`
  - `errorHandler: ((Error) -> Void)?`

### UIKit — Options
- `UISceneActivationRequestOptions` **[NEW]**
  - `requestingScene: UIScene?` — the scene that initiated the request
- `UISceneDestructionRequestOptions` **[NEW]**
  - `windowDismissalAnimation: UIWindowSceneDismissalAnimation` — `.standard`, `.commit`, `.decline`

### UIKit — App Delegate
- `UIApplicationDelegate.application(_:configurationForConnecting:options:)` — return `UISceneConfiguration` for a new or reconnecting session
- `UIApplicationDelegate.application(_:didDiscardSceneSessions:)` — called when sessions are discarded by the user from the switcher

### UIKit — Scene Delegate
- `UISceneDelegate.scene(_:willConnectTo:options:)` — configure a new or reconnecting session's window and view hierarchy
- `UISceneDelegate.scene(_:continue:)` — handle a user activity in an already-connected scene
- `UIWindowSceneDelegate.sceneDidBecomeActive(_:)`, `sceneWillResignActive(_:)`, etc. — standard lifecycle

### UIKit — Scene Session
- `UISceneSession` — represents a scene session
- `UISceneSession.stateRestorationActivity` — `NSUserActivity` for state restoration
- `UISceneActivationConditions` — predicates governing which user activities should activate a session **[NEW]**
- `UISceneConfiguration` — configuration name + storyboard + scene/delegate class
- `NSUserActivity` — carries content type and identifier for window configuration

## Code Highlights

Opening a new window (or activating an existing one) in response to user tap:
```swift
func openClown(_ clown: Clown, from scene: UIScene) {
    let activity = NSUserActivity(activityType: "com.example.clowntown.detail")
    activity.userInfo = ["clownID": clown.id]

    let options = UISceneActivationRequestOptions()
    options.requestingScene = scene

    // Check for existing session to avoid duplicates
    let existing = UIApplication.shared.openSessions
        .first { $0.stateRestorationActivity?.userInfo?["clownID"] as? String == clown.id }

    UIApplication.shared.requestSceneSessionActivation(
        existing,  // nil = new session
        userActivity: activity,
        options: options,
        errorHandler: nil
    )
}
```

Configuring the scene from the app delegate:
```swift
func application(_ application: UIApplication,
                 configurationForConnecting connectingSceneSession: UISceneSession,
                 options: UIScene.ConnectionOptions) -> UISceneConfiguration {
    if options.userActivities.first?.activityType == "com.example.clowntown.detail" {
        return UISceneConfiguration(name: "Detail Configuration",
                                    sessionRole: connectingSceneSession.role)
    }
    return UISceneConfiguration(name: "Default Configuration",
                                sessionRole: connectingSceneSession.role)
}
```

Refreshing a session after a model change:
```swift
func clownAvailabilityDidChange(_ clown: Clown) {
    for session in UIApplication.shared.openSessions {
        if session.stateRestorationActivity?.userInfo?["clownID"] as? String == clown.id {
            UIApplication.shared.requestSceneSessionRefresh(session)
        }
    }
}
```

Destroying a session when the user cancels a booking:
```swift
func cancelBooking(for session: UISceneSession) {
    let options = UISceneDestructionRequestOptions()
    options.windowDismissalAnimation = .standard  // user cancelled
    UIApplication.shared.requestSceneSessionDestruction(session, options: options)
}
```

## Takeaways
- Activation, refresh, and destruction APIs delegate complexity to UIKit/the system, letting apps gain full multitasking window behavior without managing presentation or animation themselves.
- Activation must always be triggered by direct user interaction; passing an existing session avoids duplicate windows.
- Refresh is how apps keep App Switcher snapshots up to date for background windows — listen for model changes from a long-lived object that can call the API even when the scene is disconnected.
- Destruction takes a dismissal animation style that communicates the user's intent (committed, declined, or discarded) to provide an appropriate transition.

---
_Source: WWDC19 Session 246 page (abstract, chapter summaries, code samples, and resource links)._
