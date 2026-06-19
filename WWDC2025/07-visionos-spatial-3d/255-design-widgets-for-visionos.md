# Design Widgets for visionOS
**WWDC25 · Session 255** · [Watch](https://developer.apple.com/videos/play/wwdc2025/255/)

_Platforms:_ visionOS 26

## Overview
Widgets on visionOS 26 are no longer flat UI panels — they become three-dimensional objects anchored in a person's physical space, persisting on walls, desks, and shelves across sessions. This session introduces the four core design principles — persistence, fixed size, customization, and proximity awareness — and walks through the full visual system: Paper and Glass styles, color palettes, frame widths, mounting styles (Elevated / Recessed), and the new proximity-based information density model.

Developers with existing iPad widgets can enable compatibility mode for automatic spatial treatment, or build native visionOS widgets with access to platform-specific sizes and enhanced styling. The new Widgets app on visionOS lets users discover and place widgets from any app.

## Key Topics

### Persistence
- Widgets anchor to **horizontal or vertical surfaces** permanently; horizontal placement tilts the widget toward the viewer for legibility.
- Widgets appear **behind all virtual content**, reinforcing their physical presence.
- Multiple instances of the same widget can coexist in a space; they snap to a familiar grid layout on walls.
- Widgets do not persist in virtual environments — only physical surfaces.

### Fixed Size
- Widget template sizes map to **real-world physical dimensions** (not just points).
- A new **extra-large portrait size** is available natively on visionOS — ideal for statement-piece or artwork-style widgets.
- Users can resize via a corner affordance from 75% to 125% of the template size, so always use high-resolution assets.
- Design with print/wayfinding principles: clear hierarchy, strong typography, legibility at range.

### Customization
- Choose a **style**: **Paper** (print-like, responds to ambient lighting, entire widget dims/brightens) or **Glass** (layered; foreground stays full-color; background adapts to light).
- Glass layer stack: Frame → Backplate → UI Duplicate Layer (shadow depth) → UI Layer (key content) → Coating Layer (reflective finish).
- System provides **14 color palettes** (7 light, 7 dark); developers can opt out of background tinting (but the frame always receives tint).
- **Frame width**: five options (thin to thick), independent of template size.
- **Mounting style**: Elevated (like a picture frame on a wall — default, works on all surfaces) or Recessed (inset into wall — vertical surfaces only, creates depth/window illusion). Developers can opt out of Recessed or make it exclusive (note: exclusive Recessed removes horizontal placement).
- Extend customization with **developer-supplied parameters** surfaced in the configuration UI (e.g., light/dark theme derived from album art, time-of-day adaptation).

### Proximity Awareness
- Two thresholds: **Default** (close-up) and **Simplified** (distant). Widgets adapt layout and information density as the user moves closer or farther.
- Maintain **shared elements** across both thresholds for visual continuity.
- Interactive areas should be easy to target at both distances.
- A tap on a non-interactive widget launches the app as a shortcut.

## APIs & Frameworks

### WidgetKit
- `WidgetKit` — existing framework; visionOS widgets use the same extension model. **[NEW visionOS support]**
- `WidgetConfiguration` — configure Paper vs. Glass style and customization options.
- Compatibility mode — existing iPad widgets automatically receive spatial treatment on visionOS.
- `Updating your widgets for visionOS` — **[NEW]** documentation for porting and native widget development.
- Proximity threshold APIs — **Default** and **Simplified** layout variants. **[NEW]**
- Mounting style control: `.elevated` / `.recessed`. **[NEW]**
- Frame width variants (thin → thick). **[NEW]**

### SwiftUI
- Widget views use SwiftUI; standard SwiftUI modifiers apply within the widget content region.
- Button support inside widgets for lightweight interactions (e.g., revealing live score details). **[NEW]**

## Code Highlights
No code samples were shown in this design session. Implementation guidance is in "What's new in WidgetKit" and the `Updating your widgets for visionOS` documentation.

## Takeaways
- Start by enabling compatibility mode for your existing iPad widgets, then consider native visionOS widgets for richer spatial expression.
- Choose Paper for content that should feel physically integrated (artwork, media posters); choose Glass for information-dense, always-legible data (news headlines, scores).
- Implement both Default and Simplified proximity layouts — even minor density adjustments go a long way for glanceability at room scale.
- Always use high-resolution assets since users can scale widgets up to 125% and view them up close.

---
_Source: WWDC25 Session 255 page (abstract, chapter summaries, and full transcript)._
