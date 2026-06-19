# Qualities of a Great Mac Catalyst App
**WWDC21 · Session 10053** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10053/)

_Platforms:_ macOS Monterey 12, iOS 15, iPadOS 15

## Overview
This session provides a comprehensive guide to building a high-quality Mac Catalyst app — from initial migration through interface refinements to distribution. It starts with the foundational step of checking the Mac target in Xcode and choosing between the Scaled Interface and the Mac Idiom, explaining the trade-offs between quick compatibility and a pixel-perfect, native-feeling experience.

The session then dives into concrete UI improvements: understanding the suite of Mac button styles (system, pull-down, pop-up, and checkbox), handling pointer input correctly, organizing keyboard shortcuts via the menu builder, and managing multiple scene configurations for different window types. New macOS Monterey APIs such as `NSSharingServicePickerToolbarItem` with automatic activity items configuration and Continuity Camera support via `UIPasteConfiguration` are highlighted.

Distribution options round out the session: Mac App Store with Universal Purchase, TestFlight, App Notarization, and XCFramework for framework authors.

## Key Topics

### Migration and Interface Idiom
- Building for Mac Catalyst requires removing deprecated APIs: replace `OpenGLES` with Metal, `AddressBook` with Contacts framework, `UIWebView` with `WKWebView`.
- Two interface idiom choices: **Scaled** (easy, 77% scale) vs. **Mac Idiom** (100% scale, native AppKit controls, additional layout work needed).
- Monitor lifecycle events via scene delegate; `sceneDidEnterBackground` fires far less on Mac (only on minimize/close).

### Mac Button Types (Mac Idiom)
- **System button** (`.system`): default bordered push button.
- **Pull-down button**: set `button.menu` + `showsMenuAsPrimaryAction = true` — shows a menu of actions.
- **Pop-up button** (new in macOS Monterey via Mac Catalyst): same as pull-down but also set `changesSelectionAsPrimaryAction = true` — replaces `UIPickerView`.
- **Checkbox**: use `UISwitch` with a `title` set; the system automatically renders it as a checkbox in the Mac idiom.

### Pointer Input and Keyboard Navigation
- Support all functionality without pinch/rotate — add extra buttons for mouse-only users.
- Use `UIMenuBuilder` to expose all keyboard shortcuts in the main menu bar (also drives the iPad shortcuts overlay).
- Views that are targets of menu/key-command actions must return `true` for `canBecomeFirstResponder` and `canBecomeFocused`.
- Delegate responder chain actions via `target(forAction:withSender:)` instead of overriding `nextResponder`.

### Scene Management and State Restoration
- Define multiple scene configurations in `Info.plist` under `Application Scene Manifest` → `Application Session Role`.
- Request new scenes with `UIApplication.shared.requestSceneSessionActivation(_:userActivity:options:errorHandler:)`.
- Route scene types in `application(_:configurationForConnecting:options:)` by inspecting `userActivity.activityType`.
- Support state restoration by implementing `stateRestorationActivity(for:)` in the scene delegate; fall back to `session.stateRestorationActivity` in `scene(_:willConnectTo:options:)`.

### Sharing and Continuity Camera
- `NSSharingServicePickerToolbarItem` in macOS Monterey automatically reads `activityItemsConfiguration` from the root view controller.
- Return `UIActivityItemsConfiguration` from any view's `activityItemsConfiguration` property + add `UIContextMenuInteraction` to get automatic Copy and Share menu items on both Mac and iPad.
- Continuity Camera: return `UIPasteConfiguration(forAcceptingClass: UIImage.self)` from `pasteConfiguration` and add a `UIContextMenuInteraction` to any view.

## APIs & Frameworks

- `UIButton.Configuration` / `UIButton(type: .system)` — Mac-idiomatic push button
- `UIButton.menu` property — assigned `UIMenu` for pull-down/pop-up behavior
- `UIButton.showsMenuAsPrimaryAction` — enables pull-down button behavior
- `UIButton.changesSelectionAsPrimaryAction` **[NEW]** — enables pop-up button behavior (macOS Monterey)
- `UISwitch.title` property — required for checkbox rendering in Mac idiom
- `UISwitch.style` (read-only) — returns `.checkbox` or `.switch`
- `UISwitch.preferredStyle` — set to `.automatic`, `.checkbox`, or `.switch`
- `UIMenuBuilder` — organizes key commands and menu items for menu bar
- `UIResponder.canBecomeFirstResponder` / `canBecomeFocused`
- `UIResponder.target(forAction:withSender:)` — action delegation without breaking responder chain
- `UIApplication.shared.requestSceneSessionActivation(_:userActivity:options:errorHandler:)`
- `UIApplicationDelegate.application(_:configurationForConnecting:options:)` — scene routing
- `UISceneConfiguration(name:sessionRole:)`
- `UIWindowSceneDelegate.scene(_:willConnectTo:options:)`
- `UIWindowSceneDelegate.stateRestorationActivity(for:)` — state restoration
- `UIScene.ConnectionOptions.userActivities` — user activities passed at connection time
- `UISceneSession.stateRestorationActivity`
- `NSUserActivity` — scene activation and state restoration carrier
- `NSSharingServicePickerToolbarItem` **[NEW in Monterey]** — toolbar share button with automatic activity items
- `UIActivityItemsConfiguration` — declares shareable content
- `UIActivityItemsConfigurationReading` protocol
- `UIViewController.activityItemsConfiguration` property
- `UIView.activityItemsConfiguration` property
- `UIContextMenuInteraction` — enables context menus (and sharing/copy integration)
- `UIPasteConfiguration` — enables Paste action and Continuity Camera
- `UIResponder.paste(itemProviders:)` — receives pasted/dropped items
- `NSItemProvider.loadObject(ofClass:)` — async item loading
- `Metal` framework (replacement for OpenGLES)
- `Contacts` framework (replacement for AddressBook)
- `WKWebView` (replacement for UIWebView)

## Code Highlights

Creating a pop-up button (new in macOS Monterey):
```swift
button.menu = UIMenu(...)
button.showsMenuAsPrimaryAction = true
button.changesSelectionAsPrimaryAction = true
```

Requesting a new scene for a detail view:
```swift
let userActivity = NSUserActivity(activityType: viewDetailActivityType)
userActivity.userInfo = [itemIDKey: selectedItem.itemID]
UIApplication.shared.requestSceneSessionActivation(nil,
    userActivity: userActivity, options: nil, errorHandler: { _ in })
```

Supporting Continuity Camera on any view:
```swift
override var pasteConfiguration: UIPasteConfiguration? {
    UIPasteConfiguration(forAcceptingClass: UIImage.self)
}
override func paste(itemProviders: [NSItemProvider]) {
    for provider in itemProviders where provider.canLoadObject(ofClass: UIImage.self) {
        // load and insert image
    }
}
```

## Takeaways
- Choose Mac Idiom over Scaled for a first-class Mac experience, but budget time for layout adjustments as control metrics change.
- Implement pull-down and pop-up buttons to replace iOS-specific controls like `UIPickerView` with Mac-native equivalents.
- Manage multiple window types with separate scene configurations in `Info.plist` and route them in `application(_:configurationForConnecting:options:)`.
- Use `UIActivityItemsConfiguration` and `UIPasteConfiguration` to get sharing, copy, and Continuity Camera for free on both Mac and iPad.

---
_Source: WWDC21 Session 10053 page (abstract, chapter summaries, code samples, and resource links)._
