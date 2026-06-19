# Build with iOS Pickers, Menus and Actions
**WWDC20 · Session 10052** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10052/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11 (Catalyst)

## Overview
This session surveys the new and updated UIKit controls in iOS 14: updated appearances for `UISlider`, `UIProgressView`, `UIActivityIndicatorView`, and pull-to-refresh; a fully new `UIColorPickerViewController`; compact and inline styles for `UIDatePicker`; an expanded `UIPageControl` with unlimited pages and custom indicator images; and significantly enhanced `UIMenu` support on `UIButton`, `UIBarButtonItem`, and `UISegmentedControl`. The session concludes with improvements to `UIAction` that allow segmented controls and bar button items to be created declaratively with inline handlers, eliminating switch-statement-based segment handling.

Two engineers cover complementary areas: Eric Dudiak covers control appearance updates, `UIColorPickerViewController`, `UIDatePicker`, and `UIPageControl`; David Duncan covers menus (`UIDeferredMenuElement`, `updateVisibleMenu`, navigation Back button menus, `UIContextMenuInteraction.appearance`) and `UIAction` improvements for `UIBarButtonItem`, `UIButton`, and `UISegmentedControl`.

All new control styles and `UIAction`-based initializers are also available in SwiftUI counterparts on the same platforms.

## Key Topics

**Control Appearance Refresh**
`UISlider` and `UIProgressView` now have a thicker track, matching macOS appearance. `UISlider` gains macOS-style tap-to-set on the track. `UIActivityIndicatorView` has fewer petals and consistent sizing. Pull-to-refresh is updated to match. In optimized Catalyst apps, all these controls adopt native macOS appearance; some customization APIs are ignored in that context.

**UIPageControl — Unlimited Pages + Custom Images**
`UIPageControl` now supports unlimited pages in a fixed size: when pages exceed available space, the control allows scrubbing/scrolling. Custom indicator images can be set globally (`preferredIndicatorImage`) or per-page (`setIndicatorImage(_:forPage:)`). `backgroundStyle` (`.automatic`, `.minimal`, `.prominent`) controls background visibility. The API surface for page navigation and `currentPage` is unchanged.

**UIColorPickerViewController (New)**
A new color picker view controller for iOS 14 and iPadOS, matching the macOS color panel. Supports: grid color picker, spectrum gradient, manual RGB/hex input, eyedropper tool (can sample color anywhere on screen on iPadOS multi-app), and favoriting colors across apps. Presented as a sheet or popover. Set `selectedColor` before presenting; read it in the delegate. On macOS Catalyst, uses the native macOS color picker panel.

**UIDatePicker — Compact and Inline Styles (New)**
Two new `preferredDatePickerStyle` values: `.compact` and `.inline`.
- `.compact`: shows date/time as tappable field labels; tapping date presents a modal calendar; tapping time uses a keypad. Available since iOS 13.4 SDK (for Catalyst). Sized like a `UILabel`.
- `.inline`: shows the full calendar/time picker inline in the view without a modal, for contexts where date selection is the primary UI purpose.
All existing `UIDatePicker` API (`date`, `minimumDate`, `maximumDate`, `calendar`, `locale`, `datePickerMode`, `addTarget`) is unchanged. No code migration needed beyond setting the style property.

**Menus on UIButton and UIBarButtonItem (New)**
`UIButton.menu` — assign a `UIMenu` to a button; shows on long press by default. Setting `showsMenuAsPrimaryAction = true` presents the menu on touch down (no long press required). `UIBarButtonItem.menu` — providing a menu without a primary action causes the menu to present on touch down.

**Navigation Bar Back Button Menus (Automatic)**
The navigation bar Back button in iOS 14 automatically shows a menu for jumping to any item in the navigation stack. No code required; ensure `navigationItem.backButtonTitle` or `navigationItem.title` is set to provide readable menu titles.

**UIDeferredMenuElement (New)**
Adds asynchronously-loaded items to a `UIMenu`. The system displays a loading indicator while the items are fetched, then caches them for subsequent displays of the same menu. Useful for menus whose items are expensive to compute.

**UIContextMenuInteraction.updateVisibleMenu (New)**
Mutates the currently-displayed menu without dismissing it. The update block receives a copy of the current menu (now mutable — children are no longer forced immutable) and returns the updated version.

**UIAction for UIBarButtonItem and UISegmentedControl**
- `UIBarButtonItem` gains new initializers: `init(systemItem:primaryAction:menu:)`, `init(image:menu:)`, `init(primaryAction:)`, and factory methods `.fixedSpace(width:)`, `.flexibleSpace()`.
- `UISegmentedControl` gains `init(frame:actions:)` — each segment is defined by a `UIAction`; that action's handler is called only when its segment is selected. Eliminates the need for `addTarget`/switch-statement event handling.
- `UIButton` gains `init(type:primaryAction:)` — the primary action's title and image configure the button; no separate `addTarget` needed for the primary tap.

## APIs & Frameworks

### UIKit — Controls (Updated Appearance)
- `UISlider` — thicker track; tap-to-set value on macOS behavior adopted
- `UIProgressView` — thicker track matching macOS
- `UIActivityIndicatorView` — fewer petals, consistent sizing; use `.medium`/`.large` styles; set custom color via `color` property
- `UIRefreshControl` — updated animation appearance

### UIKit — UIPageControl **[Updated]**
- `UIPageControl.backgroundStyle` **[NEW]** — `.automatic`, `.minimal`, `.prominent`
- `UIPageControl.preferredIndicatorImage` **[NEW]** — default image for all page indicators
- `UIPageControl.setIndicatorImage(_:forPage:)` **[NEW]** — per-page custom indicator image
- `UIPageControl.interactionState` **[NEW]** — `.none`, `.discrete`, `.continuous`
- Unlimited pages with built-in scrubbing when count exceeds display width

### UIKit — UIColorPickerViewController **[NEW]**
- `UIColorPickerViewController` **[NEW]** — full-featured color picker
  - `selectedColor: UIColor` **[NEW]** — get/set the selected color
  - `supportsAlpha: Bool` **[NEW]** — enables/disables alpha slider
  - `delegate: UIColorPickerViewControllerDelegate?` **[NEW]**
- `UIColorPickerViewControllerDelegate` **[NEW]**
  - `colorPickerViewControllerDidSelectColor(_:)` **[NEW]** — called on every color change
  - `colorPickerViewControllerDidFinish(_:)` **[NEW]** — called when dismissed

### UIKit — UIDatePicker **[Updated]**
- `UIDatePicker.preferredDatePickerStyle` — `.automatic`, `.wheels`, `.compact` **[NEW]**, `.inline` **[NEW]**
- `.compact` — tappable date/time labels; modal calendar on date tap; keypad on time tap; available since iOS 13.4 SDK
- `.inline` — full calendar/time picker inline in view hierarchy
- All existing API unchanged: `date`, `minimumDate`, `maximumDate`, `calendar`, `locale`, `datePickerMode`, `addTarget(_:action:for:)`

### UIKit — UIMenu & UIContextMenuInteraction
- `UIButton.menu: UIMenu?` **[NEW]** — attach a menu to a button (shows on long press)
- `UIButton.showsMenuAsPrimaryAction: Bool` **[NEW]** — present menu on touch down
- `UIBarButtonItem.menu: UIMenu?` **[NEW]** — attach a menu; no primary action = immediate display
- `UIControl.contextMenuInteraction: UIContextMenuInteraction?` **[NEW]** — access control's context menu interaction
- `UIControl.isContextMenuInteractionEnabled: Bool` **[NEW]** — enable/disable the context menu interaction
- `UIControl.Event.menuActionTriggered` **[NEW]** — event fired when menu gesture is recognized
- `UIDeferredMenuElement` **[NEW]** — async menu items with loading indicator; cached after first load
  - `init(_ elementProvider: @escaping (([UIMenuElement]) -> Void) -> Void)` **[NEW]**
- `UIContextMenuInteraction.updateVisibleMenu(_:)` **[NEW]** — mutate displayed menu without dismissal
- `UIContextMenuInteraction.menuAppearance` **[NEW]** — `.rich`, `.compact`, `.none`
- `UIMenu` children are now mutable (not forced immutable as in iOS 13)
- Navigation Bar Back button automatic menu — no API required; set `navigationItem.backButtonTitle`

### UIKit — UIAction Enhancements
- `UIBarButtonItem(systemItem:primaryAction:menu:)` **[NEW]**
- `UIBarButtonItem(image:menu:)` **[NEW]**
- `UIBarButtonItem(primaryAction:)` **[NEW]**
- `UIBarButtonItem.fixedSpace(width:)` **[NEW]** — factory method replacing `UIBarButtonItem(barButtonSystemItem: .fixedSpace)`
- `UIBarButtonItem.flexibleSpace()` **[NEW]** — factory method
- `UIButton(type:primaryAction:)` **[NEW]** — button configured from action's title/image; calls handler on primaryActionTriggered
- `UISegmentedControl(frame:actions:)` **[NEW]** — segments defined by `UIAction` array; each action's handler called when its segment is selected
- `UISegmentedControl.insertSegment(action:at:animated:)` **[NEW]**
- `UISegmentedControl.setAction(_:forSegmentAt:)` **[NEW]**
- `UISegmentedControl.actionForSegment(at:)` **[NEW]**
- `UISegmentedControl.segmentIndex(identifiedBy:)` **[NEW]**

## Code Highlights

UIPageControl with custom images and prominent background:
```swift
let pageControl = UIPageControl()
pageControl.numberOfPages = 5
pageControl.backgroundStyle = .prominent
pageControl.preferredIndicatorImage = UIImage(systemName: "bookmark.fill")
pageControl.setIndicatorImage(UIImage(systemName: "heart.fill"), forPage: 0)
```

UIColorPickerViewController:
```swift
var colorPicker = UIColorPickerViewController()

func pickColor() {
    colorPicker.supportsAlpha = true
    colorPicker.selectedColor = currentColor
    present(colorPicker, animated: true)
}

func colorPickerViewControllerDidSelectColor(_ vc: UIColorPickerViewController) {
    currentColor = vc.selectedColor
}
```

UIDatePicker with compact style and Japanese calendar:
```swift
let datePicker = UIDatePicker()
datePicker.date = Date()
datePicker.preferredDatePickerStyle = .compact
datePicker.calendar = Calendar(identifier: .japanese)
datePicker.datePickerMode = .date
datePicker.addTarget(self, action: #selector(dateSet), for: .valueChanged)
```

UISegmentedControl created declaratively with UIAction:
```swift
enum SortOrder: CaseIterable {
    case name, date, size
    var title: String { /* ... */ }
}

let control = UISegmentedControl(frame: .zero, actions: SortOrder.allCases.map { order in
    UIAction(title: order.title) { _ in
        self.sortContent(by: order)
    }
})
```

UIDeferredMenuElement for async menu items:
```swift
button.menu = UIMenu(title: "", children: [
    UIDeferredMenuElement { completion in
        fetchMenuItems { items in
            completion(items.map { UIAction(title: $0.title) { _ in } })
        }
    }
])
```

## Takeaways
- Prefer `UIDatePicker` with `.compact` style for form-style date fields and `.inline` for dedicated date-selection screens; no existing API changes are needed beyond setting `preferredDatePickerStyle`.
- Use `UIColorPickerViewController` for any color selection UI — it handles the full spectrum, eyedropper, recent colors, and macOS compatibility without any custom picker code.
- Initialize `UISegmentedControl` with `UIAction` arrays to eliminate `addTarget`/switch-statement dispatch; the action handler is called only for its own segment, and enums map cleanly to actions.
- Use `UIDeferredMenuElement` for menus whose items require async work — the system shows a loading indicator automatically and caches results so the work runs only once per menu lifetime.

---
_Source: WWDC20 Session 10052 page (transcript, code samples, and resource links)._
