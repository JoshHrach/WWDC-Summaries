# What's New in iOS Design
**WWDC19 · Session 808** · [Watch](https://developer.apple.com/videos/play/wwdc2019/808/)

_Platforms:_ iOS 13, iPadOS 13

## Overview
iOS 13 brings three major design changes that affect virtually every app: Dark Mode with a fully rebuilt semantic color system; redesigned modal presentations using a card-style sheet; and contextual menus as a replacement and expansion of the 3D Touch Peek and Pop interaction. Together these three areas represent the most comprehensive refresh of the iOS design system in years.

The session walks through each area from a designer's perspective, explaining the rationale behind design decisions (information hierarchy, vibrancy, accessibility) and providing clear guidance on when and how to adopt each feature. Apps that use standard UIKit controls get many changes automatically; custom controls require careful review against the new semantic color palette.

## Key Topics

### Dark Mode and the Semantic Color System
- Dark Mode uses a fully black background for maximum contrast and seamless hardware bezel integration.
- The iOS design system now provides a palette of **semantic colors** (colors described by purpose, not value) that automatically adapt between Light and Dark Mode.
- Label colors: primary (`label`), secondary (`secondaryLabel`), tertiary (`tertiaryLabel`), quaternary (`quaternaryLabel`) — express information hierarchy.
- Background colors: `systemBackground`, `secondarySystemBackground`, `tertiarySystemBackground` — and a parallel grouped set (`systemGroupedBackground`, etc.).
- **Base vs. Elevated** backgrounds: modal and foreground interfaces use the elevated (slightly lighter) variant in Dark Mode to create visual separation without drop shadows.
- Fill colors and separator colors are semi-transparent so they adapt gracefully to any background.
- Six fully opaque gray system values are provided for cases where transparency causes visual artifacts (e.g., overlapping grid lines).
- Tint colors are now dynamic (Light and Dark variants) with high-contrast accessibility variants; aim for a 4.5:1 contrast ratio minimum.
- Custom colors can also be dynamic; use `UIColor(dynamicProvider:)` to provide Light/Dark variants.
- Materials overhauled: four levels — thick, regular (default), thin, ultra-thin — each with vibrancy values for labels, fills, and separators.
- Vibrancy values for use inside materials maintain legibility regardless of background content.

### SF Symbols **[NEW]**
- Over 1,500 symbols designed to match San Francisco font metrics; can be used inline with text with correct baseline alignment.
- Available in nine weights (matching SF font weights) and three scale variants (small, medium, large).
- Vector-based: scale with Dynamic Type; become bolder when Bold Text accessibility setting is on.
- Use the SF Symbols app to browse and copy symbols; paste into Sketch text layers via Apple Design Resources.
- Custom symbols can be created by exporting an SVG template and modifying in Illustrator or Sketch.

### Navigation Bar Updates
- Large-title navigation bars no longer show a background or shadow by default; background/shadow fades in as content scrolls beneath.
- Standard (non-large-title) bars match this style in appropriate contexts.
- Avoid the transparent bar treatment when content is tucked beneath the bar or when dense interfaces need clear visual delineation.

### Modal Presentations — Card-Style Sheets
- Card-style modal presentation is now the **default** in iOS 13; the sheet slides up with a peek of the background visible, providing context.
- Dismiss by swiping down anywhere on the card (scroll to top first if content has been scrolled).
- Pull from the top edge at any time to dismiss immediately.
- The dismiss gesture can be prevented for mandatory decisions; in that case show an action sheet to explain options.
- Buttons (Cancel/Done) remain required for accessibility and discoverability even with swipe-to-dismiss available.
- Full-screen presentation remains available and should be used when maximizing screen space is important (e.g., photo editing, markup).
- Modal presentations are for switching modes — do not use them purely for their visual style.

### Contextual Menus **[NEW]**
- Replace Peek and Pop; work on all devices (not just 3D Touch devices) via tap-and-hold; 3D Touch still activates them faster.
- Comprised of an action menu and an optional preview of the affected item.
- On iPhone (portrait) and iPad (3 commands or fewer): preview and menu stacked vertically. Otherwise: side by side.
- Commands should also be accessible elsewhere in the main UI; contextual menus are a power-user shortcut, not the only path.
- Support command ordering, visual grouping (separators), glyphs, hierarchical submenus, and destructive action styling (red text).
- Add contextual menus to every object in the app, following the macOS precedent where right-click is universally expected.

## APIs & Frameworks

### UIKit — Colors
- `UIColor.label`, `UIColor.secondaryLabel`, `UIColor.tertiaryLabel`, `UIColor.quaternaryLabel` **[NEW]**
- `UIColor.systemBackground`, `UIColor.secondarySystemBackground`, `UIColor.tertiarySystemBackground` **[NEW]**
- `UIColor.systemGroupedBackground`, `UIColor.secondarySystemGroupedBackground`, `UIColor.tertiarySystemGroupedBackground` **[NEW]**
- `UIColor.systemFill`, `UIColor.secondarySystemFill`, `UIColor.tertiarySystemFill`, `UIColor.quaternarySystemFill` **[NEW]**
- `UIColor.separator`, `UIColor.opaqueSeparator` **[NEW]**
- `UIColor.systemGray`, `systemGray2`…`systemGray6` **[NEW]**
- `UIColor.systemBlue`, `.systemGreen`, `.systemRed`, `.systemOrange`, `.systemYellow`, `.systemPink`, `.systemPurple`, `.systemTeal`, `.systemIndigo` — tint colors, now dynamic **[UPDATED]**
- `UIColor(dynamicProvider:)` **[NEW]** — create dynamic color from Light/Dark provider closure
- `UITraitCollection.userInterfaceStyle` (`.light` / `.dark`) **[NEW]**

### UIKit — Materials and Vibrancy
- `UIBlurEffect.Style.systemThickMaterial`, `.systemMaterial`, `.systemThinMaterial`, `.systemUltraThinMaterial` **[NEW]**
- `UIVibrancyEffect(blurEffect:style:)` **[NEW]** — vibrancy styles for labels, fills, separators

### UIKit — Modal Presentations
- `UIModalPresentationStyle.pageSheet` — card-style (now the default for `present(_:animated:)`) **[UPDATED behavior]**
- `UIAdaptivePresentationControllerDelegate.presentationControllerShouldDismiss(_:)` **[NEW]** — prevent swipe-to-dismiss
- `UIAdaptivePresentationControllerDelegate.presentationControllerDidAttemptToDismiss(_:)` **[NEW]** — called when dismiss is prevented
- `UISheetPresentationController` (related, for full sheet control)
- `isModalInPresentation` on `UIViewController` **[NEW]** — prevent interactive dismissal

### UIKit — Contextual Menus
- `UIContextMenuInteraction` **[NEW]** — interaction class added to any view
- `UIContextMenuInteractionDelegate` **[NEW]** — provide menu configuration
- `UIContextMenuConfiguration` **[NEW]** — configuration with optional preview and action provider
- `UIMenu` **[NEW]** — hierarchical menu with title, image, children
- `UIAction` **[NEW]** — leaf action with title, image, handler; `.destructive` attribute for red styling
- `UIMenuElement.Attributes` **[NEW]** — `.destructive`, `.disabled`, `.hidden`
- `UIViewController.buildMenu(with:)` / app-level menu builder

### Symbols
- `UIImage(systemName:)` **[NEW]** — load an SF Symbol by name
- `UIImage.SymbolConfiguration` **[NEW]** — configure weight, scale, point size
- `UIImageView` / `UIButton` — accept symbol configurations

### Human Interface Guidelines Resources
- HIG: Layout, SF Symbols, Materials, Dark Mode, Multitasking, Modality (linked in Resources section)

## Code Highlights

Loading an SF Symbol:
```swift
let image = UIImage(systemName: "star.fill")
let config = UIImage.SymbolConfiguration(weight: .bold)
let boldStar = UIImage(systemName: "star.fill", withConfiguration: config)
```

Creating a dynamic semantic color:
```swift
let dynamicColor = UIColor { traitCollection in
    traitCollection.userInterfaceStyle == .dark
        ? UIColor(red: 0.9, green: 0.9, blue: 1.0, alpha: 1)
        : UIColor(red: 0.2, green: 0.2, blue: 0.6, alpha: 1)
}
```

Adding a contextual menu to a view:
```swift
let interaction = UIContextMenuInteraction(delegate: self)
view.addInteraction(interaction)

func contextMenuInteraction(_ interaction: UIContextMenuInteraction,
    configurationForMenuAtLocation location: CGPoint) -> UIContextMenuConfiguration? {
    return UIContextMenuConfiguration(identifier: nil, previewProvider: nil) { _ in
        let share = UIAction(title: "Share", image: UIImage(systemName: "square.and.arrow.up")) { _ in }
        let delete = UIAction(title: "Delete", image: UIImage(systemName: "trash"),
                              attributes: .destructive) { _ in }
        return UIMenu(title: "", children: [share, delete])
    }
}
```

## Takeaways
- All apps should adopt Dark Mode using semantic system colors; UIKit controls do this automatically, but custom UI requires audit against the new palette and base/elevated background awareness.
- SF Symbols (1,500+) are vector-based, weight-matched to SF font, and Dynamic Type compatible — they should replace custom glyph assets wherever possible.
- Card-style sheets are now the default modal presentation; add `isModalInPresentation` and the appropriate delegate methods to handle the swipe-to-dismiss gesture correctly.
- Contextual menus should be added to every interactive object; they work on all iOS 13 devices via tap-and-hold and provide a universal power-user shortcut pattern.

---
_Source: WWDC19 Session 808 page (abstract, chapter summaries, code samples, and resource links)._
