# What's New in AppKit
**WWDC22 · Session 10074** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10074/)

_Platforms:_ macOS Ventura 13

## Overview
This session surveys all major AppKit improvements in macOS Ventura: coordinating windows with Stage Manager, renaming "Preferences" to "Settings" across the system, a new inset-grouped form style for control-heavy interfaces, new controls (`NSComboButton`, updated `NSColorWell`), `NSToolbar` customization enhancements, an adaptive wide-layout `NSAlert`, lazy row-height calculation in `NSTableView`, SF Symbols 4 features (preferred rendering mode, variable symbols), and a redesigned sharing picker with collaboration support.

## Key Topics

### Stage Manager Window Coordination
Stage Manager moves inactive windows off-screen when a new primary window is brought to center stage. Auxiliary windows — panels, popovers, Settings windows, floating palettes — should not trigger this behavior. AppKit uses existing `NSWindow.collectionBehavior` to decide. If a window's behavior includes `.auxiliary`, `.moveToActiveSpace`, `.stationary`, or `.transient`, Stage Manager will not displace the active window. Modal windows and windows with preference-style toolbars are also exempt automatically.

### Preferences → Settings Rename
macOS Ventura renames "System Preferences" to "System Settings." AppKit automatically renames the standard "Preferences…" menu item in your app's application menu once you build against the Ventura SDK. Remaining uses of "Preferences" in window titles, labels, and other localized strings must be updated manually by developers.

### Inset-Grouped Form Style (SwiftUI on Mac)
Settings interfaces benefit from a new form style. In SwiftUI, apply `.formStyle(.grouped)` to a `Form` view to get the inset-grouped appearance: grouped sections with reduced-weight controls that highlight on hover. This style handles scrolling and layout automatically and is ideal for Settings windows and inspectors.

### NSComboButton (New)
`NSComboButton` combines a primary action button and a pull-down menu in a single control — replacing ad-hoc `NSSegmentedControl` solutions. Two styles:
- `.split` (default) — separate arrow segment for the menu; primary action fires on clicking the main portion
- `.unified` — looks like a normal button; primary action on click, menu on click-and-hold

### NSColorWell Enhancements
`NSColorWell` adopts a new bezel-style appearance automatically. Two new styles via `colorWellStyle`:
- `.minimal` — shows a disclosure arrow on hover; opens a color-palette popover with an option to open the full `NSColorPanel`
- `.expanded` — combines a miniature well (popover) on the left and a full-panel button on the right (as seen in iWork apps)

New `pulldownAction` target-action pair: override the popover that appears when the user clicks the disclosure arrow, e.g., to show a custom color menu.

### NSToolbar Enhancements
Three new API additions:
- `toolbarImmovableItemIdentifiers` delegate method — items returned here cannot be moved or removed during customization
- `toolbar(_:itemIdentifier:canBeInsertedAt:)` delegate method — veto power over any insertion, removal, or reordering
- `centeredItemIdentifiers` property — multiple items that float together in the center of the toolbar; customizable only within the centered group
- `NSToolbarItem.possibleLabels` — a set of localized strings the item might display; toolbar pre-sizes the item to fit the widest string, preventing layout shifts when the label changes

### NSAlert Wide Layout
For alerts with long informative text or large accessory views that do not fit the compact alert window, `NSAlert` now automatically uses a wider layout. No adoption required; determined at presentation time. Compact alerts continue to work as before for short text.

### NSTableView Lazy Row Heights
`NSTableView` now lazily calculates row heights for variable-height rows, using running estimated heights for unmeasured rows and measuring on demand as the user scrolls. This significantly improves initial load times for large tables. `tableView(_:heightOfRow:)` calls are no longer eager; do not assume timing. Applies to `NSTableView` and SwiftUI's `List` automatically on macOS Ventura.

### SF Symbols 4
- 450+ new symbols (laurels, household objects, currency symbols, sports)
- **Preferred rendering mode** — symbols declare their preferred rendering mode (e.g., hierarchical for AirPods Pro); AppKit uses it automatically at runtime without needing an explicit `NSImageSymbolConfiguration`
- **Variable symbols** — symbols that vary their appearance based on a 0–1 floating-point value (e.g., Wi-Fi signal strength, volume). Created via `NSImage(systemSymbolName:variableValue:accessibilityDescription:)`

### Sharing and Collaboration
`NSSharingServicePicker` gains a new sharing popover with suggested people, document preview header, and contact information. Key API additions:
- `NSPreviewRepresentableActivityItem` protocol — custom items conform to provide title, image provider, and icon provider for the picker's header
- `NSPreviewRepresentingActivityItem` class — convenience wrapper bundling an existing item with its preview metadata
- `NSSharingServicePicker.standardShareMenuItem` — creates a ready-made `NSMenuItem` that summons the sharing popover from any menu; anchors the popover to the view that produced the context menu

Full collaboration invitations (CloudKit, iCloud Drive, or custom server) are covered in separate sessions.

## APIs & Frameworks

**AppKit (all new in macOS Ventura 13)**

_Stage Manager_
- `NSWindow.collectionBehavior` — `.auxiliary`, `.moveToActiveSpace`, `.stationary`, `.transient` options exempt windows from Stage Manager displacement

_Controls_
- `NSComboButton` **[NEW]** — combined primary-action + menu control
  - `NSComboButton.Style.split` **[NEW]**
  - `NSComboButton.Style.unified` **[NEW]**
- `NSColorWell.colorWellStyle` **[NEW]** — `.default`, `.minimal`, `.expanded`
- `NSColorWell.pulldownAction` / `NSColorWell.pulldownTarget` **[NEW]** — custom popover action

_Toolbar_
- `NSToolbarDelegate.toolbarImmovableItemIdentifiers(_:)` **[NEW]**
- `NSToolbarDelegate.toolbar(_:itemIdentifier:canBeInsertedAt:)` **[NEW]**
- `NSToolbar.centeredItemIdentifiers` **[NEW]**
- `NSToolbarItem.possibleLabels` **[NEW]**

_Alerts_
- `NSAlert` wide layout — automatic for long informative text or large accessory views **[NEW behavior]**

_Table View_
- `NSTableView` lazy row-height calculation **[NEW behavior]** — on by default

_SF Symbols 4_
- Preferred rendering mode — declared per-symbol, applied automatically at runtime **[NEW]**
- `NSImage(systemSymbolName:variableValue:accessibilityDescription:)` **[NEW]** — variable symbol image

_Sharing_
- `NSPreviewRepresentableActivityItem` protocol **[NEW]**
- `NSPreviewRepresentingActivityItem` class **[NEW]**
- `NSSharingServicePicker.standardShareMenuItem` **[NEW]**

**SwiftUI (macOS Ventura)**
- `.formStyle(.grouped)` **[NEW]** — inset-grouped settings form layout

## Code Highlights

SwiftUI inset-grouped form for a Settings window:
```swift
struct ExampleFormView: View {
    @State private var name: String = "Mac Studio"
    @State private var screenSharingEnabled: Bool = true
    @State private var fileSharingEnabled: Bool = false
    @State private var airdropVisibility = AirDropVisibility.contactsOnly

    var body: some View {
        Form {
            TextField("Computer Name", text: $name)
            Toggle("Screen Sharing", isOn: $screenSharingEnabled)
            Toggle("File Sharing", isOn: $fileSharingEnabled)
            Picker("AirDrop", selection: $airdropVisibility) {
                ForEach(AirDropVisibility.allCases) {
                    Label($0.label, systemImage: $0.symbolName)
                        .labelStyle(.titleAndIcon)
                        .tag($0)
                }
            }
        }
        .formStyle(.grouped)
    }
}
```

Variable symbol image:
```swift
let signalImage = NSImage(
    systemSymbolName: "wifi",
    variableValue: signalStrength,   // 0.0 – 1.0
    accessibilityDescription: "Wi-Fi signal"
)
```

NSComboButton creation (split style):
```swift
let comboButton = NSComboButton(
    title: "Move to Inbox",
    menu: folderMenu,
    target: self,
    action: #selector(moveToInbox(_:))
)
comboButton.style = .split
```

Sharing picker with standard menu item:
```swift
let picker = NSSharingServicePicker(items: [url])
let menuItem = picker.standardShareMenuItem
contextMenu.addItem(menuItem)
```

## Takeaways
- Set `NSWindow.collectionBehavior` correctly for auxiliary windows so Stage Manager does not displace the active window when they appear.
- Use `NSComboButton` instead of segmented control workarounds when combining a primary action with a menu; use `.minimal` / `.expanded` `NSColorWell` styles for richer color picking.
- Apply `.formStyle(.grouped)` to SwiftUI `Form` views inside Settings windows to match the macOS Ventura System Settings visual language.
- Use `NSImage(systemSymbolName:variableValue:)` for variable symbols like signal strength; rely on preferred rendering mode to automatically apply the symbol's optimal visual style without explicit `NSImageSymbolConfiguration`.

---
_Source: WWDC22 Session 10074 page (abstract, transcript, and code samples)._
