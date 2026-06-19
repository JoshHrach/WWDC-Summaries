# Architecting Your App for Multiple Windows
**WWDC19 · Session 258** · [Watch](https://developer.apple.com/videos/play/wwdc2019/258/)

_Platforms:_ iOS 13, iPadOS 13

## Overview
iOS 13 introduces support for multiple simultaneous UI instances (scenes) within a single app process, fundamentally changing the responsibility split between `UIApplicationDelegate` and the new `UISceneDelegate`. This session explains the architectural changes required to adopt multiple windows and how to properly structure an app to handle windows coming and going cleanly.

The session covers the lifecycle of `UISceneSession` objects, when scene delegates are called, how scene state restoration works under the new API, and best practices for keeping multiple windows synchronized when they share the same underlying data. It also explains how to back-deploy, maintaining compatibility with iOS 12 while adopting the new scene lifecycle on iOS 13.

A key architectural insight is that view controllers should never update their own UI directly in response to user events; instead they should notify a model controller which then broadcasts updates to all active scenes via notifications, delegates, or Combine—ensuring a consistent one-way data flow.

## Key Topics

**App Delegate vs. Scene Delegate Responsibilities**
- App Delegate remains responsible for process-level lifecycle events (launch, termination) and one-time non-UI setup (database connections, data structures)
- UI lifecycle (foreground/background state, window setup/teardown) moves entirely to `UISceneDelegate`
- UIKit stops calling UI-related App Delegate methods when the scene lifecycle is adopted; a 1-to-1 mapping exists for back-deployment

**UISceneSession Lifecycle**
- `application(_:configurationForConnecting:options:)` — called before a scene is created; returns a `UISceneConfiguration` specifying scene delegate class, storyboard, and optional scene subclass
- Configurations can be declared statically in `Info.plist` or dynamically in code
- `scene(_:willConnectTo:options:)` — set up the `UIWindow` using the new designated initializer; check for state restoration activities
- Scene disconnect: system may release a scene from memory after it backgrounds to reclaim resources; not permanent deletion
- `application(_:didDiscardSceneSessions:)` — called when user explicitly destroys a scene; permanently delete associated user data/state here; also called on next launch if scenes were discarded while process was not running

**Scene-Based State Restoration (New in iOS 13)**
- No longer encodes view hierarchies; encodes only the state needed to recreate the window
- Based on `NSUserActivity` — compatible with Spotlight, Handoff, and other activity-based technologies
- `stateRestorationActivity(for:)` on `UISceneDelegate` returns the activity to persist
- On reconnect, check `UISceneSession.stateRestorationActivity` and use it to restore the window
- State restoration archive respects the app's data protection class **[NEW]**

**Keeping Multiple Scenes in Sync**
- Anti-pattern: view controller mutates its own view on user event, then notifies model controller — second scene never updates
- Correct pattern: view controller only notifies model controller → model controller persists and broadcasts an update event to all subscribers
- Typed update events via Swift enum with associated values provide strong typing and debuggability
- Distribution mechanisms: `NotificationCenter`, delegate callbacks, or Combine framework **[NEW]**

## APIs & Frameworks

### UIKit — Scene Lifecycle (NEW)
- `UISceneDelegate` **[NEW]** — protocol replacing UI-related App Delegate methods
- `UIScene` **[NEW]** — represents a single UI instance
- `UISceneSession` **[NEW]** — persistent object representing a scene; survives scene disconnect
- `UISceneConfiguration` **[NEW]** — specifies delegate class, storyboard, and optional scene subclass
- `UIWindowScene` **[NEW]** — concrete `UIScene` subclass for windowed UIs
- `UIWindow.init(windowScene:)` **[NEW]** — designated initializer for windows in scenes
- `application(_:configurationForConnecting:options:)` on `UIApplicationDelegate` **[NEW]**
- `application(_:didDiscardSceneSessions:)` on `UIApplicationDelegate` **[NEW]**
- `scene(_:willConnectTo:options:)` on `UISceneDelegate` **[NEW]**
- `sceneWillResignActive(_:)` on `UISceneDelegate` **[NEW]**
- `sceneDidEnterBackground(_:)` on `UISceneDelegate` **[NEW]**
- `sceneDidDisconnect(_:)` on `UISceneDelegate` **[NEW]**
- `stateRestorationActivity(for:)` on `UISceneDelegate` **[NEW]**
- `UISceneSession.stateRestorationActivity` **[NEW]** — `NSUserActivity?` available on reconnect
- `UIApplicationSceneManifest` Info.plist key **[NEW]** — static scene configuration

### Foundation
- `NSUserActivity` — used as the carrier for scene-based state restoration
- `NotificationCenter` — one mechanism for broadcasting model update events across scenes

### Combine (NEW)
- `Combine` framework **[NEW]** — mentioned as a modern option for reactive cross-scene updates

## Code Highlights

Scene delegate setup with state restoration check:
```swift
func scene(_ scene: UIScene, willConnectTo session: UISceneSession,
           options connectionOptions: UIScene.ConnectionOptions) {
    guard let windowScene = scene as? UIWindowScene else { return }
    let window = UIWindow(windowScene: windowScene)
    // Check for state restoration activity
    if let activity = session.stateRestorationActivity {
        // Configure window using activity
    } else {
        // Fresh window
    }
    self.window = window
    window.makeKeyAndVisible()
}
```

Typed model update event enum:
```swift
enum UpdateEvent {
    case newMessage(Message)
}

extension UpdateEvent {
    func post() {
        NotificationCenter.default.post(
            name: .newMessage, object: self)
    }
}
```

View controller observing model updates:
```swift
NotificationCenter.default.addObserver(forName: .newMessage, ...) { note in
    guard let event = note.object as? UpdateEvent else { return }
    switch event {
    case .newMessage(let message):
        self.updateUI(with: message)
    }
}
```

## Takeaways
- Migrate all UI setup and teardown from `UIApplicationDelegate` to `UISceneDelegate`; App Delegate handles only process-level and one-time setup.
- Implement scene-based state restoration with `NSUserActivity`—it is now essential, not optional, because scenes can be silently disconnected in the background.
- Adopt a unidirectional data flow: view controllers notify the model controller, which broadcasts updates to all active scenes so every window stays consistent.
- Back-deployment is straightforward—keep both the old App Delegate UI methods and the new Scene Delegate methods; UIKit calls the right set at runtime.

---
_Source: WWDC19 Session 258 page (abstract, full transcript, and resource links)._
