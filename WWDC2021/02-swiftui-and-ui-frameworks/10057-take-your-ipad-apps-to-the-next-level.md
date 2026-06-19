# Take Your iPad Apps to the Next Level
**WWDC21 · Session 10057** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10057/)

_Platforms:_ iPadOS 15, macOS Monterey 12

## Overview
This session covers three major areas of improvement for iPad apps in iPadOS 15: multitasking and scene enhancements, keyboard shortcuts and the new shortcut discovery interface, and pointer interaction updates. Together these features help developers build iPad apps that feel powerful, productive, and polished.

Scene presentation gains new styles — prominent and standard — giving developers explicit control over how new scenes appear relative to existing ones. Prominent scenes appear modally over the current workspace, ideal for opening focused documents, while standard scenes appear side by side for full-featured multitasking. State restoration also receives a dedicated callback and new interaction state properties to make saving and restoring scene state far simpler.

The keyboard shortcut system gains a new main menu structure on iPadOS 15 (mirroring Mac Catalyst), displayed when the Command key is held. Developers customize it via `UIMenuBuilder` in `AppDelegate.buildMenu(with:)`. Pointer interactions gain band selection for multi-item selection, accessories that communicate context visually, and latching axes for smooth drag interactions.

## Key Topics

### Multitasking and Scene Presentation
- New `UIWindowScenePresentationStyle` values: `.prominent` (modal over workspace), `.standard` (side-by-side), and `.automatic`
- `UIWindowScene.ActivationAction`: a `UIAction` subclass for "Open in New Window" menus and buttons with automatic hiding on iPhone
- `UIWindowScene.ActivationInteraction`: pinch-out gesture for opening scenes from custom views
- New `UICollectionViewDelegate` method `sceneActivationConfigurationForItemAt:` for collection view pinch-out support
- `UIWindowScene.ActivationConfiguration`: wraps `NSUserActivity`, optional `UITargetedPreview`, and scene request options
- `QLPreviewSceneActivationConfiguration`: system-managed Quick Look preview scenes requiring no scene delegate
- New `UISceneDelegate` method `scene(_:restoreInteractionState:)` called after storyboard load but before first foreground transition
- `UITextField.interactionState` and `UITextView.interactionState` **[NEW]**: single object capturing scroll position, cursor position, and first responder status
- `UIScene.extendStateRestoration()` and `UIScene.completeStateRestoration()` for async state restoration

### Keyboard Shortcuts and the Main Menu
- iPadOS 15 main menu system mirrors Mac Catalyst's `UIMenuBuilder` API
- `AppDelegate.buildMenu(with:)` override for customizing main menu
- `UIMenuBuilder.insertChild(_:atStartOfMenu:)` and `insertSibling(_:afterMenu:)` for menu organization
- `UIMenu.Identifier` constants for referencing system menus (e.g., `.file`, `.view`)
- Commands identified by action selectors; unique identifiers required; duplicate shortcuts logged as errors
- `UIKeyCommand.discoverabilityTitle` preferred by iPadOS over regular title in shortcut overlay
- `UIResponder.canPerformAction(_:withSender:)` for controlling command availability
- `UIResponder.validate(_:)` for dynamically updating command title/state
- `UIKeyCommand.allowsAutomaticMirroring` **[NEW]**: opt out of RTL mirroring for specific shortcuts
- Keyboard shortcut localization: system automatically remaps shortcuts for different keyboard layouts in iPadOS 15 and macOS 12
- Focus system integration: responder traversal now begins at focused item when keyboard navigation is adopted

### Pointer Enhancements
- Band selection built into `UICollectionView` for non-list layouts via existing `shouldBeginMultipleSelectionInteraction` API
- `UIBandSelectionInteraction` **[NEW]**: custom band selection for non-collection view content
- `UIBandSelectionInteraction.initialModifierFlags`: keys held at drag start for Shift/Command modifier behavior
- `UIPointerAccessory` **[NEW]**: secondary shapes (arrows, custom) combined with primary pointer
- `UIPointerAccessory.Position`: offset and angle from pointer midpoint; predefined constants (`.topRight`, `.left`, `.right`, etc.)
- `UIPointerStyle.accessories` **[NEW]**: array of accessories applied to any pointer style
- `UIPointerStyle.system()` **[NEW]**: default system pointer with accessories
- `UIPointerRegion.latchingAxes` **[NEW]**: pointer effect follows dragging along specified axes

## APIs & Frameworks

- `UIKit`
- `UIWindowScene.ActivationAction` **[NEW]**
- `UIWindowScene.ActivationConfiguration` **[NEW]**
- `UIWindowScene.ActivationInteraction` **[NEW]**
- `UIWindowScenePresentationStyle` **[NEW]** (`.prominent`, `.standard`, `.automatic`)
- `UIWindowScene.ActivationAction(alternate:_:)` **[NEW]**
- `QLPreviewSceneActivationConfiguration` **[NEW]**
- `UISceneDelegate.scene(_:restoreInteractionState:)` **[NEW]**
- `UITextField.interactionState` **[NEW]**
- `UITextView.interactionState` **[NEW]**
- `UIScene.extendStateRestoration()` **[NEW]**
- `UIScene.completeStateRestoration()` **[NEW]**
- `UICollectionViewDelegate.collectionView(_:sceneActivationConfigurationForItemAt:point:)` **[NEW]**
- `UIMenuBuilder`
- `UIMenuBuilder.insertChild(_:atStartOfMenu:)`
- `UIMenuBuilder.insertSibling(_:afterMenu:)`
- `UIMenu.Identifier` (`.file`, `.view`, `.edit`, etc.)
- `UIKeyCommand`
- `UIKeyCommand.discoverabilityTitle`
- `UIKeyCommand.allowsAutomaticMirroring` **[NEW]**
- `UIResponder.canPerformAction(_:withSender:)`
- `UIResponder.validate(_:)`
- `UIBandSelectionInteraction` **[NEW]**
- `UIBandSelectionInteraction.initialModifierFlags` **[NEW]**
- `UIBandSelectionInteraction.selectionRect` **[NEW]**
- `UIBandSelectionInteraction.state` **[NEW]**
- `UIPointerAccessory` **[NEW]**
- `UIPointerAccessory.Position` **[NEW]**
- `UIPointerStyle.accessories` **[NEW]**
- `UIPointerStyle.system()` **[NEW]**
- `UIPointerRegion.latchingAxes` **[NEW]**
- `UITargetedPreview`
- `NSUserActivity`
- `UISceneConfiguration`
- `UISceneSession`

## Code Highlights

Opening content in a new scene via menu action:
```swift
let newSceneAction = UIWindowScene.ActivationAction({ _ in
    let userActivity = NSUserActivity(activityType: "com.example.MyActivity")
    return UIWindowScene.ActivationConfiguration(userActivity: userActivity)
})
```

Restoring scene state with the new dedicated callback:
```swift
func scene(_ scene: UIScene, restoreInteractionState stateRestorationActivity: NSUserActivity) {
    // Restore content first, then interaction state
    viewController.textField.text = userInfo["content"] as? String
    viewController.textField.interactionState = userInfo["interactionState"]
}
```

Adding pointer accessories to a lift effect:
```swift
let style = UIPointerStyle(effect: .lift(UITargetedPreview(view: self)))
style.accessories = [.arrow(.left), .arrow(.right)]
```

## Takeaways

- Adopt `UIWindowScene.ActivationAction` and `ActivationInteraction` to enable natural "Open in New Window" behaviors on iPad, with automatic graceful degradation on iPhone.
- Move all keyboard shortcut declarations into `buildMenu(with:)` using `UIMenuBuilder` to appear properly in the new iPadOS 15 shortcut overlay.
- Use `UITextField.interactionState` and the new `scene(_:restoreInteractionState:)` callback for much simpler and more accurate state restoration.
- Add `UIPointerAccessory` to communicate drag direction, operation availability, and other contextual hints without replacing the pointer style.

---
_Source: WWDC21 Session 10057 page (abstract, chapter summaries, code samples, and resource links)._
