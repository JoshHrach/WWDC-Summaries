# Keep your complications up to date
**WWDC20 · Session 10049** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10049/)

_Platforms:_ watchOS 7

## Overview
Apple Watch complications are always visible, making timely and accurate data critical to a great user experience. This session covers three background mechanisms watchOS provides to keep complications current even when the app is not in the foreground: Background App Refresh (periodic CPU access, up to 4 times/hour), Background URLSession (network data pulls, up to 4 times/hour), and Complication Push Notifications via PushKit (server-driven event updates, up to 50/day per Watch).

The session uses a kite-flying app with three complications as its example: an activity complication using HealthKit data (Background App Refresh), a wind/weather complication (background URLSession), and an encouragement/social complication (complication pushes). All three background paths converge on calling `CLKComplicationServer.sharedInstance().reloadTimeline(for:)` once new data is available.

Complication apps receive special treatment by watchOS: they are kept in memory longer than other apps and restarted if stopped, because they are always visible on the watch face and considered "in use" for privacy purposes.

## Key Topics

### Foreground Updates
- Best opportunity to update complications is during active app use (user input or data received)
- Call `CLKComplicationServer.sharedInstance().reloadTimeline(for:)` for each active complication needing update
- `CLKComplicationServer.activeComplications` returns the array of currently configured complications
- `CLKComplicationDataSource.getCurrentTimelineEntry(for:withHandler:)` is called by the system after reload; provide the current entry using the completion handler

### Background App Refresh
- Schedule via `WKExtension.shared().scheduleBackgroundRefresh(withPreferredDate:userInfo:completionHandler:)`
- System wakes the app after the requested date (typically within a minute or two, depending on conditions)
- Extension delegate receives a `WKApplicationRefreshBackgroundTask` in `handle(_ backgroundTasks:)`
- Limits: up to 4 times per hour; 4 seconds max active CPU time; 15 seconds total task time
- Only one outstanding request at a time — schedule the next refresh before marking the current task complete
- URLSession networking is NOT allowed in Background App Refresh (use background URLSession instead)
- Mark complete with `task.setTaskCompletedWithSnapshot(false)` — complication update already triggers a snapshot request

### Background URLSession
- Create a `URLSessionConfiguration.background(withIdentifier:)` with `isDiscretionary = false` and `sessionSendsLaunchEvents = true`
- Create a `URLSession` with the background configuration, set `delegateQueue` to `nil` for background serial queue delivery
- Create a `URLSessionDownloadTask`, set `earliestBeginDate`, `countOfBytesClientExpectsToSend`, `countOfBytesClientExpectsToReceive`; call `.resume()`
- App receives a `WKURLSessionRefreshBackgroundTask` in the extension delegate when the download completes
- Must reattach to the `URLSession` whenever the app is activated (before delegate methods arrive)
- Limits: up to 4 requests per hour; same 15-second task window; no expensive processing
- Intermediate delegate calls: `willBeginDelayedRequest` (can update/cancel URL), `didReceiveChallenge` (authentication)
- Final delegates: `urlSession(_:downloadTask:didFinishDownloadingTo:)` and `urlSession(_:task:didCompleteWithError:)`
- Do NOT schedule the next task during `sessionDidFinishEvents`; instead mark the task complete and schedule inside the WKURLSession background task handler

### Complication Push Notifications (PushKit)
- Register using `PKPushRegistry(queue:)`, set `desiredPushTypes = [.complication]`
- Requires an app identifier ending in `.watchkitapp.complication` and a corresponding APNs SSL certificate
- WatchKit Extension must have Remote Notification Background Mode and Push Notifications capabilities
- Upload credentials from `pushRegistry(_:didUpdate:for:)` to your server
- Server sends background pushes (with `content-available: 1`) to Apple's push servers using these credentials
- Watch receives up to 50 complication pushes per day (51st and beyond are ignored — apply server-side throttling)
- `pushRegistry(_:didReceiveIncomingPushWith:for:completion:)` — process payload and call `updateActiveComplications()`; call completion handler when done
- Pushes don't need to be evenly spaced; suitable for bursty, event-driven data

## APIs & Frameworks

- **ClockKit**
  - `CLKComplicationServer.sharedInstance()` — singleton that manages all complications
  - `CLKComplicationServer.activeComplications` — array of currently active complications
  - `CLKComplicationServer.reloadTimeline(for:)` — triggers a timeline refresh for a specific complication
  - `CLKComplicationDataSource` — protocol for providing complication data
  - `CLKComplicationDataSource.getCurrentTimelineEntry(for:withHandler:)` — provide current entry on request
  - `CLKComplicationTimelineEntry` — a dated complication template entry
  - `CLKComplicationTemplate` — base class for complication templates (e.g., `CLKComplicationTemplateModularLargeTallBody`)
  - `CLKTextProvider` — base class for text content providers (e.g., header, body)
- **WatchKit**
  - `WKExtension.shared().scheduleBackgroundRefresh(withPreferredDate:userInfo:completionHandler:)` — schedules Background App Refresh
  - `WKExtensionDelegate.handle(_:)` — handles background tasks
  - `WKApplicationRefreshBackgroundTask` — background app refresh task
  - `WKURLSessionRefreshBackgroundTask` — background URL session task
  - `WKRefreshBackgroundTask.setTaskCompletedWithSnapshot(_:)` — marks task done; pass `false` when a complication update was performed
- **Foundation / URLSession**
  - `URLSessionConfiguration.background(withIdentifier:)` — creates a background session configuration
  - `URLSessionConfiguration.isDiscretionary` — set to `false` for timely updates
  - `URLSessionConfiguration.sessionSendsLaunchEvents` — set to `true` to wake the app on completion
  - `URLSession(configuration:delegate:delegateQueue:)` — create session with `delegateQueue: nil` for background queue
  - `URLSessionDownloadTask` — download task for background sessions
  - `URLSessionDownloadTask.earliestBeginDate` — minimum date before the task runs
  - `URLSessionDownloadTask.countOfBytesClientExpectsToSend` / `.countOfBytesClientExpectsToReceive` — hints for system scheduling
  - `URLSessionDownloadDelegate.urlSession(_:downloadTask:didFinishDownloadingTo:)` — download complete
  - `URLSessionTaskDelegate.urlSession(_:task:didCompleteWithError:)` — task complete
  - `URLSessionTaskDelegate.urlSession(_:dataTask:willBeginDelayedRequest:completionHandler:)` — can update/cancel URL
  - `URLSessionTaskDelegate.urlSession(_:task:didReceive:completionHandler:)` — authentication challenges
- **PushKit**
  - `PKPushRegistry` — registers for push types
  - `PKPushRegistry.desiredPushTypes` — set to `[.complication]`
  - `PKPushRegistryDelegate` — protocol for push events
  - `PKPushRegistryDelegate.pushRegistry(_:didUpdate:for:)` — credentials available; upload to server
  - `PKPushRegistryDelegate.pushRegistry(_:didReceiveIncomingPushWith:for:completion:)` — push received
  - `PKPushCredentials` — device push credentials
  - `PKPushPayload` — received push payload
- **HealthKit** — used for activity data via async queries (no specific new API, used within Background App Refresh)
- **Core Location** — used for caching location for weather URL construction

## Code Highlights

Reload all active complications:
```swift
func updateActiveComplications() {
    let server = CLKComplicationServer.sharedInstance()
    server.activeComplications?.forEach { server.reloadTimeline(for: $0) }
}
```

Schedule Background App Refresh:
```swift
WKExtension.shared().scheduleBackgroundRefresh(
    withPreferredDate: Date().addingTimeInterval(15 * 60),
    userInfo: ["submissionDate": Date()]
) { error in
    if let error { print("BAR scheduling failed: \(error)") }
}
```

Background URLSession setup:
```swift
let config = URLSessionConfiguration.background(withIdentifier: "BackgroundWeather")
config.isDiscretionary = false
config.sessionSendsLaunchEvents = true
let session = URLSession(configuration: config, delegate: self, delegateQueue: nil)
let task = session.downloadTask(with: url)
task.earliestBeginDate = Date().addingTimeInterval(15 * 60)
task.countOfBytesClientExpectsToSend = 200
task.countOfBytesClientExpectsToReceive = 1024
task.resume()
```

Complication push registration:
```swift
let registry = PKPushRegistry(queue: .main)
registry.delegate = self
registry.desiredPushTypes = [.complication]
```

## Takeaways
- Use all three background mechanisms together for maximum update freshness: Background App Refresh for local/HealthKit data, background URLSession for server data, PushKit for event-driven server pushes.
- Always schedule the _next_ background refresh before marking the current task complete; networking is forbidden in Background App Refresh tasks — use background URLSession for that.
- Complication pushes (up to 50/day) are ideal for bursty or event-driven data; apply server-side throttling to avoid exceeding the cap.
- Mark every background task complete within 15 seconds; set `setTaskCompletedWithSnapshot(false)` when a complication reload was issued (the reload itself triggers a snapshot).

---
_Source: WWDC20 Session 10049 page (abstract, transcript, code samples, and resource links)._
