# Finish tasks in the background

**Session ID:** 227  
**WWDC Year:** 2025  
**Folder:** `13-system-frameworks-and-design`  
**URL:** https://developer.apple.com/videos/play/wwdc2025/227/

---

## Overview

This session introduces `BGContinuedProcessingTask`, the most significant new addition to BackgroundTasks in iOS/iPadOS 26. Unlike the existing `BGProcessingTask` (which gives apps a burst of CPU time when the device is idle and charging), `BGContinuedProcessingTask` is explicitly triggered by user intent when they leave the app, and provides an extended runtime window — up to several minutes — with progress reporting to the system so the task survives device lock and low-power conditions. The session also recaps the full BackgroundTasks taxonomy and gives guidance on choosing the right task type for common use cases such as ML inference, media export, and sync.

---

## Key Topics

- Full taxonomy of iOS background task types and when to use each
- New `BGContinuedProcessingTask`: user-intent-driven long-running tasks
- How `BGContinuedProcessingTask` differs from `BGProcessingTask` and background URL sessions
- Reporting progress so the system allows continued execution
- System UI shown to users while a continued processing task runs
- Scheduling and cancellation lifecycle
- Energy and performance budgeting for background tasks

---

## APIs & Frameworks

- **BackgroundTasks** framework (`import BackgroundTasks`) – Framework for scheduling deferred and background work on iOS, iPadOS, and macOS.
- **`BGContinuedProcessingTask`** – **[NEW]** (iOS 26, iPadOS 26) Background task subclass for work initiated by explicit user action (e.g., "Export video" or "Train model") that should continue after the user leaves the app. Provides extended runtime proportional to reported progress.
- **`BGContinuedProcessingTaskRequest`** – **[NEW]** Request object for scheduling a `BGContinuedProcessingTask`; properties: `identifier: String`, `userInfo: [String: Any]?`.
- **`BGContinuedProcessingTask.progress`** – **[NEW]** `Progress` object; must be updated regularly. The system uses reported `fractionCompleted` to determine how much additional runtime to grant. Failing to advance progress causes termination.
- **`BGContinuedProcessingTask.expirationHandler`** – Closure called when the system needs to terminate the task; use to checkpoint state so work can resume on next launch.
- **`BGTaskScheduler.shared.submit(_:)`** – Existing API for submitting any `BGTaskRequest`; now accepts `BGContinuedProcessingTaskRequest`.
- **`BGTaskScheduler.shared.register(forTaskWithIdentifier:using:launchHandler:)`** – Register the task handler at app launch; required for all BG task types.
- **`BGProcessingTask`** – Existing API for opportunistic background processing (idle device, charging); still appropriate for non-user-initiated batch work.
- **Background URL Session** (`URLSession` with `.background` configuration) – Still the preferred API for large uploads/downloads; not superseded by `BGContinuedProcessingTask`.
- **`BGAppRefreshTask`** – Existing short-duration refresh; unchanged.
- **Info.plist key `BGTaskSchedulerPermittedIdentifiers`** – Required array listing all registered background task identifiers, including `BGContinuedProcessingTask` identifiers.

---

## Code Highlights

Registering a continued processing task at app launch:
```swift
import BackgroundTasks

BGTaskScheduler.shared.register(
    forTaskWithIdentifier: "com.example.app.exportVideo",
    using: nil
) { task in
    guard let continuedTask = task as? BGContinuedProcessingTask else { return }
    handleVideoExport(task: continuedTask)
}
```

Scheduling the task when the user taps "Export and close":
```swift
let request = BGContinuedProcessingTaskRequest(
    identifier: "com.example.app.exportVideo"
)
request.userInfo = ["exportID": exportSession.id]
try BGTaskScheduler.shared.submit(request)
```

Running the task with progress reporting:
```swift
func handleVideoExport(task: BGContinuedProcessingTask) {
    task.expirationHandler = {
        exportSession.cancel()
        checkpointExportState()
    }

    Task {
        for await progressUpdate in exportSession.progressUpdates {
            task.progress.fractionCompleted = progressUpdate.fraction
            if progressUpdate.isComplete { break }
        }
        task.setTaskCompleted(success: true)
    }
}
```

---

## Takeaways

- `BGContinuedProcessingTask` fills a critical gap: work the user explicitly starts (media export, model training, large file sync) that must survive the user backgrounding the app.
- Progress reporting via `task.progress.fractionCompleted` is mandatory — the system uses it as a signal of liveness; stalled progress leads to termination.
- Unlike `BGProcessingTask`, the system shows UI indicating a background task is running (similar to background app refresh indicators), so users are aware.
- The task is tied to a specific user action: schedule it when the user commits the action, not speculatively.
- For downloads and uploads, continue using background `URLSession`; `BGContinuedProcessingTask` is for CPU-bound or local I/O work.
- Always implement `expirationHandler` to checkpoint state so partial work can be resumed if the system terminates the task early.
