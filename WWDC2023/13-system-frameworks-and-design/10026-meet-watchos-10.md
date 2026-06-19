# Meet watchOS 10
**WWDC23 · Session 10026** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10026/)

_Platforms:_ watchOS 10

## Overview
watchOS 10 represents the most significant redesign of Apple Watch since its introduction in 2014. This session from the Apple Design Team tours the new user interface paradigm: a Smart Stack of timely widgets surfaced via the Digital Crown from the watch face, a redesigned Home Screen with vertical Digital Crown navigation, and a comprehensive visual overhaul of every system app built around three core design principles — focused utility, glanceability, and consistent navigation.

The session is design-focused with no code samples; it establishes the visual language and navigation patterns developers must adopt when building watchOS 10 apps. The companion sessions "Design and build apps for watchOS 10" and "Update your app for watchOS 10" cover implementation details.

## Key Topics

### Design Principles
Three foundational principles guide every watchOS 10 design decision:
1. **Focused and specialized** – each app should do one thing extremely well
2. **Glanceable and brief** – screens must convey information immediately; interactions are short
3. **Clear and consistent controls** – standard layout, control placement, and navigation patterns reduce the learning curve

### New System UI
- **Smart Stack** – timely, relevant widgets surfaced directly from the watch face by turning the Digital Crown; widgets surface at contextually appropriate times
- **Redesigned Home Screen** – vertical Digital Crown scroll through alphabetically organized app icons (replaces the honeycomb grid or list)
- **Control Center** – single Side Button press from anywhere; Wallet with double Side Button press

### Layout System
Three foundational screen layouts adapt automatically to all Apple Watch sizes supported in watchOS 10:
- **Dial** – circular gauge or ring-centric layout (e.g., Activity ring detail)
- **Infographic** – data-dense metric display with icons and values (e.g., Weather, Workout stats)
- **List** – vertical scrollable list of items (e.g., Mail, Contacts)

All layouts define standardized sizes and placements for controls, labels, and content to ensure cross-app consistency and ergonomic tap targets.

### Background Colors and Content
- Background colors convey app identity and state (e.g., Activity ring color tints, Weather conditions, stock direction)
- Background content should provide additional utility — not decoration alone
- Can range from a simple color or gradient to an animated element communicating real-time data
- Use color to communicate sense of place and help users with navigation

### Materials
- watchOS 10 introduces translucent layering across system elements
- Status Bar uses a material so content scrolling beneath it remains readable
- Distinct functional layers use translucency to establish visual hierarchy
- Semantic system colors automatically adapt contrast when placed on vibrant background materials

### Navigation Patterns (New in watchOS 10)

**Vertical Pagination**
- Replaces horizontal pagination (horizontal swipe is harder on Apple Watch)
- Digital Crown navigates between discrete full-screen pages within an app
- Page indicator appears aligned to Digital Crown and adapts to any background
- Shared elements across pages (e.g., Activity Rings) should animate smoothly between pages using scale, position, and information density changes — "object permanence"
- Individual pages should ideally fit a single screen height; scroll views within a page should follow fixed-height pages and be used sparingly

**Source List**
- New pattern for apps with multiple entities of the same data type (e.g., World Clock locations, Contacts)
- App opens directly to the detail view for the default entity
- Source List Button (upper-left) lets users switch between entities
- Two-level design (source list + detail) replaces deeper hierarchies in most cases

**Refined Hierarchical Navigation**
- Still supported but should be used only when source list or vertical pagination don't fit
- New animation makes navigation direction obvious
- Suitable for apps with naturally deep hierarchies (Settings, Mail)

**Digital Crown as Data Inspector**
- In single-view contexts (no pagination needed), the Crown can scroll through data dimensions — e.g., World Clock advances the time of day at a selected location to compare with local time

### Design Guidance Summary
- Use standard layouts, controls, label sizes, semantic colors, and materials
- Keep navigation shallow; prefer source lists and vertical pages over deep hierarchies
- Limit pages to single screen height where possible; use scroll views only for overflowing secondary content after fixed-height primary pages
- Animate shared elements across pages for visual continuity
- Background content must serve a functional purpose, not just aesthetics

## APIs & Frameworks

- **watchOS 10** – major platform release; new UI paradigm and system chrome
- **Smart Stack** **[NEW]** – widget surface accessible from the watch face via Digital Crown scroll
- **Vertical Pagination** **[NEW]** – SwiftUI `TabView` with `.tabViewStyle(.verticalPage)` (implementation covered in "Update your app for watchOS 10")
- **Source List** **[NEW]** – `NavigationSplitView`-based pattern for entity switching (details in "Design and build apps for watchOS 10")
- **Digital Crown navigation** – `focusable()` + `digitalCrownRotation` in SwiftUI; also used for paginated navigation
- **Dial, Infographic, List layouts** – design system; implemented via SwiftUI layout containers (covered in companion sessions)
- Translucent materials – `Material` type in SwiftUI; semantic colors adapt automatically
- Semantic system colors – `Color.primary`, `Color.secondary`, etc.; adapt contrast on vibrant backgrounds
- `WidgetKit` – powers Smart Stack widgets; `accessoryCircular`, `accessoryRectangular`, `accessoryInline` families
- Human Interface Guidelines: watchOS – design reference for watchOS 10 layout system

## Code Highlights

No code samples in this session — it is a design overview. For implementation, refer to:
- **"Update your app for watchOS 10"** (session 10031) – adopting vertical pagination, toolbar placement, NavigationSplitView
- **"Design and build apps for watchOS 10"** (session 10138) – full build walkthrough
- **"Design widgets for the Smart Stack on Apple Watch"** (session 10309) – Smart Stack widget implementation

## Takeaways
- watchOS 10 is a full platform visual redesign; apps that follow the new layout system, navigation patterns, and material guidelines will feel native and consistent; apps that do not will feel out of place
- The three new navigation patterns — vertical pagination, source list, and refined hierarchical navigation — cover the vast majority of watchOS app structures; most apps should move away from horizontal pagination and deep hierarchies
- Smart Stack widgets are a new first-class surface for surfacing timely, contextual content from your app directly on the watch face; building a widget is a high-impact investment for watchOS 10
- Background content (colors, gradients, animations) should always carry functional meaning — color helps users identify their location within an app, not merely decorate screens

---
_Source: WWDC23 Session 10026 page (abstract, chapter summaries, and transcript)._
