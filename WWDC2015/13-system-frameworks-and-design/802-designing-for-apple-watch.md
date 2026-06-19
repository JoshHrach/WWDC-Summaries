# Designing for Apple Watch
**WWDC15 · Session 802** · [Watch](https://developer.apple.com/videos/play/wwdc2015/802/)

_Platforms:_ watchOS 2

## Overview
This foundational design session by Mike Stern (UX Evangelist, Apple) establishes the three core design themes for Apple Watch: Personal Communication, Holistic Design, and Lightweight Interaction. It explains why Apple Watch requires a fundamentally different mindset from iOS design, covering the unique hardware inputs (Digital Crown, Force Touch, Taptic Engine), how to think about notifications, Glances, and apps, and how to bring beauty and animation to small-screen wearable experiences.

The session emphasizes that Apple Watch apps must complement — not replicate — their iPhone counterparts, focusing on the single or few tasks that genuinely benefit from wrist-level quick access. Target interaction times are five seconds or less, which demands radical focus on essential content and actions.

watchOS 2 introduces native app support, WKInterfacePicker, runtime animations, HealthKit workout sessions, and programmatic haptic feedback — all of which are covered in the context of good design practice.

## Key Topics

### Theme 1: Personal Communication
- **Considerate notifications**: Tapping a wrist is highly disruptive; over-notification causes users to disable your app entirely. Send only timely, high-value alerts.
- **Relevance via context**: Use location (Invoice2Go billing start/stop) and time (American Airlines departure reminders) to send the right notification at the right moment.
- **Explicit vs. implicit preferences**: Settings let users specify preferences; observing usage patterns (Fitness app reorders workout types by frequency, remembers last goal selection) tailors the experience automatically without configuration.
- **Conciseness**: Get to the point — interactions are measured in seconds.

### Theme 2: Holistic Design
- **Black background = seamless bezel integration**: The Watch bezel was designed to match the display; black backgrounds make the hardware and software merge.
- **Force Touch menus**: Best used for contextual actions (relevant to current screen or whole app), view mode preferences, and global controls. Do NOT use for critical-path actions (those must be inline). Do NOT use as primary navigation.
- **Digital Crown**: Enables fluid scrolling — no need to cram everything on screen at once. Allow content to scroll freely.
- **WKInterfacePicker** (new in watchOS 2): Three styles — Stack (image carousel), List (text/numeric), Sequence (image sequence with optional coordination). Picker outlines show focus; outlines required when multiple Pickers are on screen or when screen also scrolls. Per-item captions; same caption on all items becomes a Picker label. Contextual/scroll indicator shows position and count.
- **Taptic Engine** (programmatic access new in watchOS 2): Nine haptic types:
  - Notification — for incoming alerts
  - DirectionUp / DirectionDown — value crossing a significant threshold
  - Success — action completed successfully
  - Failure — action failed
  - Start / Stop — beginning or ending a timed activity
  - Click — precise dial-increment sensation (use very sparingly; haptics cannot overlap)
  - Use haptics sparingly; overuse diminishes their meaning and effectiveness.

### Theme 3: Lightweight Interaction
- Target interaction time: **~5 seconds or less**.
- **Glances**: Scannable summaries of the most frequently needed information; show only the most essential data; deep-link into the app for more detail; use left-aligned text; built from template pairs (12 upper × 24 lower templates).
- **Notifications**: Minimum words, maximum clarity; use graphics when they communicate faster than text; interactive actions enable triage; show only the most likely action(s).
- **Apps**: Complement rather than duplicate the iPhone app; select one or two key functions; implement Handoff so users can seamlessly continue on iPhone (show Handoff lock-screen icon education only if needed).
- **HealthKit Workout Sessions** (new in watchOS 2): Keeps activity tracker apps foregrounded while a session is ongoing (user raises wrist → returns to the activity app, not clock face). User must deliberately start and stop; UI must clearly indicate active session state.

### Beauty and Animation
- Runtime animatable properties (new in watchOS 2): height, width, insets, alignment, background color, tint color, opacity — all with easing applied.
- Animation uses: bar charts, data transitions, number animations, coordinated image sequences.
- Animate with restraint — animations must feel fast, never blocking.

## APIs & Frameworks

- `WatchKit` framework
- `WKInterfaceController` — page-based and hierarchical app structure
- `WKInterfaceButton` — inline critical-path actions
- `WKInterfaceLabel` — text display
- `WKInterfaceImage` — image and animation display
- `WKInterfaceGroup` — layout and background color animations **[NEW: animatable in watchOS 2]**
- `WKInterfacePicker` **[NEW]** — Digital Crown-driven selection control with Stack, List, and Sequence styles; `WKPickerItem`, `setSelectedItemIndex(_:)`, `focus()`, `resignFocus()`
- `WKInterfaceDevice.playHaptic(_:)` **[NEW]** — programmatic Taptic Engine access
- `WKHapticType` **[NEW]**: `.notification`, `.directionUp`, `.directionDown`, `.success`, `.failure`, `.start`, `.stop`, `.click`
- `HKWorkoutSession` (HealthKit) **[NEW in watchOS 2]** — keeps app foregrounded during tracked workout
- `HKWorkoutSessionDelegate` **[NEW]**
- `HKWorkoutActivityType` — specifies workout type for session
- `WKExtensionDelegate` — extension lifecycle
- `WKAlertAction`, `WKAlertControllerStyle` — modal sheet dismiss actions
- Glance template system — upper and lower template pairs (12 × 24 options)
- Force Touch / `WKInterfaceMenu`, `WKInterfaceMenuItem` — contextual action menus
- Handoff / `NSUserActivity` — cross-device continuation from Watch to iPhone
- SF Compact (system font for watchOS) — `UIFont.preferredFont(forTextStyle:)` with watchOS text styles

## Code Highlights

Triggering haptic feedback:
```swift
WKInterfaceDevice.current().play(.success)
WKInterfaceDevice.current().play(.notification)
```

Starting a HealthKit workout session to stay foregrounded:
```swift
let config = HKWorkoutConfiguration()
config.activityType = .running
config.locationType = .outdoor
let session = try HKWorkoutSession(configuration: config)
session.delegate = self
healthStore.start(session)
```

Animating a WatchKit interface property:
```swift
animate(withDuration: 0.3) {
    self.progressGroup.setHeight(newHeight)
    self.progressGroup.setBackgroundColor(.green)
}
```

## Takeaways
- Apple Watch design demands extreme focus: five-second interaction windows require that only the most essential content and actions surface at the wrist.
- The three themes — Personal Communication, Holistic Design, Lightweight Interaction — are the lens through which every design decision should be evaluated.
- watchOS 2's WKInterfacePicker, runtime animations, and haptic API unlock rich, expressive experiences that were previously impossible in WatchKit.
- HealthKit workout sessions fundamentally change foreground behavior for fitness apps — use them correctly to ensure users remain in control.

---
_Source: WWDC15 Session 802 page (abstract, chapter summaries, code samples, and resource links)._
