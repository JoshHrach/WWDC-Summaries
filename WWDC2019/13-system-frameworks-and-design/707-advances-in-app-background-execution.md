# Advances in App Background Execution
**WWDC19 · Session 707** · [Watch](https://developer.apple.com/videos/play/wwdc2019/707/)

_Platforms:_ iOS 13, iPadOS 13, tvOS 13, macOS Catalina 10.15 (iPad apps on Mac)

## Overview
This session introduces the new BackgroundTasks framework — the most significant change to iOS background execution since background fetch — and provides updated best practices for existing background APIs. The framework introduces two new background modes: `BGAppRefreshTask` for keeping app content current throughout the day, and `BGProcessingTask` for deferrable maintenance work (database cleanup, Core ML training, backups) that runs at system-friendly times when the device is idle and charging.

The session also reviews the appropriate API for four core background use cases in a messaging app: background task completion for completing user-initiated work, VoIP pushes (with a new mandatory CallKit requirement) for incoming calls, background pushes for muted-thread content delivery, and discretionary URLSessions for deferred downloads. Each mode carries different power, performance, and privacy tradeoffs.

A live Xcode debugging demo shows how to simulate both task launch and expiration from the debugger console without waiting for real system scheduling.

## Key Topics

### Background Execution Principles
- Three design factors: **Power** (minimize battery drain), **Performance** (protect foreground apps' CPU/memory), **Privacy** (each mode has a scoped data access model).
- Apps enter background via: self-requested work (downloads, foreground task completion) or event triggers (region enter, HealthKit update, push).
- Always call completion handlers promptly to let the system suspend the app.

### Existing Modes Best Practices

**Background Task Completion (`UIApplication.beginBackgroundTask`)**
- Use for protecting completion of user-initiated work (sending a message, saving a file).
- Start the background task before the user leaves the app — not after.
- Implement an expiration handler to clean up and notify the user if time runs out.
- Pair with `UIApplication.endBackgroundTask` (or `ProcessInfo.performExpiringActivity` in extensions).

**VoIP Push Notifications (PushKit)**
- **[NEW requirement iOS 13]**: Must report every incoming push via CallKit's `CXProvider.reportNewIncomingCall` inside `PKPushRegistryDelegate.pushRegistry(_:didReceiveIncomingPushWith:for:completion:)` synchronously, before the method returns.
- Failure to report results in app termination; repeated failure results in the system stopping VoIP push delivery.
- Set `apns-expiration` to `0` or a few seconds to avoid stale incoming call UI.
- Include caller info directly in the push payload to enable immediate, rich CallKit UI.

**Background Pushes (Silent Push)**
- Use `content-available: 1` with no alert/sound/badge for best-effort content delivery.
- **[NEW requirement]**: Must set `apns-priority: 5` (system decides when to launch — not immediate).
- **[NEW requirement]**: Set `apns-push-type: background` (required on watchOS, strongly recommended everywhere).
- System launches the app at a power/performance-opportune time — treat as best effort.

**Discretionary URLSession**
- Set `URLSessionConfiguration.isDiscretionary = true` for deferrable uploads/downloads.
- Optional: `timeoutIntervalForResource`, `earliestBeginDate`, `countOfBytesClientExpectsToSend/Receive`.

### New: BackgroundTasks Framework **[NEW]**

**BGAppRefreshTask**
- Replaces the deprecated `UIApplication.setMinimumBackgroundFetchInterval` API.
- Delivers ~30 seconds of runtime; system schedules based on learned user usage patterns.
- Deprecated: `UIApplication.backgroundRefreshStatus`, `UIApplicationDelegate.application(_:performFetchWithCompletionHandler:)`.

**BGProcessingTask**
- New background mode: apps receive several minutes of runtime at system-friendly times.
- Ideal for: database cleanup, server sync, Core ML on-device training/inference, backups.
- `BGProcessingTaskRequest.requiresNetworkConnectivity: Bool` (default `false`).
- `BGProcessingTaskRequest.requiresExternalPower: Bool` — set `true` for CPU-intensive work; also disables CPU Monitor throttling.
- System guarantees the device is unlocked at least once before launching the task.
- Files must be at most `FileProtectionType.completeUntilFirstUserAuthentication` to be accessible.

**BGTaskScheduler (shared)**
- Register launch handlers in `application(_:didFinishLaunchingWithOptions:)` using `BGTaskScheduler.shared.register(forTaskWithIdentifier:using:launchHandler:)`.
- Submit task requests via `BGTaskScheduler.shared.submit(_:)` — blocking/synchronous; call on a background queue during launch to avoid blocking the main thread.
- Each `BGTaskRequest` corresponds to exactly one launch — re-submit in the launch handler for recurring behavior.
- `earliestBeginDate`: do not set more than one week in the future (privacy expectation — unused apps should not run background tasks months later).

**Info.plist requirement**: add `BGTaskSchedulerPermittedIdentifiers` (array of strings, reverse-DNS-style IDs) and enable the appropriate Background Modes capability.

**Debugging**:
- Pause app in debugger, then call `e -l objc -- (void)[[BGTaskScheduler sharedScheduler] _simulateLaunchForTaskWithIdentifier:@"com.example.refresh"]` to simulate launch.
- Replace `_simulateLaunch` with `_simulateExpirationForTaskWithIdentifier` to test expiration handling.

### UIScene / Multi-Window
- For multi-window apps adopting `UIScene`: call `UIApplication.shared.requestSceneSessionRefresh(_:)` after a background refresh to update the app switcher snapshot.

## APIs & Frameworks

**BackgroundTasks** framework **[NEW]** (iOS 13+, iPadOS 13+, tvOS 13+, macOS 10.15+)
- `BGTaskScheduler` **[NEW]** — `.shared`, `register(forTaskWithIdentifier:using:launchHandler:)`, `submit(_:)`, `cancel(taskRequestWithIdentifier:)`, `cancelAllTaskRequests()`
- `BGTask` **[NEW]** — `identifier: String`, `expirationHandler: (() -> Void)?`, `setTaskCompleted(success:)`
- `BGAppRefreshTask: BGTask` **[NEW]**
- `BGProcessingTask: BGTask` **[NEW]**
- `BGTaskRequest` **[NEW]** — `identifier: String`, `earliestBeginDate: Date?`
- `BGAppRefreshTaskRequest: BGTaskRequest` **[NEW]**
- `BGProcessingTaskRequest: BGTaskRequest` **[NEW]** — `requiresNetworkConnectivity: Bool`, `requiresExternalPower: Bool`
- `BGTaskSchedulerPermittedIdentifiers` — Info.plist key **[NEW]**

**UIKit (existing, changes in iOS 13)**
- `UIApplication.beginBackgroundTask(expirationHandler:) -> UIBackgroundTaskIdentifier`
- `UIApplication.endBackgroundTask(_:)`
- `ProcessInfo.performExpiringActivity(withReason:using:)` — extension equivalent
- `UIApplication.setMinimumBackgroundFetchInterval(_:)` **[DEPRECATED iOS 13]**
- `UIApplicationDelegate.application(_:performFetchWithCompletionHandler:)` **[DEPRECATED iOS 13]**
- `UIApplication.shared.requestSceneSessionRefresh(_:)` **[NEW]** — multi-window snapshot update

**PushKit**
- `PKPushRegistry` — `desiredPushTypes: Set<PKPushType>` including `.voIP`
- `PKPushRegistryDelegate.pushRegistry(_:didReceiveIncomingPushWith:for:completion:)` — **[CHANGED]** must call CallKit synchronously
- `apns-push-type: background` push header **[NEW requirement]**
- `apns-priority: 5` push header **[NEW requirement for background pushes]**

**CallKit**
- `CXProvider.reportNewIncomingCall(with:update:completion:)` — must call inside VoIP push handler

**Foundation**
- `URLSessionConfiguration.isDiscretionary: Bool`
- `URLSessionConfiguration.timeoutIntervalForResource: TimeInterval`
- `URLSessionConfiguration.earliestBeginDate: Date?`
- `URLSessionConfiguration.countOfBytesClientExpectsToReceive: Int64`

## Code Highlights

```swift
// 1. Register task handlers at launch (AppDelegate)
import BackgroundTasks

func application(_ application: UIApplication, didFinishLaunchingWithOptions ...) -> Bool {
    BGTaskScheduler.shared.register(forTaskWithIdentifier: "com.colorfeed.refresh", using: nil) { task in
        self.handleAppRefresh(task: task as! BGAppRefreshTask)
    }
    BGTaskScheduler.shared.register(forTaskWithIdentifier: "com.colorfeed.dbcleaning", using: nil) { task in
        self.handleDatabaseCleaning(task: task as! BGProcessingTask)
    }
    return true
}

// 2. Handle the refresh task
func handleAppRefresh(task: BGAppRefreshTask) {
    scheduleAppRefresh()   // re-schedule for next time
    let queue = OperationQueue()
    task.expirationHandler = { queue.cancelAllOperations() }
    let fetchOp = FetchLatestFeedOperation()
    fetchOp.completionBlock = {
        task.setTaskCompleted(success: !fetchOp.isCancelled)
    }
    queue.addOperation(fetchOp)
}

// 3. Schedule the request
func scheduleAppRefresh() {
    let request = BGAppRefreshTaskRequest(identifier: "com.colorfeed.refresh")
    request.earliestBeginDate = Date(timeIntervalSinceNow: 15 * 60)  // 15 min delay
    try? BGTaskScheduler.shared.submit(request)
}

// 4. Schedule a processing task only when needed
func scheduleDatabaseCleaningIfNeeded() {
    guard daysSinceLastCleaning >= 7 else { return }
    let request = BGProcessingTaskRequest(identifier: "com.colorfeed.dbcleaning")
    request.requiresNetworkConnectivity = false
    request.requiresExternalPower = true
    try? BGTaskScheduler.shared.submit(request)
}
```

```swift
// VoIP push: mandatory CallKit reporting in iOS 13
func pushRegistry(_ registry: PKPushRegistry, didReceiveIncomingPushWith payload: PKPushPayload,
                  for type: PKPushType, completion: @escaping () -> Void) {
    if type == .voIP {
        let update = CXCallUpdate()
        update.remoteHandle = CXHandle(type: .generic, value: payload.dictionaryPayload["caller"] as! String)
        provider.reportNewIncomingCall(with: UUID(), update: update) { _ in completion() }
    }
}
```

## Takeaways
- The new BackgroundTasks framework is the right way to schedule all deferrable background work on iOS 13+; it replaces deprecated background fetch APIs and adds the new `BGProcessingTask` mode.
- VoIP apps **must** call CallKit synchronously inside every `didReceiveIncomingPush` callback in iOS 13 — failure results in app termination and eventual push blacklisting.
- Background pushes require both `apns-priority: 5` and `apns-push-type: background` headers on iOS 13.
- `BGProcessingTask` with `requiresExternalPower: true` is the correct home for CPU-intensive background work (Core ML training, large syncs) — it disables the CPU Monitor throttle while the device is charging.

---
_Source: WWDC19 Session 707 page (abstract, chapter summaries, code samples, and resource links)._
