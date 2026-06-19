# What's New in iOS 11
**WWDC17 · Session 810** · [Watch](https://developer.apple.com/videos/play/wwdc2017/810/)

_Platforms:_ iOS 11

## Overview
This design-focused session introduces the visual design direction for iOS 11 around a single theme: Wayfinding. Just as physical environments use signage to orient people within a space, digital interfaces require clear visual hierarchy to help users understand where they are and how to navigate. The session focuses on three areas where iOS 11 refines this: large-title navigation bars for top-level orientation, improved typographic hierarchy throughout system apps, and higher-contrast UI controls that create better balance with bolder text.

The session uses real Apple app before/after comparisons (Music, Photos, Calendar, Weather, Clock, Wallet) to illustrate when each treatment is appropriate and when it would be counterproductive. The central message is that bold typography, purposeful color, and filled-in control shapes all serve content legibility — not decoration — and developers should audit their own apps against the same wayfinding criteria.

## Key Topics
- **Wayfinding concept** — digital navigation analogy to physical signage; UI elements should help users orient themselves, not compete with content; clarity, hierarchy, and contrast are the tools
- **Large title navigation bars** — **[NEW in iOS 11]** opt-in large-title variant that collapses to standard navigation bar on scroll; intended for top-level screens with tab bars that have scrollable, similar-layout content (Music, Phone); not appropriate where each tab has a distinct layout (Clock) or where it would compete with unique content
- **When to use large titles** — top level of apps with tabs; secondary levels with dense, homogeneous content (Music drill-down pages); avoid when tab layouts are visually distinct from each other
- **Typographic hierarchy** — three techniques for establishing hierarchy: position (top = more important), size variation (larger = primary), weight/color variation (heavier or colored = primary, gray = secondary); use at least two simultaneously for unambiguous hierarchy
- **Photos section headers** — iOS 10: small, light text; iOS 11: larger section headers with heavier weight and gray secondary text; result: faster scanning and photo location
- **Calendar typography** — increased weight for date text; purposeful color to denote current year/month/date (color conveys meaning, not decoration)
- **Weather typography** — larger, heavier text for temperature and conditions; faster to read at a glance
- **Control contrast updates** — search fields: filled shape with rounded rectangle background (vs. outline); filled button shapes instead of ghost buttons; tab bar glyphs: filled icons with thicker strokes
- **Tab bar landscape layout** — label moves beside glyph (horizontal orientation) in landscape; tab bar height decreases to match landscape navigation bar and toolbar height; glyphs slightly smaller, labels slightly larger than portrait equivalents
- **Tab bar iPad** — labels are larger than on iPhone while glyphs remain the same size
- **Material blur removal** — removed blur materials from contexts where they had no logical layering meaning (Wallet example); blurs should only appear where UI genuinely floats over still-visible content below
- **Overall principle** — remove superfluous UI elements and visual noise to improve content contrast; every element should earn its place by contributing to wayfinding or content clarity

## APIs & Frameworks

### UIKit
- **`UINavigationBar.prefersLargeTitles`** — `Bool`; **[NEW iOS 11]** enables large title display for the navigation bar; set `true` on the navigation bar to opt in
- **`UINavigationItem.largeTitleDisplayMode`** — **[NEW iOS 11]** controls per-view-controller large title behavior: `.automatic` (inherits from previous view controller), `.always` (always show large title), `.never` (always use standard title)
- **`UITabBarItem`** — glyph and label; in iOS 11 landscape, label is positioned beside glyph; glyphs use filled variants; labels use medium weight font
- **`UISearchController`** — integrated into `UINavigationItem.searchController` **[NEW iOS 11]**; search bar appears below large title when embedded in navigation item
- **`UINavigationItem.searchController`** — **[NEW iOS 11]** attaches a `UISearchController` to the navigation bar for large-title-style integration; `hidesSearchBarWhenScrolling` controls visibility on scroll

## Code Highlights

```swift
// Enable large title navigation bar for a top-level tab
navigationController?.navigationBar.prefersLargeTitles = true

// Per-view-controller control: always show large title on the root
navigationItem.largeTitleDisplayMode = .always

// Collapse to standard title on a detail view
navigationItem.largeTitleDisplayMode = .never

// Embed search controller in the navigation item (iOS 11+)
let searchController = UISearchController(searchResultsController: nil)
searchController.searchResultsUpdater = self
navigationItem.searchController = searchController
navigationItem.hidesSearchBarWhenScrolling = false
```

## Takeaways
- Large titles serve wayfinding: they confirm the user is at the top of a content hierarchy and at the top of the scroll position simultaneously; they are not a universal treatment and should be skipped when each tab has a distinct non-list layout.
- Typographic hierarchy in iOS 11 relies on two simultaneous signals (usually weight + color or weight + size) because one signal alone is often insufficient; size, weight, and color should be varied intentionally to denote importance, not applied uniformly.
- Filled-in glyphs and button shapes in tab bars and controls create better contrast against the larger, bolder text introduced in iOS 11; the two changes (bolder type and bolder controls) are designed as a matched pair.
- Removing blur materials from contexts without genuine layering intent reduces visual noise and improves contrast — every non-content visual element should have a logical reason for existing.

---
_Source: WWDC17 Session 810 page (abstract, transcript, and resource links)._
