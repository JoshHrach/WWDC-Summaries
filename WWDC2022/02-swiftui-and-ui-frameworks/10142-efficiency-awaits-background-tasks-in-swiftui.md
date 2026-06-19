# Efficiency Awaits: Background Tasks in SwiftUI
**WWDC22 · Session 10142** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10142/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, tvOS 16, watchOS 9

## Overview
This session introduces a new unified SwiftUI API for handling background tasks using Swift Concurrency. Presented by John Gallagher from the watchOS Frameworks team, the talk walks through building "Stormy" — a sky-photo app that uses background app refresh and background URLSession tasks to check weather conditions and send conditional notifications without requiring the app to be in the foreground.

The new `.backgroundTask(_:action:)` scene modifier replaces platform-specific background task registration boilerplate with a single, Swift Concurrency-compatible API that works identically across watchOS, iOS, tvOS, Mac Catalyst, and Widgets. Swift's native task cancellation mechanism handles the "runtime expiring" scenario automatically, enabling clean promotion of foreground network requests to background URLSession downloads.

## Key Topics

### Background Task Lifecycle
- The system wakes a suspended app at the scheduled time and delivers a background task.
- Apps have limited background runtime per task type; if work is not completed, the system may quit and throttle the app.
- When runtime is expiring the system cancels the Swift `Task` running the background task closure — apps should use `withTaskCancellationHandler` to respond.
- Background app refresh: scheduled in advance, provides brief runtime at a target date.
- URLSession background task: wakes the app when a background download or upload completes.

### `backgroundTask(_:action:)` Scene Modifier **[NEW]**
- Attach to any `Scene` (typically `WindowGroup` or `WKNotificationScene`).
- Takes a `BackgroundTask` value describing which system task to handle and an `async` closure.
- Closure returns when all work is complete; task is implicitly marked finished when the closure returns.
- Multiple modifiers can be chained for different task types.
- Consistent API across all Apple platforms — same code structure on watchOS and iOS.

### Background App Refresh
- Schedule with `BGAppRefreshTaskRequest(identifier:)` and `BGTaskScheduler.shared.submit(_:)`.
- Set `earliestBeginDate` to control when the system should wake the app.
- Use `BackgroundTask.appRefresh("identifier")` in the modifier to match a scheduled request.
- Best practice: reschedule the next refresh at the start of the background task closure to ensure continuity.

### Background URLSession
- For network requests that may outlive background runtime: create `URLSessionConfiguration.background(withIdentifier:)` with `sessionSendsLaunchEvents = true`.
- On watchOS: ALL network requests from background tasks must use background URLSessions.
- Register a handler with `BackgroundTask.urlSession("identifier")` using the same identifier as the session configuration.
- URLSession deduplicates in-flight requests — promoting a foreground request to background does not duplicate the network call.

### Swift Concurrency Integration
- `await` replaces nested completion handlers throughout background task closures.
- `withTaskCancellationHandler(operation:onCancel:)` — run async work with a cancellation handler; the `onCancel` block fires synchronously when the parent task is cancelled.
- Use the cancellation handler to promote an in-flight `URLSession.data(for:)` request to a `URLSession.downloadTask(with:)` on the background session, ensuring the download survives suspension.
- `UNUserNotificationCenter.add(_:)` has an async overload — fully awaitable in background task closures.

## APIs & Frameworks

### SwiftUI **[NEW]**
- `Scene.backgroundTask(_:action:)` — register background task handler **[NEW]**
- `BackgroundTask` — namespace for task type values **[NEW]**
- `BackgroundTask.appRefresh(_ identifier: String)` — app refresh task type **[NEW]**
- `BackgroundTask.urlSession(_ identifier: String)` — URLSession background task type **[NEW]**

### BackgroundTasks Framework (existing, used internally)
- `BGAppRefreshTaskRequest(identifier:)` — create scheduled app refresh request
- `BGTaskScheduler.shared.submit(_:)` — submit background task request
- `BGAppRefreshTaskRequest.earliestBeginDate` — target wakeup time

### Foundation / URLSession
- `URLSessionConfiguration.background(withIdentifier:)` — background session config
- `URLSessionConfiguration.sessionSendsLaunchEvents` — triggers app launch events on task completion
- `URLSession(configuration:)` — create session from configuration
- `URLSession.data(for:)` — async data download (cancellable)
- `URLSession.downloadTask(with:)` — background download (survives suspension)

### Swift Concurrency
- `withTaskCancellationHandler(operation:onCancel:)` — handle task cancellation
- `Task.isCancelled` — check cancellation state
- `async`/`await` — core concurrency primitives throughout

### UserNotifications
- `UNUserNotificationCenter.add(_:)` — async overload for scheduling notifications
- `UNMutableNotificationContent` — notification content
- `UNTimeIntervalNotificationTrigger` — time-based trigger

## Code Highlights

```swift
// Schedule background app refresh for noon tomorrow
func scheduleAppRefresh() {
    let today = Calendar.current.startOfDay(for: .now)
    let tomorrow = Calendar.current.date(byAdding: .day, value: 1, to: today)!
    let noonTomorrow = Calendar.current.date(bySettingHour: 12, minute: 0, second: 0, of: tomorrow)!

    let request = BGAppRefreshTaskRequest(identifier: "com.example.stormy.refresh")
    request.earliestBeginDate = noonTomorrow
    try? BGTaskScheduler.shared.submit(request)
}

// Register handler using SwiftUI backgroundTask modifier
@main
struct StormyApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .backgroundTask(.appRefresh("com.example.stormy.refresh")) {
            // Reschedule immediately to ensure continuity
            scheduleAppRefresh()
            // Check weather and notify if stormy
            if await isStormy() {
                await notifyForPhoto()
            }
        }
        .backgroundTask(.urlSession("com.example.stormy.session")) {
            // Handle completed background URLSession task
        }
    }
}

// Async weather check with cancellation handler for runtime expiry
func isStormy() async -> Bool {
    let session = URLSession(configuration: {
        let config = URLSessionConfiguration.background(withIdentifier: "com.example.stormy.session")
        config.sessionSendsLaunchEvents = true
        return config
    }())
    let request = URLRequest(url: weatherServiceURL)

    let (data, _) = try await withTaskCancellationHandler {
        // Normal async download
        try await session.data(for: request)
    } onCancel: {
        // Runtime expiring — promote to background download that survives suspension
        session.downloadTask(with: request).resume()
    }
    return parseStormy(from: data)
}
```

## Takeaways
- The new `.backgroundTask(_:action:)` SwiftUI modifier provides a single, cross-platform API for registering background task handlers — the same code works on watchOS, iOS, tvOS, and Mac Catalyst.
- Swift Concurrency's task cancellation is the mechanism for handling "runtime expiring" events; use `withTaskCancellationHandler` to promote foreground requests to background URLSession downloads when time runs short.
- On watchOS, all network requests from background tasks must use background URLSessions — this is enforced by the system.
- Always reschedule the next background app refresh at the beginning of the task handler to ensure future wakeups are registered even if the task is terminated early.

---
_Source: WWDC22 Session 10142 page (transcript, resource links)._
