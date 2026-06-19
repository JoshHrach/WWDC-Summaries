# Apple Watch Design Tips and Tricks
**WWDC15 · Session 805** · [Watch](https://developer.apple.com/videos/play/wwdc2015/805/)

_Platforms:_ watchOS 2

## Overview
This session presents the top 10 common design pitfalls developers encounter when building Apple Watch apps, covering performance, information architecture, navigation, labeling, menus, interactivity, tap targets, legibility, visual design, and app icon design. Presenters Mike Stern and Rachel Roth from Apple's UX Evangelism team walk through concrete examples showing what goes wrong and how to fix it.

The session reinforces three core Apple Watch design themes introduced at WWDC15: Personal Communication (intimate and personal experience), Holistic Design (blurring hardware/software boundaries), and Lightweight Interaction (quick looks and fast interactions). Every pitfall discussed ties back to these principles and the unique constraints of a wrist-worn device with brief interaction windows.

Practical guidance covers graphic optimization for file size and performance, choosing between page-based and hierarchical navigation structures, making buttons look and feel tappable, Dynamic Type implementation, and adapting iOS app icons for the circular Watch icon format.

## Key Topics

### Pitfall 1 — Slow Apps
- Use **progressive loading**: show the page immediately with available data; fill in slower content as it arrives.
- Hold space open with **placeholder graphics** to avoid jarring layout shifts when content loads in.
- Order content so slower-loading items (photos) appear lower on the page.
- Optimize graphics: JPEG compression (find quality/size balance), 8-bit PNG palettes for non-photo graphics (up to 10x smaller), avoid unnecessary alpha channels, size images to the display size needed.

### Pitfall 2 — Complex Apps
- Apple Watch is not a miniature iPhone. Focus on the essential subset of functionality relevant to a wrist context.
- Watch apps should complement, not duplicate, the iOS companion app.
- Consider extending iPhone capabilities in new ways (e.g., Camera remote viewfinder).

### Pitfall 3 — Tedious Navigation
- Two structural models: **page-based** (swipe horizontally between peer screens) and **hierarchical** (tap to drill into detail). These cannot be mixed in watchOS.
- Page-based works best for flat, small collections of peer content (Weather app).
- Hierarchical works best for larger or more complex data sets (Stocks app).
- Avoid overusing modal sheets for navigation — they hide the time and cause jarring transitions.
- Hierarchical apps: max 2 levels, 3 at most. Page-based: keep page count low.

### Pitfall 4 — Confusing Labels
- The chevron/title area in a hierarchical app is a **page title**, not a Back button label — avoid putting "Back" there.
- Modal sheet dismiss controls must look actionable (use "Done", "Close", "Cancel") — not plain titles.

### Pitfall 5 — Menu for Primary Navigation
- Force Touch menus are for **contextual actions** and view-mode preferences, not app navigation.
- Using menus for navigation removes visual cues (hierarchy arrow, page indicator dots) that orient users in the app structure.
- Avoid designing app UI to visually mimic the menu appearance (dark icons, gray circles) — users will think they accidentally invoked the menu.

### Pitfall 6 — Buttons That Don't Look Like Buttons
- Apple Watch design language uses **rounded rectangles** and **circles** to signal interactivity (not position in toolbars or chevrons, as on iOS).
- Page titles use the Global Tint color for branding — don't rely on color alone to signal tappability.
- Avoid using rounded rectangle/circle shapes for non-interactive content.
- Use separator lines (not shaded backgrounds) to group related non-interactive content.

### Pitfall 7 — Inadequate Tap Targets
- Minimum circular button: 80×80 px (42 mm Watch), 75×75 px (38 mm Watch).
- Minimum rectangular button height: 53 px (42 mm), 50 px (38 mm); system typically uses 80 px+.
- Extend buttons to the full canvas width even for short labels.
- Maximum three buttons side by side (only with single characters or icons); two buttons with text is the practical limit.

### Pitfall 8 — Legibility
- Test on an actual Watch on the wrist, in motion, outdoors — not just in Sketch or the simulator.
- Use **SF Compact** (the system font) for maximum legibility; custom fonts at equivalent sizes are harder to read.
- Use the five system **text styles** (Headline, Body, Caption 1, Caption 2, Footnote) to get Dynamic Type automatically.
- Dynamic Type adjusts size, leading, and tracking per the user's preferred reading size; custom fonts must handle these adjustments manually.
- Bigger is better — do not use Footnote for all text just because it is the smallest style.

### Pitfall 9 — Misuse of Color and Padding
- Avoid gratuitously bright background colors — they clash with the black status bar and make the screen feel smaller.
- Do not inset content with padding; the hardware bezel serves as the visual padding.
- Background photos increase file size and hurt legibility; use only when compositionally justified.
- Black backgrounds blend seamlessly with the bezel and let content stand out.
- Carry branding through the Global Tint color, typography, tone, and character rather than background color.

### Pitfall 10 — Hard-to-Find App Icons
- Apple Watch icons are **circular** and **unlabeled** — recognition depends entirely on the icon.
- Best approach: directly adapt the iOS icon into a circle if the shape reads well.
- If adaptation is needed: simplify shapes, remove or abbreviate text, enlarge key elements, retain the brand's color palette and visual style.
- Sometimes a complementary icon concept is better than a literal circular crop (e.g., Camera app's shutter-button icon, Sky Guide's astronomical event icon).
- Watch and iOS icons need not be identical twins but should look like siblings.

## APIs & Frameworks

- `WatchKit` — overall framework for watchOS app development **[NEW in watchOS 2: native apps]**
- `WKInterfaceController` — page-based and hierarchical navigation root
- `WKInterfaceTable` — list-based hierarchical navigation
- `WKInterfaceImage` — progressive image loading / placeholder support
- `WKInterfaceSeparator` — separator line for grouping non-interactive content
- `WKInterfaceButton` — rounded-rectangle and full-width tap targets
- `WKInterfaceGroup` — layout container; circular button shape achieved via `cornerRadius`
- `WKInterfaceLabel` — text display with text-style support
- **Dynamic Type text styles**: `.headline`, `.body`, `.caption1`, `.caption2`, `.footnote`
- `UIFont.preferredFont(forTextStyle:)` (via WatchKit text style mapping)
- **Global Tint color** — set in the WatchKit extension's asset catalog / `Info.plist`
- **Force Touch / Digital Crown / Taptic Engine** — hardware capabilities to design around holistically
- SF Compact (system font for Apple Watch) **[NEW]**
- JPEG compression / 8-bit PNG optimization — file-size guidance for Watch graphics

## Code Highlights

Dynamic Type text style usage (conceptual — WatchKit sets styles via Interface Builder attributes):
```swift
// In WatchKit, set the text style in storyboard or via:
label.setTextStyle(.body) // uses SF Compact, respects user's preferred reading size
```

Circular button minimum sizes (design constants):
```
42 mm Watch: circular controls ≥ 80 × 80 pt
38 mm Watch: circular controls ≥ 75 × 75 pt
Rectangular button minimum height: 53 pt (42 mm) / 50 pt (38 mm)
```

## Takeaways
- Design for performance first: progressive loading and optimized graphics directly affect perceived app speed on the Watch.
- Choose the correct navigation structure (page-based vs. hierarchical) for your data set and do not mix them — the constraint is a watchOS architectural limitation, not just a guideline.
- Use the hardware bezel as your padding and a black background as the canvas — this makes Watch apps feel native and integrated.
- Every design decision (button shape, font size, icon simplicity) must account for split-second glance interactions on a wrist in motion.

---
_Source: WWDC15 Session 805 page (abstract, chapter summaries, code samples, and resource links)._
