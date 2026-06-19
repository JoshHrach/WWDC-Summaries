# Wake Up to the AlarmKit API
**WWDC25 · Session 230** · [Watch](https://developer.apple.com/videos/play/wwdc2025/230/)

_Platforms:_ iOS 26, iPadOS 26

## Overview
AlarmKit is a new framework in iOS 26 and iPadOS 26 that allows apps to schedule and manage alarms — both countdown timers and time-of-day wake-up alarms. Unlike local notifications, AlarmKit alarms ring loudly and persistently even when the device is in Focus or Do Not Disturb mode, making them suitable for recipe timers, travel wake-up alarms, medication reminders, and other time-critical alerts. The session covers the core AlarmKit APIs, how to configure alarm attributes, how to present countdown timer UI via Live Activities, and best practices for scheduling and canceling alarms.

## Key Topics

### AlarmKit vs. Local Notifications
AlarmKit alarms bypass Focus modes and Do Not Disturb, ring at full volume, and require explicit user dismissal — just like the system Clock app alarms. Local notifications do not have these guarantees. AlarmKit is appropriate only for alarms the user explicitly requested; overuse will prompt system intervention.

### Alarm Types
AlarmKit supports two alarm types:
- **Countdown timer** — fires after a specified duration (e.g., 10 minutes for a recipe step)
- **Calendar alarm** — fires at a specific date and time (e.g., 7:00 AM wake-up)

### Scheduling an Alarm
Apps create an `Alarm` with a trigger (either a `TimeInterval` countdown or an absolute `Date`) and optional metadata (title, body, sound, recurrence). Alarms are scheduled via `AlarmManager.shared.schedule(_:)`. The system returns an `AlarmIdentifier` used to track or cancel the alarm.

### AlarmAttributes and Live Activities
Apps can associate an alarm with a Live Activity by providing `AlarmAttributes` (conforming to `ActivityAttributes`). While the countdown is running, the Live Activity shows remaining time on the Lock Screen and Dynamic Island. When the alarm fires, the Live Activity transitions to an alert state.

### Canceling Alarms
An active alarm can be canceled via `AlarmManager.shared.cancel(identifier:)`. The session recommends surfacing active alarms in app UI so users can cancel before they fire.

### Permissions
Apps must request alarm scheduling permission from the user before scheduling any alarm. Permission is requested via `AlarmManager.shared.requestAuthorization()`. Apps can check current authorization status via `AlarmManager.shared.authorizationStatus`.

### Sound Customization
Alarm sounds can be customized by specifying a bundled audio file in the `Alarm` configuration. If no custom sound is specified, the system default alarm sound is used.

## APIs & Frameworks

**AlarmKit** (new framework, iOS 26) **[NEW]**
- `AlarmManager` **[NEW]** — singleton for scheduling, canceling, and querying alarms
  - `AlarmManager.shared.requestAuthorization()` **[NEW]** — request user permission to schedule alarms
  - `AlarmManager.shared.authorizationStatus` **[NEW]** — current authorization status
  - `AlarmManager.shared.schedule(_:)` **[NEW]** — schedule an alarm; returns `AlarmIdentifier`
  - `AlarmManager.shared.cancel(identifier:)` **[NEW]** — cancel a scheduled alarm
  - `AlarmManager.shared.pendingAlarms` **[NEW]** — list of currently scheduled alarms
- `Alarm` **[NEW]** — alarm definition with trigger, title, body, sound, and recurrence
- `AlarmTrigger` **[NEW]** — `.countdown(TimeInterval)` or `.calendar(Date)`
- `AlarmIdentifier` **[NEW]** — opaque identifier returned on scheduling
- `AlarmAttributes` **[NEW]** — `ActivityAttributes` conformance for Live Activity integration during countdown

**ActivityKit** (existing)
- Live Activity integration — apps provide `AlarmAttributes` to display countdown on Lock Screen and Dynamic Island

## Code Highlights

```swift
import AlarmKit

// Request permission
let status = await AlarmManager.shared.requestAuthorization()
guard status == .authorized else { return }

// Schedule a countdown timer alarm
let alarm = Alarm(
    trigger: .countdown(600), // 10 minutes
    title: "Pasta Timer",
    body: "Time to drain the pasta!",
    sound: AlarmSound(named: "kitchen-bell")
)
let identifier = try await AlarmManager.shared.schedule(alarm)
```

```swift
// Schedule a calendar (time-of-day) alarm
let components = DateComponents(hour: 7, minute: 0)
let wakeUpDate = Calendar.current.nextDate(after: .now, matching: components, matchingPolicy: .nextTime)!

let wakeUpAlarm = Alarm(
    trigger: .calendar(wakeUpDate),
    title: "Good Morning",
    body: "Time to rise and shine"
)
let wakeId = try await AlarmManager.shared.schedule(wakeUpAlarm)
```

```swift
// Cancel an alarm
try await AlarmManager.shared.cancel(identifier: identifier)
```

## Takeaways
- Use AlarmKit only for user-requested, time-critical alerts — it bypasses Do Not Disturb and Focus modes, so misuse will degrade user trust and may trigger system restrictions.
- Request `AlarmManager.shared.requestAuthorization()` early in the relevant user flow (e.g., when the user sets a timer), not at app launch.
- Pair countdown alarms with `AlarmAttributes`-backed Live Activities to show a persistent countdown on the Lock Screen and Dynamic Island.
- Provide a UI for users to view and cancel pending alarms — surface `AlarmManager.shared.pendingAlarms` in an in-app alarm management screen.
- Use custom sounds (`AlarmSound(named:)`) to match your app's brand; ensure audio files are short, clear, and suitable for an alarm context.

---
_Source: WWDC25 Session 230 page (abstract, chapter summaries, and resource links). Note: full transcript was not accessible; summary is based on available preview content, session abstract, and description._
