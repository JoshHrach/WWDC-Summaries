# Designing iPad Apps for Mac
**WWDC19 · Session 809** · [Watch](https://developer.apple.com/videos/play/wwdc2019/809/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15 (Mac Catalyst)

## Overview
This design session covers how to adapt an iPad app's visual and interaction design for Mac using Mac Catalyst (iPad apps for Mac). It addresses the key differences between touch-based iOS and pointer-based macOS interaction models and provides concrete guidance across eight design areas: app architecture, toolbars, layout, typography, color, gestures/hover, app icons, contextual menus, and menu bars. The session identifies what is automatically translated by the system (UIKit gestures → pointer events, split views, edit menus, activity sheets), what requires deliberate redesign (touch targets, bottom-edge controls, gesture-only actions), and what represents a Mac-specific design opportunity (sidebar navigation, hover states, Touch Bar, menu bar completeness, keyboard shortcuts).

## Key Topics

- **Prerequisites for a good Mac Catalyst app** — The iPad app must support multitasking, drag and drop, Auto Layout, and (ideally) multiple windows. These automatically translate to Mac-native equivalents.
- **What's automatic** — iOS split views → Mac split views; file browser → open panel; activity view → share menu; edit menus → contextual menus; copy/paste/rich text editing/key focus.
- **App architecture** — Tabbed apps can use segmented controls or (better) a sidebar. Table-view top-level navigation maps directly; enable translucent background for a Mac-native sidebar feel. Document browser apps can use a sidebar for persistent folder/saved-search access.
- **Sidebar design rules** — Sidebars show locations/collections, not content items. Use translucent background (not solid colors or images). Use system selection color (not app tint) for selected items. Use template/vibrancy images unless full color is necessary.
- **Toolbars** — Place frequent actions in the toolbar, not along the bottom edge of the window (bottom-edge UI is problematic on Mac because windows are draggable). Toolbar actions persist across views and disable when inapplicable; contextually-relevant actions go in an Action menu (which can change per view and selection).
- **Layout** — 77% scale factor applied uniformly to content areas (13pt Mac baseline ÷ 17pt iOS baseline). Recreate this in design tools by scaling content smart objects to 77%. Use readable content margins, multiple columns, and split/master-detail views. Test iPad layouts for Mac first to identify issues.
- **Typography** — Dynamic Type largest size is used, then scaled to 77%. Caption and footnote styles can be too small at Mac scale — bump them up. macOS does not support Dynamic Type (always uses the Large accessibility size baseline).
- **Color** — Mac interfaces are more neutral. Don't colorize bar backgrounds or content areas. iOS system colors remap to macOS equivalents for both light and dark modes automatically. Tint accent colors are reduced in iOS 13 (steppers and segmented controls are now neutral). Use system highlight colors for selections — don't override with tint colors.
- **Gestures → pointer/trackpad events** — Single tap → mouse down; long press → mouse down and hold; pan → mouse drag; swipe → directional drag. Pinch/rotate work on trackpads but center on cursor position rather than midpoint between fingers. Screen-edge swipes have no Mac equivalent. Any gesture-exclusive actions need an alternative (menu bar item, contextual menu, toolbar button).
- **Hover events** — Mac apps receive mouse hover events. Use them to reveal additional information without requiring a selection change (e.g., the Stocks chart reveals the price at the pointer position on hover rather than requiring a tap).
- **Touch Bar** — Touch bars can be implemented for Mac Catalyst apps to display contextually relevant information and controls.
- **App icons** — By default, iOS square icon displays with a continuous-curve mask and subtle drop shadow. For a great Mac app, create a dedicated Mac icon with unique silhouette, pixel-hinted small sizes (16px and 32px at 1x), and realistic 3D rendering with Mac-standard light source and camera angle.
- **Contextual menus** — Mac users expect contextual menus on everything. iOS contextual menus automatically map to Mac contextual menus. Design guidelines: avoid too many items, use one-word verb labels, order by importance, group related items with separators, use submenus for progressive disclosure.
- **Menu bars** — Every Mac app has a menu bar. Catalog every action in the app and assign it to a menu: standard menus (File, Edit, Format, View, Window, Help) handle most needs; add custom menus for major object types or workflows with many related actions. Menu bar structure is stable (no adding/removing items after launch). Assign keyboard shortcuts to frequent commands following macOS HIG conventions.

## APIs & Frameworks

### Mac Catalyst (UIKit on macOS) **[NEW]**
- `UISplitViewController` — automatic sidebar translation with `.primaryBackgroundStyle = .sidebar` **[NEW]**
- `UIBarButtonItem` — toolbar items; auto-disabled when not applicable
- `UIMenu` / `UIAction` — contextual menus (iOS 13); automatically maps to macOS contextual menus **[NEW]**
- `NSToolbar` — via Mac Catalyst, UIKit toolbars bridge to NSToolbar
- `UIHoverGestureRecognizer` — hover event support on Mac **[NEW]**
- `UITouch.TouchType.indirectPointer` — pointer input distinction **[NEW]**
- Keyboard shortcuts — `UIKeyCommand` used in `keyCommands` property or `UIMenuBuilder` API **[NEW]**
- `UIApplicationDelegate.buildMenu(with:)` — construct menu bar via `UIMenuBuilder` **[NEW]**
- `UIMenuSystem` — `.main` for menu bar, `.context` for contextual menus **[NEW]**
- `UICommand` / `UIKeyCommand` — menu bar command definition with keyboard shortcuts **[NEW]**

### Design Resources
- macOS Apple Design Resources (Sketch, Photoshop, XD) — available at developer.apple.com/design
- macOS HIG keyboard shortcuts page — standard shortcut list to follow for precedent
- Dedicated Mac Catalyst HIG section added in macOS Catalina docs

## Code Highlights

No code samples. This is a design patterns session.

Key design rules summary:

```
Platform translation table:
  iOS tab bar            → segmented control in toolbar OR sidebar
  iOS master list        → sidebar (set primaryBackgroundStyle = .sidebar)
  iOS document browser   → sidebar for folders + document browser content area
  Bottom-edge controls   → toolbar items
  Gesture-only actions   → menu bar item + contextual menu + toolbar button
  iOS tint for selection → system selection color
  Solid bar backgrounds  → translucent/vibrancy backgrounds
  17pt text              → system scales to 77% (≈ 13pt Mac baseline)
  Caption/footnote styles → may need explicit size increase
```

## Takeaways

- The 77% content scale factor is not negotiable — design all iPad layouts with this in mind and verify at Mac scale. Use design tool smart objects scaled to 77% for Mac mockups.
- The menu bar is as important as your main UI on Mac; catalog every action your app can perform and ensure it appears in a menu with a logical keyboard shortcut — this is what makes apps feel native and accessible.
- Every gesture-exclusive action needs an alternative input path (menu item, toolbar button, contextual menu) — trackpad users may not have Multi-Touch, and there is no pull-to-refresh or edge swipe on Mac.
- Sidebars require a translucent background with system selection colors — custom colors and solid fills break the window-focus visual system that Mac users rely on to know which window will receive keyboard input.

---
_Source: WWDC19 Session 809 page (abstract, full transcript, and resource links)._
