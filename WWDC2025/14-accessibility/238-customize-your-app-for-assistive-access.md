# Customize Your App for Assistive Access
**WWDC25 · Session 238** · [Watch](https://developer.apple.com/videos/play/wwdc2025/238/)

_Platforms:_ iOS 26, iPadOS 26

## Overview
Assistive Access is Apple's streamlined iOS/iPadOS experience for people with cognitive disabilities, introduced in iOS 17. In iOS 26, third-party apps can now provide a tailored Assistive Access experience using the new `AssistiveAccess` SwiftUI scene type. This session covers what Assistive Access is, how to create an Assistive Access scene (SwiftUI and UIKit), and the key design principles for building a high-quality experience for people with cognitive disabilities.

## Key Topics

### Assistive Access Overview
Assistive Access reduces cognitive load by providing apps with large controls, simplified interfaces, and visual alternatives to text. Built-in apps (Camera, Messages) set the design language. Apps not optimized for Assistive Access display in a reduced frame with a persistent back button. Two opt-in paths exist:
- **Full screen (`UISupportsFullScreenInAssistiveAccess`):** App fills the full screen unchanged — best for apps already designed for cognitive disabilities (e.g., AAC apps).
- **Assistive Access scene (new iOS 26):** Custom lightweight scene that automatically renders native SwiftUI controls in the Assistive Access style (large, grid/row layout). Best for most apps.

### Creating an AssistiveAccess Scene (SwiftUI)
Set `UISupportsAssistiveAccess = true` in Info.plist, then add an `AssistiveAccess { }` scene alongside the app's main `WindowGroup`. The scene provides a separate view hierarchy with only the essential features. Native controls inside the scene automatically adopt the Assistive Access visual style. Test with `#Preview(traits: .assistiveAccess)`.

### Creating an AssistiveAccess Scene (UIKit)
Declare the scene in a `UIHostingSceneDelegate` subclass via the `static var rootScene` property. Activate it by returning that delegate class from the AppDelegate's `application(_:configurationForConnecting:)` when the session role is `.windowAssistiveAccessApplication`.

### Design Principles for Assistive Access
1. **Distill to essentials:** Pick 1–2 core features. Fewer options reduce cognitive load.
2. **Prominent controls:** Avoid hidden gestures or nested UI; use large, visible controls.
3. **No timed interactions:** Remove or redesign anything that disappears or changes state on a timeout.
4. **Incremental flows:** Step-by-step guided navigation, not multi-option screens.
5. **Convey information in multiple ways:** Pair icons with labels; include navigation title icons.
6. **Avoid destructive actions:** Remove deletion features or confirm twice where unavoidable.

### Navigation Icons in Assistive Access
The `assistiveAccessNavigationIcon` modifier adds an icon alongside navigation titles when an Assistive Access scene is active. Pass a system image name or `Image` value. This provides a visual anchor in addition to the text title, improving recognition for users who rely on images over text.

## APIs & Frameworks

**SwiftUI (iOS 26, iPadOS 26)**
- **[NEW]** `AssistiveAccess { }` scene type — tailored Assistive Access view hierarchy
- **[NEW]** `.assistiveAccess` preview trait — test Assistive Access layout in Xcode Previews
- **[NEW]** `.assistiveAccessNavigationIcon(systemImage:)` modifier — icon alongside navigation title
- **[NEW]** `.assistiveAccessNavigationIcon(_:)` modifier — Image variant

**UIKit (iOS 26, iPadOS 26)**
- **[NEW]** `UIHostingSceneDelegate` — bridge UIKit lifecycle app to SwiftUI scenes
- **[NEW]** `UISceneSession.Role.windowAssistiveAccessApplication` — role for Assistive Access scene
- **[NEW]** `UIHostingSceneDelegate.rootScene` — static computed property declaring the scene

**Info.plist Keys**
- `UISupportsAssistiveAccess` — opt app into Assistive Access scene support (lists under Optimized Apps, launches full screen)
- `UISupportsFullScreenInAssistiveAccess` — alternative: display existing app full-screen unchanged

## Code Highlights
SwiftUI app with Assistive Access scene:
```swift
@main
struct MyApp: App {
    var body: some Scene {
        WindowGroup { ContentView() }
        AssistiveAccess {
            AssistiveAccessContentView()
        }
    }
}
```

Preview with Assistive Access trait:
```swift
#Preview(traits: .assistiveAccess) {
    AssistiveAccessContentView()
}
```

UIKit scene delegate declaration:
```swift
class AssistiveAccessSceneDelegate: UIHostingSceneDelegate {
    static var rootScene: some Scene {
        AssistiveAccess { AssistiveAccessContentView() }
    }
}
```

UIKit activation in AppDelegate:
```swift
func application(_ application: UIApplication, configurationForConnecting session: UISceneSession, options: UIScene.ConnectionOptions) -> UISceneConfiguration {
    let config = UISceneConfiguration(name: nil, sessionRole: session.role)
    if session.role == .windowAssistiveAccessApplication {
        config.delegateClass = AssistiveAccessSceneDelegate.self
    }
    return config
}
```

Add icon to navigation title:
```swift
.navigationTitle("Draw")
.assistiveAccessNavigationIcon(systemImage: "hand.draw.fill")
```

## Takeaways
- Set `UISupportsAssistiveAccess` in Info.plist and provide an `AssistiveAccess` scene to have your app listed under "Optimized Apps" in Assistive Access settings.
- Restrict the Assistive Access scene to 1–2 essential features — simplicity is the design goal, not feature parity.
- Use `assistiveAccessNavigationIcon` on every navigation-level view — visual icons alongside titles significantly improve comprehension for users with cognitive disabilities.
- Test with `#Preview(traits: .assistiveAccess)` to verify automatic control sizing and grid/row layout without running on a device.

---
_Source: WWDC25 Session 238 page (abstract, chapter summaries, code samples, and resource links)._
