# Extend Your App's Controls Across the System
**WWDC24 · Session 10157** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10157/)

_Platforms:_ iOS 18, iPadOS 18

## Overview
iOS 18 introduces **Controls** — a new WidgetKit extension point that places app actions in Control Center, the Lock Screen, and the Action Button. Controls are action-focused (not information-focused) and use App Intents to perform work. The session builds a productivity timer control from scratch: a `ControlWidgetToggle` that starts/stops a timer, displays a live activity, syncs state asynchronously from a server, supports user configuration (work vs. personal timer), and is polished with custom action hints, status text, display name, and description.

## Key Topics

### What Controls Are
Controls appear in system spaces (Control Center, Lock Screen, Action Button) alongside built-in toggles. They come in two patterns: **toggle** (on/off boolean state with immediate action) and **button** (discrete one-time action). Controls can be displayed in three sizes in Control Center; only the symbol is shown in the small size and on the Lock Screen, so value text and title may be hidden. Apps provide symbol, title, tint color, and value text; the system handles the chrome.

### Build a Control: ControlWidget + StaticControlConfiguration
A type conforming to `ControlWidget` defines the control. `StaticControlConfiguration` handles non-configurable controls; it takes a `kind` string (unique identifier) and a closure providing a `ControlWidgetToggle` or `ControlWidgetButton`. The toggle takes a title, `isOn` state, action App Intent, and a label closure that maps the state to a symbol (or `Label` for custom value text).

### Customizing Appearance
Different symbols for on/off states use the `isOn` closure argument. Custom value text (replacing the default "on"/"off") uses a `Label` with both text and `systemImage`. A custom tint color via `.tint(_:)` applies to the symbol and value text when the toggle is on.

### Updating Toggle State
Three triggers cause a reload: (1) the control's App Intent `perform()` completes — system reloads automatically; (2) the app calls `ControlCenter.shared.reloadControls(ofKind:)` when state changes from within the app; (3) a push notification via the push handling API for off-device state changes. Enable **WidgetKit Developer Mode** in Developer Settings during development to remove system throttling.

### ControlValueProvider for Async State
When state must be fetched asynchronously (e.g., from a server), implement `ControlValueProvider` with `currentValue() async throws` and a `previewValue` (used in the gallery and Lock Screen customization before the user adds the control — should correspond to the off state). Use a `StaticControlConfiguration` initializer that accepts a `provider:` parameter.

### Making Controls Configurable
To let users choose which timer the control acts on, implement `AppIntentControlValueProvider` (instead of `ControlValueProvider`). The provider's `currentValue(configuration:)` receives a user-selected App Intent configuration. Use `AppIntentControlConfiguration` in place of `StaticControlConfiguration`. Add `.promptsForUserConfiguration()` if the control is non-functional without configuration — the system will prompt the user to configure it when they add it.

### Refinements
- `controlWidgetActionHint(_:)` — customizes the Action Button pre-action hint (should start with a verb: "Start"/"Stop" yields "Hold to Start"/"Hold to Stop")
- `controlWidgetStatus(_:)` — shows a momentary status in Control Center when an action is performed; use sparingly for pertinent info not already visible
- `.displayName(_:)` / `.description(_:)` — shown in the Controls Gallery; default display name is the app name

## APIs & Frameworks

**WidgetKit**
- `ControlWidget` protocol **[NEW]** — entry point for a control
- `ControlWidgetConfiguration` **[NEW]** — base type for control configurations
- `StaticControlConfiguration` **[NEW]** — non-configurable control; optionally takes a `provider:`
- `AppIntentControlConfiguration` **[NEW]** — user-configurable control driven by an App Intent
- `ControlWidgetToggle` **[NEW]** — toggle control view; takes title, `isOn`, `action`, and label closure
- `ControlWidgetButton` **[NEW]** — button control view for discrete actions
- `ControlValueProvider` protocol **[NEW]**
  - `currentValue() async throws -> Value` **[NEW]**
  - `previewValue: Value` **[NEW]**
- `AppIntentControlValueProvider` protocol **[NEW]** — configurable value provider
  - `currentValue(configuration:) async throws -> Value` **[NEW]**
  - `previewValue(configuration:) -> Value` **[NEW]**
- `.tint(_:)` modifier on control content — custom tint color for on state
- `controlWidgetActionHint(_:)` modifier **[NEW]** — customize Action Button hint text
- `controlWidgetStatus(_:)` modifier **[NEW]** — momentary status text in Control Center
- `.promptsForUserConfiguration()` modifier **[NEW]** — auto-prompt when control is added
- `.displayName(_:)` modifier on configuration **[NEW]**
- `.description(_:)` modifier on configuration **[NEW]**

**App Intents**
- `SetValueIntent` protocol — used by toggle actions that set a boolean value **[NEW usage]**
- `LiveActivityIntent` protocol — used when the action modifies a Live Activity **[NEW usage]**
- `AppIntent.perform()` — called when control is interacted with; system reloads control on completion (existing)

**ControlCenter** (new class)
- `ControlCenter.shared.reloadControls(ofKind:)` **[NEW]** — reload a specific control by kind
- `ControlCenter.shared.reloadAllControls()` **[NEW]**

**Push Notifications** (for remote state updates)
- Push handler API for configuring control reloads from push notifications **[NEW]** — documented separately

## Code Highlights

Complete timer toggle with custom symbols, value text, and tint:
```swift
struct TimerToggle: ControlWidget {
    var body: some ControlWidgetConfiguration {
        StaticControlConfiguration(
            kind: "com.apple.Productivity.TimerToggle"
        ) {
            ControlWidgetToggle(
                "Work Timer",
                isOn: TimerManager.shared.isRunning,
                action: ToggleTimerIntent()
            ) { isOn in
                Label(isOn ? "Running" : "Stopped",
                      systemImage: isOn ? "hourglass" : "hourglass.bottomhalf.filled")
            }
            .tint(.purple)
        }
    }
}
```

App Intent for toggling (SetValueIntent + LiveActivityIntent):
```swift
struct ToggleTimerIntent: SetValueIntent, LiveActivityIntent {
    static let title: LocalizedStringResource = "Productivity Timer"
    @Parameter(title: "Running")
    var value: Bool
    func perform() throws -> some IntentResult {
        TimerManager.shared.setTimerRunning(value)
        return .result()
    }
}
```

Reload the control from within the app:
```swift
ControlCenter.shared.reloadControls(ofKind: "com.apple.Productivity.TimerToggle")
```

Async ValueProvider for server-fetched state:
```swift
struct TimerValueProvider: ControlValueProvider {
    func currentValue() async throws -> Bool {
        try await TimerManager.shared.fetchRunningState()
    }
    let previewValue: Bool = false
}
```

Configurable control with AppIntentControlValueProvider:
```swift
struct ConfigurableTimerValueProvider: AppIntentControlValueProvider {
    func currentValue(configuration: SelectTimerIntent) async throws -> TimerState {
        let timer = configuration.timer
        let isRunning = try await TimerManager.shared.fetchTimerRunning(timer: timer)
        return TimerState(timer: timer, isRunning: isRunning)
    }
    func previewValue(configuration: SelectTimerIntent) -> TimerState {
        return TimerState(timer: configuration.timer, isRunning: false)
    }
}
```

Action hints and metadata:
```swift
Label(isOn ? "Running" : "Stopped", systemImage: isOn ? "hourglass" : "hourglass.bottomhalf.filled")
    .controlWidgetActionHint(isOn ? "Start" : "Stop")
// ...
.displayName("Productivity Timer")
.description("Start and stop a productivity timer.")
```

## Takeaways
- Controls are action-first; keep content to a symbol, title, and optional value text — they are not mini-widgets.
- Use `ControlValueProvider` with `currentValue() async throws` when state must be fetched from a server or database; `previewValue` must be synchronous and correspond to the off/default state.
- Call `ControlCenter.shared.reloadControls(ofKind:)` from your app whenever relevant state changes — don't rely solely on post-action system reloads.
- Add `.promptsForUserConfiguration()` to configurable controls so the system auto-presents configuration when users add the control, preventing a non-functional default state.

---
_Source: WWDC24 Session 10157 page (abstract, chapter summaries, code samples, and resource links)._
