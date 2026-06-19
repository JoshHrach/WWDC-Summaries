# Design Dynamic Live Activities
**WWDC23 · Session 10194** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10194/)

_Platforms:_ iOS 17, iPadOS 17

## Overview
Live Activities let apps display continuously updating information in key system locations — the Lock Screen, StandBy, and the Dynamic Island — without requiring users to open the app. This session covers the design principles behind creating visually rich, contextually appropriate Live Activity layouts that feel native to each surface they appear on.

The session walks through Lock Screen presentation, the new StandBy ambient display mode, and the Dynamic Island in detail, including all three size classes (Compact, Expanded, and Minimal). It emphasizes that Live Activities are a new paradigm distinct from notifications and require purpose-built layouts that reflect the visual identity of each app.

For developers, this session is the companion design guide to the ActivityKit engineering sessions, translating system constraints (margins, sizing, transitions, alerting behavior) into actionable guidelines that ensure Live Activities feel polished and purposeful across the system.

## Key Topics

### Design Principles for Live Activities
Live Activities use rich graphical layouts and update seamlessly inline. They are suitable for events lasting minutes to a couple of hours: sports, ride-sharing, delivery tracking, live workouts.

### Lock Screen
- Share 14-point margins with notifications for visual rhythm alignment.
- Create unique layouts specific to the information being displayed — do not replicate notification layouts.
- Prioritize information hierarchy: make the most important data largest and most visible.
- Interactive buttons (via ButtonKit) are allowed but should only control essential aspects of the activity.
- Reflect the app's visual personality (colors, iconography, typeface) and draw inspiration from the app icon.
- Maintain light/dark color consistency if breaking it disrupts the visual association with the app icon.
- Integrate brand logo marks directly into the layout — do not use the full app icon container.
- Check that the auto-generated dismiss button colors match the Live Activity design.
- Minimize height; dynamically adjust height based on available information (e.g., show fewer details while searching for a driver, expand after confirmation).
- Use numeric content transitions for counting up/down, content replace transitions for graphic swaps.
- Animate element position changes (e.g., map pins updating location) for richer visual representation.
- Alert when critical updates occur; alert should emphasize the triggering information.
- Remove ended Live Activities promptly to avoid clutter.

### StandBy
- StandBy scales the Live Activity layout 200% and extends the background color to fill the screen.
- Avoid graphic elements drawn to the edges of the layout — they will be clipped; use dividing lines or containing shapes instead.
- Consider removing the background for a fully ambient, borderless look at a slightly larger scale.
- Ensure all images are high enough resolution for 2× display.
- Check color contrast in night mode (red tint) which is applied automatically.

### Dynamic Island
- The Dynamic Island is a unified system layer for alerts and activity indicators.
- Design must harmonize with the island's organic, rounded shape using concentric shapes and even margins.
- Use the blur trick to verify visual mass/centroid is concentric with the outer border.
- Three size classes: **Compact** (most common, informational at a glance), **Expanded** (triggered by press or alert, more content and controls), **Minimal** (when multiple sessions compete for space).
- Compact: keep it narrow, content snug to the sensor, no wasted space. Show essential, identity-reinforcing information.
- Expanded: bring app character into the island; avoid a "forehead" at the top — hug the sensor tightly. Maintain relative placement coherence between Compact and Expanded.
- Minimal: convey session state even in tiny form; do not just show a logo.
- Prefer using the Dynamic Island for alerts over push notifications when the app has an active session.

## APIs & Frameworks

### ActivityKit **[NEW]**
- `ActivityKit` framework — manages the lifecycle of Live Activities
- `Activity` — the main type for starting, updating, and ending a Live Activity **[NEW]**
- `ActivityAttributes` protocol — defines the static and dynamic content of a Live Activity **[NEW]**
- `ActivityContent` — wraps the dynamic state of a Live Activity **[NEW]**

### WidgetKit / SwiftUI (Live Activity rendering)
- `ActivityConfiguration` — the widget configuration type for Live Activities **[NEW]**
- `LockScreenRegion` — context for Lock Screen rendering **[NEW]**
- `DynamicIsland` — builder for Dynamic Island presentations **[NEW]**
- `DynamicIslandExpandedRegion` — regions within an expanded Dynamic Island view **[NEW]**
- Numeric content transition (`.numericText()`) — animates numeric changes **[NEW]**
- Content replace transition — animates graphical element swaps
- `ButtonStyle` / interactive buttons in Live Activities **[NEW]**

### Human Interface Guidelines
- [Live Activities HIG](https://developer.apple.com/design/human-interface-guidelines/live-activities)
- [Displaying live data with Live Activities](https://developer.apple.com/documentation/ActivityKit/displaying-live-data-with-live-activities)

## Code Highlights
No code samples were shown in this design-focused session. See the engineering companion sessions (Meet ActivityKit, Update Live Activities with push notifications) for implementation details.

## Takeaways
- Design bespoke graphical layouts for each Live Activity surface (Lock Screen, StandBy, Dynamic Island) — do not reuse notification layouts.
- Dynamically adjust layout height to show only relevant information at each moment, with smooth transitions.
- Make Dynamic Island elements concentric with the island's rounded shape, maintaining consistent visual identity between Compact and Expanded states.
- Always respect the dismiss button auto-generated from your background/foreground colors and verify it looks correct in your design.

---
_Source: WWDC23 Session 10194 page (abstract, chapter summaries, code samples, and resource links)._
