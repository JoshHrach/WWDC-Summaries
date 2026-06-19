# Designed for iPad
**WWDC20 · Session 10206** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10206/)

_Platforms:_ iPadOS 14, macOS Big Sur 11 (Mac Catalyst)

## Overview
This design-focused session presents the core principles and new components for building apps that feel genuinely native to iPad — not simply iPhone apps scaled up. The session is organized around four themes: filling the display intelligently, supporting all iPad input modalities, leveraging the new Sidebar navigation, and placing toolbar actions correctly.

iPadOS 14 introduces a redesigned Sidebar that replaces or supplements tab bars for regular-width layouts. The Sidebar supports collapsible sections, drag-and-drop, overlay presentation, and spring-loaded folders. It converts naturally to a Mac Catalyst sidebar, making it the right structural investment for cross-platform apps. Three-column layouts are now available on all iPads (not just the largest iPad Pro).

Toolbars in iPadOS 14 are repositioned to the top navigation bar rather than the bottom of the screen, making better use of the wide display and reducing thumb travel.

## Key Topics

### Filling the iPad Display
- **Flatten navigation**: Replace full-screen push transitions with split views that show navigation and content simultaneously (e.g., Photos' new sidebar + content layout).
- **Show more content**: Increase density thoughtfully — smaller icons in Files showing nearly 3x as many items at once while remaining legible and tappable.
- **Add more context**: Bring controls inline (e.g., in-place file renaming) rather than overlaying full-screen modals; use popovers only when they provide genuine spatial context.
- **Immersive focus**: For content-centric modes (editing a photo, Now Playing), design custom full-display layouts that keep controls accessible without covering the content.

### iPad Input Support
- Always start with Multi-Touch — every other input is additive.
- Support keyboard shortcuts for common actions (they transfer directly to Mac Catalyst).
- Pointer/Trackpad support is largely automatic for system controls; extend it for custom views (see "Designing for the iPadOS Pointer").
- Support Scribble with Apple Pencil for all text-input controls.
- Combine inputs for unique interactions: Command+tap for link behaviors in Safari; Pencil + simultaneous touch (e.g., dial control + drawing canvas in Loom).
- Keep the app **always responsive**: allow input during animations; dismiss menus via scroll gesture outside them.

### Sidebar (New in iPadOS 14)
- Replaces or complements the tab bar in **regular-width** layouts only.
- Supports overlay presentation (auto in Portrait and compact Split View); toggled by sidebar button or left-edge swipe gesture.
- Supports **three-column layouts** on all iPads.
- Supports drag-and-drop for rearranging items and creating shortcuts.
- Supports collapsible section headers, spring-loaded folders, and Edit mode for user customization.
- Sidebars built for iPadOS convert directly into macOS sidebars in Mac Catalyst.

**Sidebar design rules:**
- Keep primary navigation (tab equivalents) at the top.
- Never mix sidebar and tab bar in the same view — they are two presentations of the same structure.
- Use outlined SF Symbol glyphs in the sidebar; use filled glyphs in the tab bar.
- Add user-configurable list content (albums, playlists, folders) below primary items, nested under collapsible headers.
- Support add buttons at the bottom of configurable sections.
- In compact width (iPhone, compact multitasking), convert sidebar to tab bar or table rows.

### Toolbar Placement
- In iPadOS 14, move toolbar buttons to the top navigation bar instead of the bottom toolbar.
- For compact-width contexts, retain buttons at the bottom.

## APIs & Frameworks

### UIKit
- `UISplitViewController` **[NEW multi-column API in iOS 14]** — supports two- and three-column layouts
- `UISplitViewController.Style` — `.doubleColumn`, `.tripleColumn` **[NEW]**
- `UINavigationController` with sidebar integration
- `UICollectionView` with compositional layout — used for sidebar list layouts
- `UICollectionLayoutListConfiguration` **[NEW]** — list-style collection view sections with sidebar appearance
- `UICollectionViewListCell` **[NEW]**
- `NSSidebarAppearance` (Mac Catalyst) — automatic from iPadOS sidebar
- `UITabBarController` — still used in compact-width
- `UIBarButtonItem` — repositioned to navigation bar in iPadOS 14
- Pointer interaction APIs (see session 10640)
- Scribble / `UIScribbleInteraction` — automatic for system text fields

### Human Interface Guidelines
- HIG: Designing for iOS — sidebar, split view, toolbar guidance

## Code Highlights
No code samples were included in this design-focused session. The companion engineering session "Build for iPad" covers implementation.

## Takeaways
- A great iPad app is not a scaled-up iPhone app: flatten navigation, increase content density, minimize modals, and support all input types as first-class citizens.
- Adopt the new Sidebar for regular-width layouts; always keep the tab bar for compact-width; never show both simultaneously.
- Three-column split views are now available on all iPads — use them for hierarchical apps like Mail or Files.
- Move toolbar actions to the top navigation bar on iPad to maximize content area; the same code translates well to Mac Catalyst.

---
_Source: WWDC20 Session 10206 page (abstract, transcript, and resource links)._
