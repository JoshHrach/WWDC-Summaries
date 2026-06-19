# Elevate the Design of Your iPad App
**WWDC25 · Session 208** · [Watch](https://developer.apple.com/videos/play/wwdc2025/208/)

_Platforms:_ iPadOS 26

## Overview
iPadOS 26 introduces a sweeping redesign of multitasking and navigation that makes iPad apps feel more Mac-like and powerful without sacrificing touch simplicity. This design-focused session walks through the four pillars of the new system: fluid navigation (sidebar ↔ tab bar morphing), the new windowing system (floating windows, Window Controls, additive multi-window behavior), a redesigned pointer with Liquid Glass hover effects, and the new iPad menu bar.

The session provides concrete guidance on how developers should update their apps to take full advantage — especially around wrapping toolbars around window controls, providing descriptive window names, and designing static, always-visible menu bars.

## Key Topics

### Navigation
- **Sidebar** — ideal for deep content hierarchies (Mail, Music); flattens navigation to the top level for fast access.
- **Tab bar** — more compact and immersive; the recommended starting point for most apps.
- **Sidebar ↔ Tab bar morphing** — **[NEW]** a tab bar can fluidly morph into a sidebar and back; orientation change (portrait → landscape) can trigger this transition automatically.
- Layout must be **non-destructive** on resize — reverting to the previous state when the window returns to its original size.
- Use the new **scroll edge effect** to draw content beneath the toolbar and sidebar for a more immersive feel.

### Windows
- **[NEW windowing system]** — any app that supports multitasking shows a resize handle in the bottom-right corner; dragging creates a floating window above the wallpaper.
- **Window Controls** — appear on the leading edge of the toolbar; tapping reveals their functionality; press-and-hold shows layout shortcut grid.
- **Wrap toolbars** around window controls so they appear inline — do not rely on the system adding a safe area above the toolbar (that placement is compatibility-only and wastes screen space).
- **Additive multi-window behavior** — **[NEW]** each document should open in its own new window rather than replacing the current one; "Open in Place" is no longer recommended for multitasking apps.
- The app menu now includes a list of open windows — provide **descriptive, unique window names** (e.g., document title) so users can identify them.

### Pointer
- **New pointer shape** — **[NEW]** a precise, arrow-style cursor that tracks input 1:1 (no magnetization or rubber-banding to targets).
- **Liquid Glass highlight** — **[NEW]** replaces the old morph-into-highlight hover effect; a Liquid Glass platter materializes on top of hovered buttons and refracts underlying content.
- For clusters of buttons, the highlight snaps quickly between adjacent targets as the pointer moves.
- Test your app with the new pointer to identify unexpected layout or interaction issues.

### Menu Bar
- **[NEW]** Every app on iPad now gets a menu bar (revealed by moving the pointer to the top edge or swiping down).
- Structure: App menu → System-provided default menus → App custom menus.
- **Custom menu design rules**:
  - Order items by **frequency of use** (not alphabetically).
  - Group related items into **sections**.
  - Move secondary actions into submenus if a menu becomes too long.
  - Assign **SF Symbols** matching in-app representations.
  - Assign **keyboard shortcuts** to frequently performed actions.
- **Populate the View menu** with tabs (with keyboard shortcuts for fast switching) and a sidebar toggle.
- **Never hide inactive menu items** — dim them instead. Users rely on spatial memory; disappearing items are disorienting.
- **Never hide entire menus** — even if all items in a menu are inactive, keep the menu visible.

## APIs & Frameworks

### UIKit / iPadOS
- Sidebar / tab bar morphing — `UISplitViewController` + `UITabBarController` adaptive behavior. **[NEW morphing behavior]**
- Scroll edge effect — new modifier for extending content under navigation UI. **[NEW]**
- `UIWindowScene` — multi-window support; provide descriptive `title` for each scene.
- Window Controls integration — wrap `UINavigationBar` around the window controls region. **[NEW]**
- Pointer hover effect — `UIPointerInteraction`, `UIPointerStyle`; updated to Liquid Glass style. **[NEW]**
- Menu bar — `UIMenuBuilder`, `UICommand`, `UIKeyCommand`. **[NEW on iPad]**
- `UIResponder.buildMenu(with:)` — populate custom menu items.
- HIG: Windows, The menu bar, Multitasking — updated guidelines linked in Resources.

## Code Highlights
This is a design-focused session; no code samples were shown. See the Human Interface Guidelines for Windows, The menu bar, and Multitasking for implementation references.

## Takeaways
- Wrap your app's `UINavigationBar` around the window controls position to reclaim the safe area and display more content.
- Adopt additive multi-window behavior — open each document in a new `UIWindowScene` rather than replacing the current one, and give each scene a descriptive `title`.
- Design a static menu bar: all items always present, dimmed when inactive, never hidden — this is the single most important rule for a predictable menu bar experience.
- Start with a tab bar if unsure about navigation patterns; it can morph into a sidebar as the app's content hierarchy grows.

---
_Source: WWDC25 Session 208 page (abstract, chapter summaries, and full transcript)._
