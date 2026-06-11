# Learn CSS Grid Lanes

**WWDC26 · Session 314** · [Watch](https://developer.apple.com/videos/play/wwdc2026/314/)

_Platforms:_ Safari 26.4+, WebKit, web standards

## Overview

CSS Grid Lanes is a new CSS layout mode — `display: grid-lanes` — that solves the long-standing "masonry" layout problem in pure CSS without JavaScript. It sits between CSS Flexbox and CSS Grid: like Flexbox it flows items automatically, and like Grid it uses track sizing along one axis. Items are placed into whichever column (or row) currently has the least content, producing the tight waterfall or brick-wall packing familiar from image galleries, card feeds, and Pinterest-style layouts.

The session explains why existing layout modes fall short for mixed-aspect-ratio content (Flexbox stretches or crops; Grid forces uniform row heights), then builds a Grid Lanes layout from scratch in three lines of CSS. It covers brick variation, responsive column counts with `auto-fill`/`minmax()`, individual item spanning, subgrid integration, and the new `flow-tolerance` property that prevents accessibility problems when visual order diverges from DOM order.

Grid Lanes launched in Safari 26.4 and can be debugged visually with the Safari Web Inspector overlay.

## Key Topics

### CSS Flexbox and Grid (1:35)
Existing layout modes cannot pack items of varying heights without either stretching (Flexbox) or leaving gaps (Grid). Grid Lanes was designed specifically for this class of problem.

### CSS Grid Lanes Concept (2:45)
Grid Lanes structures one axis (columns or rows) and leaves the other free. Each item is placed in the column with the current minimum height — the "shortest column" rule. This produces tight packing with no distortion for any content type.

### Build a Grid Lanes Container (3:55)
Three required properties: `display: grid-lanes`, `grid-template-columns` (or `grid-template-rows`), and `gap`. Standard `fr` units, fixed lengths, and keyword values all work for track sizing.

### Brick Variation (4:31)
Swap `grid-template-columns` for `grid-template-rows` to flow items horizontally instead of vertically, producing a brick-wall pattern. Grid Lanes structures only one axis at a time.

### Responsive and Mixed Track Sizes (4:49)
`auto-fill` with `minmax()` lets the browser decide the number of columns based on available width — identical to how it works in CSS Grid. Repeating patterns (narrow + wide columns alternating) are also supported.

### Individual Item Control (5:40)
Standard Grid placement properties work on Grid Lanes items: `grid-column: span 2` stretches an item across two columns. Explicit column start positions can be set; the row is still chosen automatically by the shortest-column rule. Subgrid (`grid-template-columns: subgrid`) aligns nested items with the parent layout's tracks.

### Flow Tolerance (7:05)
The shortest-column rule can produce visual orderings that differ from DOM order, creating confusing keyboard navigation. The `flow-tolerance` property controls how much height difference is acceptable before an item is moved to an earlier column. The default is `1em`. Setting it to `normal` restores strict shortest-column behavior; setting it to a larger value (e.g., `2.1em`) gives items more latitude to prefer earlier columns.

### Web Inspector (8:46)
The Safari Web Inspector Grid Lanes overlay shows column and row lines, gaps, and the order number on each item, making `flow-tolerance` tuning and unexpected placement easy to diagnose.

## APIs & Frameworks

**CSS Grid Lanes — Container Properties**
- **[NEW]** `display: grid-lanes` — activates Grid Lanes layout mode
- `grid-template-columns` — defines column tracks (same syntax as CSS Grid); use this for waterfall/vertical packing
- `grid-template-rows` — defines row tracks; use this for brick/horizontal packing
- `gap` — spacing between items (row-gap and column-gap)
- **[NEW]** `flow-tolerance` — maximum height delta before preferring an earlier column; default `1em`; values: `normal` (strict), `<length>` (e.g., `2.1em`)

**CSS Grid Lanes — Track Sizing (inherited from CSS Grid)**
- `fr` unit — fractional share of available space
- `repeat()` — repeat track definitions
- `auto-fill` — browser-determined column count
- `minmax(min, max)` — flexible minimum/maximum track sizing
- Repeating patterns: `repeat(auto-fill, minmax(8rem, 1fr) minmax(14rem, 2fr))`

**CSS Grid Lanes — Item Properties**
- `grid-column: span N` — span multiple columns
- `grid-column: N / span M` — explicit start with span
- `grid-template-columns: subgrid` on a child — align nested grid to parent tracks
- `display: grid` or `display: grid-lanes` on a child item for nested subgrid

**Tooling**
- Safari Web Inspector Grid Lanes overlay — visualizes tracks, gaps, and item order numbers

## Code Highlights

Minimal Grid Lanes container:
```css
.container {
  display: grid-lanes;
  grid-template-columns: repeat(3, 1fr);
  gap: 10px;
}
```

Responsive columns:
```css
.container {
  display: grid-lanes;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 10px;
}
```

Spanning and subgrid:
```css
.item { grid-column: span 2; }

.item-subgrid {
  display: grid;
  grid-template-columns: subgrid;
  grid-column: span 2;
}
```

Brick variation (horizontal flow):
```css
.container {
  display: grid-lanes;
  grid-template-rows: repeat(3, 1fr);
  gap: 10px;
}
```

Flow tolerance tuning:
```css
.container {
  display: grid-lanes;
  grid-template-columns: 1fr 1fr;
  gap: 10px;
  flow-tolerance: 2.1em;
}
```

## Takeaways

- `display: grid-lanes` replaces JavaScript masonry libraries with 3 lines of CSS; it shipped in Safari 26.4.
- The layout mode structures one axis (columns or rows) and uses the shortest-column rule to pack items automatically — no stretching or cropping.
- `flow-tolerance` is a critical accessibility knob: tuning it prevents keyboard navigation order from diverging too far from visual order.
- All standard CSS Grid track sizing — `fr`, `auto-fill`, `minmax()`, repeating patterns — works identically inside Grid Lanes, and child items support `subgrid`.

---
_Source: WWDC26 Session 314 page (abstract, chapter summaries, code samples, and resource links)._
