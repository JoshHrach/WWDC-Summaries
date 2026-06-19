# Introducing Multiple Windows on iPad
**WWDC19 · Session 212** · [Watch](https://developer.apple.com/videos/play/wwdc2019/212/)

_Platforms:_ iOS 13, iPadOS 13

## Overview
iPadOS 13 introduces first-class support for multiple windows (scenes) in any app, replacing the single-window model that has existed since iOS's inception. The App Switcher now shows windows (scenes) rather than apps, and every app can have multiple concurrent scenes visible in Split View, Slide Over, or full screen. This session covers the design philosophy, adoption path, lifecycle changes, state restoration, and common pitfalls.

The core programming model change is the introduction of `UIWindowScene` and `UISceneSession`. `UIWindowScene` holds the actual UI (windows, view controllers), while `UISceneSession` is a persistent identifier for a scene slot that survives even when the scene's UI is not loaded. Application lifecycle events (foreground, background, active) now fire per-scene rather than per-process; the application's aggregate state reflects the most active scene.

This session is the introductory talk. Advanced topics, animations, and distribution details are covered in the companion session "Getting the Most out of Multitasking" (Session 242) and other WWDC19 talks.

## Key Topics

**Design: Should Your App Support Multiple Windows?**
All of Apple's first-party apps support multiple windows. Common patterns:
- All windows are equal (Safari, Maps): any window can do everything; user creates more for convenience.
- Document-based apps (Pages, Files): each window shows a document; browser-style navigation still present in each window.
- Dedicated/transactional windows (Mail compose, Messages thread): windows are tied to a specific action or piece of content and have a "Done"/"Send" button to close themselves.
- The first rule: every single window must be able to do everything in the app. Multiple windows should never be required for normal use.

**Enabling Multiple Windows**
Check "Supports multiple windows" in Xcode's General tab. This adds an `UIApplicationSceneManifest` entry to `Info.plist`, declaring the scene configuration and enabling the new lifecycle APIs. The default Xcode project template uses the scene lifecycle.

**UIWindowScene and UISceneSession**
- `UIWindowScene` replaces the role of the root window in the old model; it holds windows, manages interface orientation, status bar, etc.
- `UISceneSession` is a persisted token with a stable `persistentIdentifier` string. Sessions survive app termination. The session stores a `stateRestorationActivity: NSUserActivity?` and a `userInfo: [String: Any]?` dictionary for per-scene settings.
- Scene delegates replace the parts of `UIApplicationDelegate` that deal with UI: `UISceneDelegate` gets `willConnectTo(session:options:)`, `sceneDidBecomeActive`, `sceneWillResignActive`, `sceneWillEnterForeground`, `sceneDidEnterBackground`, `sceneDidDisconnect`.
- `UIApplicationDelegate` retains process-level events: launch, termination, process-level notifications.

**Application Lifecycle with Scenes**
The process state (foreground active, background) reflects the most foreground scene. If any scene is foreground active, the application reports foreground active. If all scenes background, the application reports background.

**State Restoration**
State restoration uses `NSUserActivity`. The scene delegate implements `stateRestorationActivity(for:)` to return the activity, and reads `connectionOptions.userActivities` (for Handoff) or `session.stateRestorationActivity` (for local restoration) in `willConnectTo(session:options:)`. State must be stored per-scene (using `session.persistentIdentifier`) when files or databases are involved — storing to a single location causes content to be overwritten by the other scene.

**Programmatic Scene Management**
- `UIApplication.requestSceneSessionActivation(_:userActivity:options:completionHandler:)` — bring forward an existing scene or request a new one (pass `nil` for the session to create new) **[NEW]**
- `UIApplication.requestSceneSessionRefresh(_:)` — schedule a background connection to update the App Switcher snapshot and state restoration activity **[NEW]**
- `UIApplication.requestSceneSessionDestruction(_:options:completionHandler:)` — close a scene with a specified dismissal animation **[NEW]**
- `UISceneDestructionRequestOptions` / `UIWindowSceneDestructionRequestOptions` — choose close animation (e.g., dismiss up like Send, dismiss down like Save Draft) **[NEW]**
- `UIApplicationDelegate.application(_:didDiscardSceneSessions:)` — called when the user destroys a scene in the App Switcher; clean up per-scene persistent data here **[NEW]**

**Creating Windows via Drag and Drop**
The primary user gesture for creating a second window is dragging the app icon from the Dock to a side of the screen. Dragging content (e.g., a table row, a tab) to the screen edge should also open a new window. Implement by including an `NSUserActivity` as an alternate representation in the drag item's `NSItemProvider`. Universal links and declared file URLs work automatically.

**Deprecated UIApplication Properties**
Properties like `statusBarStyle`, `statusBarOrientation`, `isStatusBarHidden`, `keyWindow` are deprecated in iOS 13 in favor of per-scene equivalents on `UIWindowScene`:
- `UIWindowScene.statusBarManager` **[NEW]**
- `UIWindowScene.interfaceOrientation` **[NEW]**
- `UIWindowScene.windows` **[NEW]**
- `UIScene.activationState` **[NEW]**

**Common Pitfalls (Case Studies)**
1. Shared file storage: both scenes writing to the same file path causes data loss. Fix: key storage paths to `session.persistentIdentifier`.
2. Delegate-based state propagation: a setting changed in one scene's view controller notifies only the delegate in the same scene. Fix: use `UserDefaults` + KVO with the `.initial` option so all scenes observe the same source of truth and receive the current value on setup.

## APIs & Frameworks

**UIKit** (iOS 13 / iPadOS 13) **[NEW]**

Scene infrastructure:
- `UIWindowScene` **[NEW]**
- `UISceneSession` **[NEW]**
  - `UISceneSession.persistentIdentifier: String` **[NEW]**
  - `UISceneSession.stateRestorationActivity: NSUserActivity?` **[NEW]**
  - `UISceneSession.userInfo: [String: Any]?` **[NEW]**
  - `UISceneSession.role: UISceneSession.Role` **[NEW]**
- `UIScene` **[NEW]**
  - `UIScene.activationState: UIScene.ActivationState` **[NEW]**
- `UISceneDelegate` protocol **[NEW]**
  - `scene(_:willConnectTo:options:)` **[NEW]**
  - `sceneDidBecomeActive(_:)` **[NEW]**
  - `sceneWillResignActive(_:)` **[NEW]**
  - `sceneWillEnterForeground(_:)` **[NEW]**
  - `sceneDidEnterBackground(_:)` **[NEW]**
  - `sceneDidDisconnect(_:)` **[NEW]**
  - `stateRestorationActivity(for:) -> NSUserActivity?` **[NEW]**
- `UIWindowSceneDelegate` protocol **[NEW]**
- `UISceneConnectionOptions` **[NEW]**
  - `UISceneConnectionOptions.userActivities: Set<NSUserActivity>` **[NEW]**
  - `UISceneConnectionOptions.stateRestorationActivity: NSUserActivity?` (accessed via session) **[NEW]**

Programmatic control:
- `UIApplication.requestSceneSessionActivation(_:userActivity:options:completionHandler:)` **[NEW]**
- `UIApplication.requestSceneSessionRefresh(_:)` **[NEW]**
- `UIApplication.requestSceneSessionDestruction(_:options:completionHandler:)` **[NEW]**
- `UIWindowSceneDestructionRequestOptions` **[NEW]**
- `UIApplicationDelegate.application(_:didDiscardSceneSessions:)` **[NEW]**

UIWindowScene properties:
- `UIWindowScene.statusBarManager` **[NEW]**
- `UIWindowScene.interfaceOrientation: UIInterfaceOrientation` **[NEW]**
- `UIWindowScene.windows: [UIWindow]` **[NEW]**
- `UIWindowScene.screen: UIScreen` **[NEW]**

Info.plist key:
- `UIApplicationSceneManifest` **[NEW]**
- `UIApplicationSupportsMultipleScenes` **[NEW]**
- `UISceneConfigurations` **[NEW]**

**Deprecated UIApplication properties** (in iOS 13)
- `UIApplication.statusBarStyle`, `statusBarOrientation`, `isStatusBarHidden`
- `UIApplication.keyWindow` (use `UIWindowScene.windows`)

## Code Highlights

Enabling multiple windows (Info.plist key added by Xcode):
```xml
<key>UIApplicationSceneManifest</key>
<dict>
    <key>UIApplicationSupportsMultipleScenes</key>
    <true/>
    <key>UISceneConfigurations</key>
    <dict>
        <key>UIWindowSceneSessionRoleApplication</key>
        <array>
            <dict>
                <key>UISceneConfigurationName</key>
                <string>Default Configuration</string>
                <key>UISceneDelegateClassName</key>
                <string>$(PRODUCT_MODULE_NAME).SceneDelegate</string>
            </dict>
        </array>
    </dict>
</dict>
```

Scene delegate restoring state:
```swift
func scene(_ scene: UIScene, willConnectTo session: UISceneSession,
           options connectionOptions: UISceneConnectionOptions) {
    let activity = connectionOptions.userActivities.first
        ?? session.stateRestorationActivity
    if let activity = activity {
        // Restore UI from activity
    }
}

func stateRestorationActivity(for scene: UIScene) -> NSUserActivity? {
    return (scene as? UIWindowScene)?.windows.first?.rootViewController?.userActivity
}
```

Opening a new window programmatically:
```swift
let activity = NSUserActivity(activityType: "com.example.openDocument")
activity.userInfo = ["documentID": documentID]
UIApplication.shared.requestSceneSessionActivation(nil, userActivity: activity, options: nil)
```

Using KVO on UserDefaults to sync state across scenes:
```swift
extension UserDefaults {
    @objc dynamic var isInfoBarHidden: Bool {
        get { bool(forKey: "isInfoBarHidden") }
        set { set(newValue, forKey: "isInfoBarHidden") }
    }
}

// In view controller:
observation = UserDefaults.standard.observe(\.isInfoBarHidden, options: [.initial]) { [weak self] _, _ in
    self?.updateInfoBar()
}
```

## Takeaways
- The new scene lifecycle (`UIWindowScene` + `UISceneDelegate`) is the default for new iOS 13 projects; adopt it even without multi-window support to benefit from future-proofing and the new scene-level status bar / orientation APIs.
- Per-scene state must be keyed to `UISceneSession.persistentIdentifier` to avoid data conflicts between simultaneous scene instances sharing the same app container.
- The primary UX entry point for new windows is drag and drop; implement `NSUserActivity` as an item provider representation to enable dragging content into new scenes.
- Any UI state previously stored globally (in `UIApplication`, singletons, or `UserDefaults` without KVO) needs review: it may now need to either remain global (app-wide preference) or be scoped per-scene.

---
_Source: WWDC19 Session 212 page (abstract, chapter summaries, code samples, and resource links)._
