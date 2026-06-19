# What's New in iPad App Design
**WWDC22 · Session 10009** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10009/)

_Platforms:_ iPadOS 16

## Overview
iPadOS 16 introduces a set of design improvements that help iPad apps become more powerful and desktop-like, targeting two main use cases: document editing and content browsing. The session covers new toolbar layouts, document menus, edit menus, find and replace, navigation patterns, search placement, tables, and selection behaviors.

A key theme is supporting larger screens, Stage Manager, extended display resolutions, display zoom, and a full range of inputs (touch, pointer, keyboard). The new toolbar layout moves titles to the left and adds more center space for frequently used actions, with support for customizable, grouped, and collapsing items.

For content browsing, browser-style navigation with back/forward buttons, inline search in the navigation bar, improved multi-selection without entering edit mode, submenus in context menus, pop-up buttons in lists, and multi-column sortable tables all contribute to a more powerful and efficient experience.

## Key Topics

### Toolbar and Customization
New toolbar layout: leading section for navigation/back and document title, center section for common actions, trailing section for inspectors and overflow. Customizable toolbars let users add/remove/rearrange center items. Toolbar items can be grouped (related actions collapse into a single button) and collapse to compact glyphs at smaller window sizes.

### Document Menu
A new title-and-menu in the toolbar for document-editing apps. Contains document-level actions (Duplicate, Rename, Move, Export, Print) plus app-specific actions. Share-related actions stay under the Share button; in-document content actions use toolbar customization and edit menus.

### Edit Menus
Redesigned for both touch (horizontal scrollable menu) and pointer (vertical comprehensive list). Custom actions should be organized near related system actions. Applies to text fields and document canvas objects. Standard actions (Cut, Copy, Paste) must always remain available.

### Find and Replace
System-level find and replace integrated into the keyboard, available above the app when a hardware keyboard is attached. Supports match options (case sensitivity, whole word) and replace-all. Add to overflow menu and support keyboard shortcuts.

### Browser-Style Navigation
New navigation pattern for complex hierarchies (file browsers, web browsers): back and forward buttons left of the title. For flat/shallow hierarchies, sidebar-based navigation is sufficient.

### Search in Navigation Bar
Search field placed in the top-right of the navigation bar for filtering content on the current screen. Supports recent searches, query suggestions, and filter suggestions.

### Tables
New multi-column table control for SwiftUI on iPadOS 16. Supports column sorting by tapping headers and swapping which columns are shown. Fully supports multi-selection. At compact widths, automatically collapses to a single-column list.

### Multi-Selection and Context Menus
Band selection no longer automatically enters edit mode. Command/Shift keyboard selection without edit mode. Secondary click or long press for multi-item context menus. Context menus in empty areas for creating new items.

### Submenus and Pop-Up Buttons in Lists
Submenus open horizontally on iPadOS 16 for fast pointer interaction. Pop-up buttons in lists replace navigation pushes for simple option selection (well-suited for small, well-defined option sets).

## APIs & Frameworks

**UIKit / SwiftUI**
- `UINavigationBarAppearance` — toolbar/navigation bar customization
- Customizable toolbars **[NEW in iPadOS 16]** — center section items are add/remove/rearrangeable
- Toolbar item groups with collapsing behavior **[NEW]**
- Document menu (title menu) in navigation bar **[NEW]** — for document-editing apps
- Edit menu (UIEditMenuInteraction / `editMenu`) **[NEW redesigned]** — horizontal (touch) and vertical (pointer) layouts
- System find and replace (UIFindInteraction) **[NEW]** — integrated keyboard panel
- Browser-style navigation (back/forward buttons) **[NEW pattern]**
- Search in navigation bar (trailing position) **[NEW]** — with suggestions support
- `Table` (SwiftUI) **[NEW]** — multi-column sortable table
- `TableColumn` (SwiftUI) **[NEW]** — column definition for `Table`
- Multi-selection without edit mode **[NEW]** — pointer/keyboard-driven band selection
- Multi-item context menus **[NEW]** — act on multiple selected items
- Empty-area context menus **[NEW]** — create new items from empty space
- Submenus in context menus (horizontal opening on iPad) **[NEW behavior]**
- Pop-up buttons in list rows **[NEW]** — inline option pickers replacing navigation pushes
- Stage Manager multi-window support

## Code Highlights

No code samples were shown in this session (design-focused). See related sessions "Build a desktop-class iPad app" and "SwiftUI on iPad: Add toolbars, titles, and more" for implementation details.

## Takeaways

- The new toolbar layout (title left, actions center, controls trailing) enables more efficient document editing; enable customization when apps have many optional features.
- Use the document menu for document-level actions (rename, export, print) and keep share/in-content actions in their appropriate locations.
- Tables are for richer list views with multiple sortable columns, not spreadsheets; they gracefully collapse to single-column lists at compact widths.
- Multi-selection in iPadOS 16 no longer requires entering edit mode — support pointer band selection, Command/Shift keyboard selection, and multi-item context menus for a desktop-class feel.

---
_Source: WWDC22 Session 10009 page (abstract, chapter summaries, code samples, and resource links)._
