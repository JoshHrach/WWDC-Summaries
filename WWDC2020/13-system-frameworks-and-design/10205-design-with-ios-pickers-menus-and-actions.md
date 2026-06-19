# Design with iOS Pickers, Menus and Actions
**WWDC20 · Session 10205** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10205/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11

## Overview
iOS 14 introduces three new UI components — pull-down menus, redesigned date/time pickers, and a system color picker — designed to make apps faster, lighter, and more directly interactive. The session is a design-focused walkthrough that explains why these controls replace older patterns (action sheets, wheel pickers) and how to use them correctly in both iPhone and iPad apps.

The new pull-down menus appear directly adjacent to the triggering button rather than sliding up from the bottom of the screen, reducing finger travel and eliminating the heavy dimming transition of action sheets. They support disambiguation, navigation, selection, and secondary actions. Critically, destructive actions still require confirmation via action sheets or popovers, which intentionally add friction.

The redesigned date/time pickers present a calendar-grid layout for dates and a direct-entry field for times, replacing the legacy spinning-wheel UI. A new compact presentation mode renders as a small button that expands into a modal overlay — ideal for inline contexts like form rows.

## Key Topics

### Pull-Down Menus
- Appear adjacent to the tapped button; no background dimming; fast, lightweight transition.
- Support SF Symbols or custom images as trailing icons, optional title, separators for hierarchy.
- No cancel option needed — tapping outside dismisses with no changes.
- Four primary use cases: **disambiguation** (ask a more specific question after a clear action), **navigation** (back-stack history list), **selection** (checkmarked list of choices), **secondary actions** ("more" button pattern).
- Do not hide all primary actions behind a menu; keep frequent, important actions prominent.
- For tap-and-hold variants, a single button can expose different menus on tap vs. long press.
- Destructive actions: always use action sheets (iPhone) or popovers (iPad) for confirmation to ensure friction.

### Redesigned Date and Time Pickers
- Three configurations: date picker (calendar grid), time picker (direct text entry), combined date-and-time.
- Inline mode: embed directly in the view (e.g., Reminders date-picker row).
- Compact mode **[NEW]**: renders as a small button showing the current value; tap expands a modal overlay picker.
- Supports all input modalities: touch, Apple Pencil, hardware keyboard, pointer/cursor.
- Improved parity between iPad and Mac Catalyst apps.

### Color Picker **[NEW]**
- Four selection modes: color grid, spectrum gradient, RGB hex/numeric entry, and on-screen color sampler (pipette/magnifier).
- Selected color shown in bottom-left swatch; colors can be saved to a shared system palette accessible across apps.
- System component ensures consistent behavior, accessibility, and localization.

## APIs & Frameworks

### UIKit
- `UIMenu` **[NEW — pull-down from any button]**
- `UIAction` — represents a single menu action with title, image, identifier, handler
- `UIMenuElement` — base type for menu items
- `UIMenuOptions` — `.displayInline`, `.destructive`
- `UIButton.menu` property **[NEW in iOS 14]** — assign a `UIMenu` to any button
- `UIButton.showsMenuAsPrimaryAction` **[NEW]** — present menu on tap (vs. long press)
- `UIContextMenuInteraction` — existing context-menu API (iOS 13) that menus are visually aligned with
- `UIDatePicker` — existing class, new styles added:
  - `UIDatePickerStyle.compact` **[NEW]**
  - `UIDatePickerStyle.inline` **[NEW]**
  - `UIDatePickerStyle.wheels` — legacy style (still available)
- `UIColorPickerViewController` **[NEW]** — system color picker view controller
- `UIColorPickerViewControllerDelegate` **[NEW]** — `colorPickerViewControllerDidSelectColor(_:)`, `colorPickerViewControllerDidFinish(_:)`

### Human Interface Guidelines
- HIG: Designing for iOS — menus, pickers, and color picker guidance

## Code Highlights
No code samples were included in this design-focused session. The companion engineering session "Build with iOS pickers, menus and actions" (WWDC20 session 10052) covers implementation details.

## Takeaways
- Replace action sheets (iPhone) and popovers (iPad) with pull-down `UIMenu` for disambiguation, navigation, selection, and secondary actions — but keep action sheets for destructive confirmation flows.
- Use `UIDatePicker` with `.compact` or `.inline` style instead of the legacy `.wheels` style wherever possible.
- Adopt `UIColorPickerViewController` to give users a full-featured, system-standard color selection experience with a cross-app shared palette.
- All three new components are fully accessibility-ready (VoiceOver, Larger Text, Increased Contrast, Reduce Motion) and improve iPad-to-Mac Catalyst parity.

---
_Source: WWDC20 Session 10205 page (abstract, transcript, and resource links)._
