# Explore navigation design for iOS
**WWDC22 · Session 10001** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10001/)

_Platforms:_ iOS 16, iPadOS 16

## Overview
Good navigation is invisible — when it works well, people focus on content rather than on the structure. This design session from an Apple Evangelist covers the two most fundamental iOS navigation patterns: tab bars (for top-level information hierarchy) and screen transitions (push navigation vs. modal presentation). The session diagnoses common mistakes — overloaded first tabs, redundant "Home" tabs, forced tab-switching, hiding tab bars, and misused modals — and provides concrete design principles for each.

## Key Topics

**Tab bars as information hierarchy** — The tab bar represents the top level of an app's content hierarchy. Each tab should be a distinct, self-contained content area that makes sense on its own. Balance features across tabs rather than loading everything into the first tab. Common antipatterns: "everything in one tab," a "Home" tab that duplicates functionality from other tabs, and forcing automatic tab-switching when a user taps an element.

**Persistent tab bar** — The tab bar should always be visible throughout navigation, including when pushing deep into a hierarchy. Persistent access allows users to compare content across multiple tabs while preserving state in each. Never hide or remove the tab bar during hierarchical navigation.

**Clear, concise tab labels** — Labels should be descriptive enough that users understand the content area without seeing any of the content. Use noun-form labels that describe content categories (e.g., "Library," "Itinerary") rather than generic labels.

**Hierarchical (push) navigation** — A push transition slides the new view in from right (left in RTL languages), directly representing drilling into a hierarchy. Use push for: traversing deeper into content, workflows where users switch between views frequently, and whenever a disclosure indicator (chevron) is present. The navigation bar's back button should always show the title of the parent screen. The tab bar must remain visible.

**Modal presentations** — Modals slide in from the bottom and intentionally cover the tab bar to create focus on a self-contained task. Use modals for: simple tasks (creating an event), multi-step tasks (adding a transit card), and full-screen content with minimal navigation (viewing an article or video). Never present modals from anywhere other than the bottom of the screen.

**Modal navigation bar anatomy** — Title: orients the user to the screen content. Right button: the preferred action (bold, affirmative verb like "Add" or "Save"; inactive if required fields are empty). Left button: "Cancel" (shows an action sheet warning if data will be lost). Use the close (X) symbol only for content with minimal interaction and no user input. Avoid stacking multiple modal layers; limit modals-over-modals.

## APIs & Frameworks

This is a design session with no code samples. Referenced UI components and patterns from UIKit and HIG:

- `UITabBarController` / `TabView` — top-level navigation via tab bar
- `UINavigationController` / `NavigationStack` — push navigation
- `UINavigationBar` — back button label should reflect parent screen title
- `UIViewController.present(_:animated:)` — modal presentation (always from bottom on iOS)
- Disclosure indicator (chevron `>`) — signals push navigation
- Action sheet — confirm cancellation when user has unsaved input in a modal
- `UITabBar` — must remain persistent throughout navigation depth

## Code Highlights

No code samples were included in this session.

## Takeaways
- Each tab on a tab bar should represent a distinct, independently meaningful section of your app; if you can't clearly describe what belongs in a tab, it's a sign the tab's scope is too broad.
- Never force automatic tab-switching when a user taps content — it disorients users and breaks their mental model of the app's hierarchy.
- Use push navigation to traverse your content hierarchy (drill-down); use modals only to isolate a self-contained task that doesn't require accessing the rest of the app.
- Always keep the tab bar visible during push navigation; always hide it during modal presentation (this contrast is intentional and communicates the difference between exploration and focused task completion).

---
_Source: WWDC22 Session 10001 page (abstract, chapter summaries, and resource links)._
