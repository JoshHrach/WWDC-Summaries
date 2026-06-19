# Modernizing Your UI for iOS 13
**WWDC19 · Session 224** · [Watch](https://developer.apple.com/videos/play/wwdc2019/224/)

_Platforms:_ iOS 13, iPadOS 13

## Overview
This session is the comprehensive iOS 13 UIKit modernization guide, covering six areas: flexible UI requirements, bar appearance, sheet presentations, search improvements, new productivity gestures, and the new context menu API. It is both a "what's required" compliance briefing (Launch Storyboards mandatory by April 2020, apps linked against iOS 13 SDK must support any screen size) and a deep dive into the major new APIs that ship in iOS 13.

The bar appearance system is completely redesigned around `UIBarAppearance` and its subclasses, enabling per-scroll-position, per-bar, and per-navigation-item appearance customization. Sheet presentations (`pageSheet`/`formSheet`) become the system default for most view controller presentations and gain interactive pull-to-dismiss with delegate hooks for the modal flow. Search gets search tokens (`UISearchToken`), an exposed `UISearchTextField`, and control over scope bar and cancel button visibility. Gestures add two-finger multi-select in table/collection views, three-finger undo/redo/copy/paste, and a new `UITextInteraction` for custom text views. Peek and Pop is deprecated in favor of `UIContextMenuInteraction`, which works across iPhone, iPad, and Mac.

## Key Topics
- **Flexible UI requirements** — Launch Storyboard required (no Launch Images alone) by April 2020; apps linked to iOS 13 SDK displayed at native full-screen on all hardware; iPad apps must support Split Screen multitasking
- **Bar appearance** — transparent-to-opaque scroll-edge transition is automatic when linked to iOS 13; new `UIBarAppearance` subclasses; `scrollEdgeAppearance` / `standardAppearance` / `compactAppearance`; per-`UINavigationItem` appearance
- **Sheet presentations** — `UIModalPresentationStyle.automatic` (new default); `pageSheet` / `formSheet` stay as sheets in compact width; `isModalInPresentation`; delegate callbacks for the modal flow
- **Search** — `UISearchTextField` exposed as public property; `UISearchToken` for tokenized queries with copy/paste/drag-drop support; `showsSearchResultsController` and `automaticallyShowsSearchResultsController`
- **Gestures** — two-finger pan for multiple selection in table/collection views (opt-in); three-finger swipe for undo/redo, three-finger pinch for copy/paste (automatic via `UndoManager`); `UITextInteraction` for system text gestures in custom text views
- **Context menus** — `UIContextMenuInteraction` replaces Peek and Pop (deprecated); `UIMenu` / `UIAction` hierarchical menu system; `UITargetedPreview` for custom animations; Table/Collection View delegate convenience API

## APIs & Frameworks

### Bar Appearance (all NEW)
- `UIBarAppearance` — base class; `configureWithOpaqueBackground()`, `configureWithTransparentBackground()`, `configureWithDefaultBackground()`
- `UINavigationBarAppearance` — `titleTextAttributes`, `largeTitleTextAttributes`, `buttonAppearance`, `doneButtonAppearance`
- `UIToolbarAppearance`
- `UITabBarAppearance` — `stackedLayoutAppearance`, `inlineLayoutAppearance`, `compactInlineLayoutAppearance` (each a `UITabBarItemAppearance`)
- `UINavigationBar.standardAppearance`, `.compactAppearance`, `.scrollEdgeAppearance` **[NEW]**
- `UINavigationItem.standardAppearance`, `.compactAppearance`, `.scrollEdgeAppearance` **[NEW]** — per-item override

### Sheet Presentations (all NEW unless noted)
- `UIModalPresentationStyle.automatic` **[NEW]** — default style; resolves to `pageSheet` for custom VCs, or platform-appropriate style for system VCs
- `pageSheet` / `formSheet` — no longer adapt to full screen in compact width; new layout with readable width, stacked appearance on iPad
- `UIViewController.isModalInPresentation: Bool` **[NEW]** — prevents interactive dismissal; causes rubber-band effect
- `UIAdaptivePresentationControllerDelegate` — new methods **[NEW]**:
  - `presentationControllerShouldDismiss(_:) -> Bool`
  - `presentationControllerWillDismiss(_:)`
  - `presentationControllerDidDismiss(_:)`
  - `presentationControllerDidAttemptToDismiss(_:)` — called when user pulls down while `isModalInPresentation == true`
- Appearance callbacks: presenting VC does NOT receive `viewWillDisappear`/`viewDidDisappear` during sheet presentation (view stays in hierarchy)

### Search (all NEW)
- `UISearchBar.searchTextField: UISearchTextField` **[NEW]** — public access to the internal `UITextField` subclass for full styling
- `UISearchController.showsSearchResultsController: Bool` **[NEW]**
- `UISearchController.automaticallyShowsSearchResultsController: Bool` **[NEW]**
- `UISearchController.showsScopeBar: Bool` **[NEW]**
- `UISearchController.automaticallyShowsScopeBar: Bool` **[NEW]**
- `UISearchToken` **[NEW]** — represents a complex query as a tappable chip; supports copy/paste/drag-drop
  - `init(icon: UIImage?, text: String)`
  - `representedObject: Any?` — attach app data to the token
- `UISearchTextField` **[NEW]** — `UITextField` subclass
  - `tokens: [UISearchToken]`
  - `replaceTextualPortion(of:with:at:)` — convert selected text to a token
  - `textualRange: UITextRange` — text range excluding token positions

### Gestures (all NEW)
- `UITextInteraction` **[NEW]** — adds system text selection gestures to custom text views
  - `init(for mode: UITextInteractionMode)` — `.editable` or `.nonEditable`
  - `textInput: UITextInput?` — the view implementing `UITextInput`
  - `UIResponder.editingInteractionConfiguration: UIEditingInteractionConfiguration` **[NEW]** — return `.none` to opt out of system three-finger gestures
- `UITableView` / `UICollectionView` multiple selection gesture (opt-in):
  - `UITableViewDelegate.tableView(_:shouldBeginMultipleSelectionInteractionAt:) -> Bool` **[NEW]**
  - `UITableViewDelegate.tableView(_:didBeginMultipleSelectionInteractionAt:)` **[NEW]**
  - `UICollectionViewDelegate` equivalents **[NEW]**
- Three-finger undo (swipe left), redo (swipe right), copy (pinch in), paste (pinch out) — free if using `UndoManager` / `NSUndoManager`

### Context Menus (all NEW)
- `UIContextMenuInteraction` — replaces `UIViewControllerPreviewing` (Peek and Pop, now deprecated)
  - `init(delegate: UIContextMenuInteractionDelegate)`
  - Gesture adapts: 3D touch → Haptic Touch → long press → right click (Mac)
  - Coordinates with `UIDragInteraction` for fluid drag transition
- `UIContextMenuInteractionDelegate`:
  - `contextMenuInteraction(_:configurationForMenuAtLocation:) -> UIContextMenuConfiguration?` — required
- `UIContextMenuConfiguration` **[NEW]**
  - `init(identifier:previewProvider:actionProvider:)`
  - `previewProvider: (() -> UIViewController?)` — deferred preview view controller creation
  - `actionProvider: (([UIMenuElement]) -> UIMenu?)` — receives suggested system actions
- `UIMenu` **[NEW]** — composable menu node
  - `init(title:image:identifier:options:children:)`
  - `UIMenuOptions`: `.destructive`, `.displayInline`
- `UIAction` **[NEW]** — leaf menu item
  - `init(title:image:identifier:discoverabilityTitle:attributes:state:handler:)`
  - `UIMenuElementAttributes`: `.destructive`, `.disabled`, `.hidden`
- `UITargetedPreview` **[NEW]** — generalizes `UITargetedDragPreview`; configure shape, background color, target view for menu animation
- Table/Collection View convenience:
  - `UITableViewDelegate.tableView(_:contextMenuConfigurationForRowAt:point:) -> UIContextMenuConfiguration?` **[NEW]**
  - `UICollectionViewDelegate` equivalent **[NEW]**
- **`UIViewControllerPreviewing` (Peek and Pop)** — **[DEPRECATED iOS 13]**

## Code Highlights

```swift
// Bar appearance: transparent scroll edge, opaque when scrolled
let appearance = UINavigationBarAppearance()
appearance.configureWithOpaqueBackground()
appearance.titleTextAttributes = [.foregroundColor: UIColor.label]

navigationBar.standardAppearance = appearance
navigationBar.scrollEdgeAppearance = UINavigationBarAppearance()  // transparent by default
```

```swift
// Per-navigation-item color tinting (e.g., Reminders list color)
let tinted = navigationController!.navigationBar.standardAppearance.copy()
tinted.titleTextAttributes = [.foregroundColor: listColor]
navigationItem.standardAppearance = tinted
```

```swift
// Sheet presentation: prevent accidental dismissal, show action sheet on attempt
class EditViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        presentationController?.delegate = self
    }
    func userStartedEditing() {
        isModalInPresentation = true  // lock the sheet
    }
}
extension EditViewController: UIAdaptivePresentationControllerDelegate {
    func presentationControllerDidAttemptToDismiss(_ pc: UIPresentationController) {
        // Show "Discard Changes?" action sheet here
    }
}
```

```swift
// UIContextMenuInteraction setup
let interaction = UIContextMenuInteraction(delegate: self)
photoView.addInteraction(interaction)

// Delegate
func contextMenuInteraction(_ interaction: UIContextMenuInteraction,
                            configurationForMenuAtLocation location: CGPoint)
    -> UIContextMenuConfiguration? {
    return UIContextMenuConfiguration(identifier: nil, previewProvider: nil) { _ in
        let share  = UIAction(title: "Share",  image: UIImage(systemName: "square.and.arrow.up")) { _ in }
        let delete = UIAction(title: "Delete", image: UIImage(systemName: "trash"),
                              attributes: .destructive) { _ in }
        return UIMenu(title: "", children: [share, delete])
    }
}
```

```swift
// UISearchToken creation
func searchController(_ sc: UISearchController, didSelectSuggestion suggestion: String) {
    let token = UISearchToken(icon: UIImage(systemName: "tag"), text: suggestion)
    token.representedObject = suggestion
    let textField = sc.searchBar.searchTextField
    let range = textField.textualRange   // exclude existing tokens
    textField.replaceTextualPortion(of: range, with: token, at: textField.tokens.endIndex)
}
```

## Takeaways
- Add a Launch Storyboard immediately if you haven't — it is required for App Store submission starting April 2020; apps linked to iOS 13 SDK will display full-screen with no letterboxing and must handle all sizes including iPad Split View.
- The new `UIBarAppearance` system is the only supported way to customize bar appearance in iOS 13; do not manipulate `barTintColor` or `isTranslucent` directly — use the appearance objects and assign them to `standardAppearance` / `scrollEdgeAppearance`.
- Sheet presentations are the new default for all modals; set `isModalInPresentation = true` when the user has unsaved state, and implement `presentationControllerDidAttemptToDismiss` to handle the rubber-band pull-down with an action sheet — this is the expected iOS 13 modal flow.
- Replace all `UIViewControllerPreviewing` (Peek and Pop) implementations with `UIContextMenuInteraction` — it handles 3D Touch, Haptic Touch, long press, and right click (Mac Catalyst) automatically, and integrates with Drag and Drop.
- `UISearchToken` is ready to use as-is for the same tokenized search UI found in Photos and Mail; pair it with `UISearchTextField.textualRange` for correct programmatic selection around tokens.

---
_Source: WWDC19 Session 224 page (full transcript, abstract, and resource links)._
