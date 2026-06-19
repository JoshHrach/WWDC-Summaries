# Design Foundations: From Idea to Interface
**WWDC25 · Session 359** · [Watch](https://developer.apple.com/videos/play/wwdc2025/359/)

_Platforms:_ iOS, macOS, iPadOS, watchOS (design guidance; no platform-specific API)

## Overview
A practical design walkthrough from Apple's Design Evangelism team, using a fictional vinyl record collection app as the running example. The session covers the four foundational layers of app design — Structure, Navigation, Content, and Visual Design — showing how each builds on the previous to create an experience that feels clear, intuitive, and effortless.

The session is aimed at designers and developers of all skill levels and covers no new APIs; it focuses on design thinking, Human Interface Guidelines application, and the process of iterative improvement.

## Key Topics

### Structure: Information Architecture
Start by listing everything the app does. Then model the user: when and where do they use it, what helps them, what gets in the way? Remove non-essential features, rename ambiguous items, group related items. A clear information architecture sharpens the app's purpose and sets up navigation correctly.

Three questions every screen should answer immediately:
1. **Where am I?** (orientation)
2. **What can I do?** (available actions)
3. **Where can I go from here?** (next steps)

### Navigation: Tab Bar and Toolbar
Use the iOS tab bar for top-level navigation. Each extra tab adds cognitive load — merge sections that naturally belong together. The tab bar is for navigation, not for primary actions; move "Add" into the relevant context (e.g., inside the Records tab). Use SF Symbols for tab icons — they're familiar and consistent across Apple platforms. Rename tabs to direct, explicit labels. Use a toolbar to orient users (title = screen name) and surface screen-specific actions via SF Symbols.

### Content: Organization and Layout
**Progressive disclosure:** Show only what's essential upfront; reveal more on interaction (e.g., expand a section vs. showing all at once). Use consistent layout and toolbar across nested screens.

**Grouping strategies for large content:**
- By **time** (recency, season, events)
- By **progress** (draft, ongoing, continue watching)
- By **patterns/relationships** (genre, style, related items)

**Layout choices:**
- Grid: best for visual browsing (images, products)
- List: best for structured text-heavy content (fast scanning, flexible text length)
- Collection: ideal for large sets of visual items (photos, albums, products) with consistent spacing

Use Apple Design Resources list templates as a starting point rather than designing from scratch.

### Visual Design: Hierarchy, Typography, Color
**Visual hierarchy:** Use size and contrast to guide the eye to what matters most first. Create a clear anchor before adding supporting content.

**System text styles:** Use `title`, `headline`, `body`, `caption` — they automatically support Dynamic Type (accessibility), maintain consistency, and adapt to different screen conditions. Avoid custom fixed font sizes.

**Image + text legibility:** When text overlays images, add a gradient or blur behind text. Use full-bleed images with persistent text regions.

**Semantic colors:** Use system semantic colors (`label`, `secondarySystemBackground`, etc.) for anything that needs to adapt to dark mode, high contrast, and tinted appearances. Use tint/accent color sparingly — for selection states, buttons, and highlights only.

**Custom palette:** Define 4–5 brand colors with explicit rules; use them for decorative and brand elements where dynamic adaptation isn't required.

**Icon and image consistency:** Ensure images share a consistent visual style (color palette, mood). Use bold/expanded font weight for category labels to distinguish them from list text.

## APIs & Frameworks

This is a pure design guidance session. Key Apple design tools and guidelines referenced:
- **Human Interface Guidelines** — primary reference for all guidance in this session
- **SF Symbols** — system icon library for consistent, recognizable tab bar and toolbar icons
- **Apple Design Resources** — templates for Figma, Sketch, Photoshop, Illustrator (icon templates, list templates, component libraries)
- **System text styles** — `title`, `headline`, `body`, `subheadline`, `caption`, etc. (Dynamic Type support)
- **Semantic colors** — `label`, `secondaryLabel`, `systemBackground`, `secondarySystemBackground`, etc.
- **Dynamic Type** — accessible text scaling (handled automatically by system text styles)

## Takeaways
- Build information architecture before navigation — know what the app does and who uses it before deciding on tabs or hierarchy.
- Tab bars are for navigation only; use contextual toolbars for actions.
- Apply progressive disclosure to manage complexity — surface the most important content first and let users drill in.
- Use system text styles and semantic colors as the default; add brand customization on top, not instead of, the system defaults.

---
_Source: WWDC25 Session 359 page (abstract, chapter summaries, transcript, and resource links)._
