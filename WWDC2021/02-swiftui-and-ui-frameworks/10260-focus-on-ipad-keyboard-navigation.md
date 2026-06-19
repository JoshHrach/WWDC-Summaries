# Focus on iPad Keyboard Navigation
**WWDC21 · Session 10260** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10260/)

_Platforms:_ iPadOS 15, macOS Monterey 12 (Mac Catalyst)

## Overview
iPadOS 15 and Mac Catalyst introduce system-level keyboard navigation using the existing tvOS focus system. Tab moves between significant UI areas (focus groups), arrow keys navigate within an area, and Return/Space activates items. This behavior is automatic for text fields, text views, and sidebars when compiled against the iOS 15 SDK; developers must explicitly opt in collection and table views.

The session covers making views focusable, customizing focus appearance (halo effects, highlight), the new focus group system (tab loop customization), sidebar selection-follows-focus behavior, and the relationship between keyboard focus and the UIKit responder chain.

## Key Topics

**Making Views Focusable**
Override `canBecomeFocused` to return `true` on any `UIFocusItem`. For collection/table views, set `allowsFocus = true` (default `true` in sidebars) or implement `collectionView(_:canFocusItemAt:)` for per-cell control. Not every element should be focusable — focus is for text input, lists, and collection views.

**Focus Appearance**
Two standard styles: `UIFocusHaloEffect` (halo outline, default on Mac Catalyst, opt-in on iPadOS) and a tint-color highlight (automatic with background/content cell configurations from iOS 14). `UIFocusHaloEffect` supports rounded rects and has `referenceView` (controls z-order) and `containerView` (controls superview for the halo) properties. Custom appearance: set `focusEffect = nil` and override `didUpdateFocus(in:withAnimationCoordinator:)`.

**Focus Groups and Tab Loop**
Focus groups define the areas navigated by Tab. Groups are inferred from scroll views, text fields, and text views by default; custom groups are declared via `focusGroupIdentifier` on any view/view controller. Items sharing an identifier (or inheriting it) belong to the same group. The tab loop moves between group primary items. Primary items are determined by `focusGroupPriority` (`.ignored`, `.previouslyFocused`, `.prioritized`, `.currentlyFocused`). Container views (e.g., a sidebar containing a search field and list) should declare their own `focusGroupIdentifier` to group children correctly.

**Selection Follows Focus**
`selectionFollowsFocus` property on `UICollectionView`/`UITableView` synchronizes selection and focus. Implement `collectionView(_:selectionFollowsFocusForItemAt:)` to override per cell (e.g., disable for cells that trigger alerts).

**Responder Chain Integration**
UIKit keeps the first responder synchronized with the focused item. When focus changes, UIKit attempts to make the focused item (or an ancestor) first responder. Key events are delivered to the focused item and propagate up the responder chain. Avoid calling `becomeFirstResponder()` in response to focus updates, as it disrupts user navigation. If existing `UIKeyCommand` handlers use Tab/Arrow keys, they will stop working when compiling with iOS 15 SDK — either remap them or set `wantsPriorityOverSystemBehavior = true`.

**Debugging Tools**
- `UIFocusDebugger.checkFocusability(for:)` — explains why an item is not focusable
- `UIFocusDebugger.checkFocusGroupTree(for:)` — prints the focus group hierarchy
- Launch argument `-UIFocusLoopDebuggerEnabled YES` — live overlay showing tab loop order (Option key) and focus group boundaries (Option+Control)

## APIs & Frameworks

- **UIKit** — focus system, iPadOS 15 / Mac Catalyst
- `UIFocusItem` protocol — `var canBecomeFocused: Bool` (override to return `true`)
- `UIFocusEnvironment` protocol — defines focus hierarchy
- `UICollectionView` / `UITableView`:
  - `var allowsFocus: Bool` **[NEW]** — makes all cells focusable
  - `var selectionFollowsFocus: Bool` **[NEW]** — syncs selection with focus
- `UICollectionViewDelegate` / `UITableViewDelegate`:
  - `func collectionView(_:canFocusItemAt:) -> Bool` — per-cell focusability
  - `func collectionView(_:selectionFollowsFocusForItemAt:) -> Bool` **[NEW]** — per-cell override
- `UIFocusEffect` — base class for focus appearance
- `UIFocusHaloEffect` **[NEW]** — halo outline focus effect
  - `init()` — inferred shape
  - `init(roundedRect:cornerRadius:curve:)` **[NEW]**
  - `var referenceView: UIView?` — controls z-order of halo in hierarchy
  - `var containerView: UIView?` — controls superview of halo
- `UIView.focusEffect: UIFocusEffect?` **[NEW]** — assign halo or nil for custom
- `UIFocusUpdateContext` — in `didUpdateFocus(in:withAnimationCoordinator:)`
  - `var nextFocusedItem: UIFocusItem?`
  - `var previouslyFocusedItem: UIFocusItem?`
- `UIView.focusGroupIdentifier: String?` **[NEW]** — declares/assigns a focus group
- `UIViewController.focusGroupIdentifier: String?` **[NEW]**
- `UIFocusGroupPriority` **[NEW]** — struct with predefined values
  - `.ignored` (0)
  - `.previouslyFocused` (1000)
  - `.prioritized` (2000)
  - `.currentlyFocused` (Int.max)
- `UIView.focusGroupPriority: UIFocusGroupPriority` **[NEW]** — item's priority within its group
- `UIKeyCommand.wantsPriorityOverSystemBehavior: Bool` **[NEW]** — opt out of Tab/Arrow override
- `UIFocusDebugger` — static debug helper (LLDB)
  - `checkFocusability(for:)`
  - `checkFocusGroupTree(for:)`
- Launch argument: `-UIFocusLoopDebuggerEnabled YES` **[NEW]**

## Code Highlights

Make a collection view focusable:
```swift
collectionView.allowsFocus = true
```

Custom rounded halo with reference and container views:
```swift
let effect = UIFocusHaloEffect(roundedRect: bounds, cornerRadius: layer.cornerRadius, curve: .continuous)
effect.referenceView = imageView
effect.containerView = scrollView
self.focusEffect = effect
```

Custom focus appearance without halo:
```swift
init(frame: CGRect) {
    super.init(frame: frame)
    self.focusEffect = nil  // disable system styling
}
override func didUpdateFocus(in context: UIFocusUpdateContext, withAnimationCoordinator: UIFocusAnimationCoordinator) {
    if context.nextFocusedItem === self { /* apply focused styling */ }
    else if context.previouslyFocusedItem === self { /* restore normal styling */ }
}
```

Selection follows focus (with per-cell override):
```swift
collectionView.selectionFollowsFocus = true
func collectionView(_ cv: UICollectionView, selectionFollowsFocusForItemAt indexPath: IndexPath) -> Bool {
    return action(for: indexPath).type != .showAlert
}
```

Focus group for sidebar container:
```swift
sidebarContainerView.focusGroupIdentifier = "com.myapp.groups.sidebar"
```

Elevated focus group priority:
```swift
cell.focusGroupPriority = .previouslyFocused + 10
```

## Takeaways

- Keyboard focus navigation is automatic for text fields, sidebars, and text views in iOS 15; opt in collection/table views with `allowsFocus = true` for a complete keyboard-navigable app.
- Focus groups (declared via `focusGroupIdentifier`) define the areas Tab navigates between — container views like sidebars need their own identifier to group their children correctly in the tab loop.
- `UIFocusHaloEffect` is the standard focus indicator; for cells, the tint-color highlight is preferred and comes for free with iOS 14 background/content cell configurations.
- Key commands using Tab or Arrow keys conflict with system keyboard navigation in iOS 15 and must be remapped or marked with `wantsPriorityOverSystemBehavior = true`.

---
_Source: WWDC21 Session 10260 page (abstract, chapter summaries, code samples, and resource links)._
