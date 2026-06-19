# What's New in Background Assets
**WWDC23 · Session 10108** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10108/)

_Platforms:_ iOS 16.4+, iPadOS 16.4+, macOS Ventura 13.3+

## Overview
Background Assets (introduced in iOS 16.1 / macOS Ventura) receives two major additions: **essential downloads** — content that downloads as part of the app installation flow and blocks app launch until complete — and `backgroundassets-debug`, a command-line tool for simulating extension entry points during development.

The session walks through migrating a URLSession-based download app to Background Assets, implementing both the app-side `BADownloadManager` integration and the app extension `BADownloaderExtension` protocol. Essential assets integrate directly with the iOS Home Screen, macOS Launchpad, and the App Store installation progress indicator, giving users a single unified install experience that includes required game/app assets.

The session also covers the lifecycle and memory constraints of the Background Assets extension, best practices for file management (caches directory, purgeability), and using `withExclusiveControl` to coordinate between the app and its extension.

## Key Topics

### Essential Downloads (iOS 16.4 / macOS 13.3)
- **Essential assets** contribute to the app's installation progress on the iOS Home Screen, macOS Launchpad, and the App Store.
- The app is blocked from launching until all essential downloads complete.
- Essential downloads can only be enqueued during a `BAContentRequest` with type `.install` or `.update` (not `.periodic`).
- HTTP range requests must be supported on the server (to support pause/resume).
- Accurate `fileSize` is mandatory for essential downloads; a mismatch causes the download to fail.
- If essential downloads fail, the app eventually becomes launchable; re-enqueue as non-essential using `removingEssential()`.
- Users can disable essential assets in App Store Settings ("in-app content" toggle).

### New Info.plist Keys
- `BAEssentialDownloadAllowance` **[NEW]** — upper bound in bytes for all essential downloads combined (used to set up the installation progress indicator); should be as close to actual download size as possible.
- `BAEssentialMaxInstallSize` **[NEW]** — maximum extracted size of essential assets on disk; appears in the App Store to inform users of storage requirements.
- Existing keys: `BAInitialDownloadRestrictions` (with `BADownloadAllowance`, `BADownloadDomainAllowList`), `BAMaxInstallSize`, `BAManifestURL`.

### New BAURLDownload API
- `BAURLDownload(identifier:request:essential:fileSize:applicationGroupIdentifier:priority:)` **[NEW]** — new initializer supporting essential downloads; `essential: true` marks the download as contributing to install progress.
- `BAURLDownload.removingEssential()` **[NEW]** — returns a non-essential copy of an essential download; use in `backgroundDownloadDidFail` to re-enqueue failed essential downloads as background downloads.

### Extension Lifecycle & Constraints
- Extension invoked during: app install, app update, periodic background fetch.
- Runtime is measured from function entry to function exit; async `withExclusiveControl` extends runtime until completion handler returns.
- Memory limit: a few megabytes; use memory-mapped files for large data; exceeding the limit causes termination.
- Default runtime allotment: a few minutes per day; throttled for rarely used apps, potentially increased for frequently used apps.
- Low Power Mode or disabled Background App Refresh prevents extension from running.
- Extension's in-memory state is not persisted across invocations — serialize state to disk.

### BADownloadManager (App Side)
- `BADownloadManager.shared` — singleton; used in both app and extension.
- `fetchCurrentDownloads(completionHandler:)` — enumerate in-flight downloads to check if extension already scheduled them.
- `promoteScheduledDownload(_:)` — promotes a background download to foreground priority; does not restart the download.
- `withExclusiveControl(completionHandler:)` **[NEW in usage]** — async mutual exclusion between app and extension; use when both may access shared resources simultaneously.
- `delegate` — assign from the app to receive download callbacks instead of the extension (when app is running).

### Debugging: backgroundassets-debug
- New Xcode CLI tool: `xcrun backgroundassets-debug`
- Simulates extension entry points: `--simulate app-install`, `--simulate app-update`, `--simulate periodic`
- Works over Bluetooth or Wi-Fi (no USB required); supports multiple connected devices by UUID
- Developer Mode must be enabled on the device
- Resetting the extension's runtime occurs automatically when simulating events

### File Management Best Practices
- Downloaded assets are marked **purgeable** by the system; store in the app's **caches directory**.
- Moving files (not copying) preserves purgeable tracking; use `FileManager.replaceItemAt` or `moveItem`.
- Modified or expanded files are not tracked as purgeable by the system.

## APIs & Frameworks

- `BackgroundAssets` framework — background download management
- `BADownloaderExtension` protocol — extension entry points: `backgroundDownload(contentForRequest:manifest:)`, `backgroundDownloadDidFinish(_:fileURL:)`, `backgroundDownloadDidFail(_:)`
- `BAContentRequest` — enum: `.install`, `.update`, `.periodic`
- `BAURLDownload` — download descriptor; conforms to `BADownload`
- `BAURLDownload(identifier:request:essential:fileSize:applicationGroupIdentifier:priority:)` **[NEW]** — essential download initializer
- `BAURLDownload.essential` **[NEW]** — Bool property indicating essential status
- `BAURLDownload.removingEssential()` **[NEW]** — returns non-essential copy
- `BADownloadManager` — singleton managing downloads
- `BADownloadManager.shared`
- `BADownloadManager.fetchCurrentDownloads(completionHandler:)`
- `BADownloadManager.promoteScheduledDownload(_:)`
- `BADownloadManager.withExclusiveControl(completionHandler:)` — async mutual exclusion
- `BADownloadManager.delegate` — `BADownloadManagerDelegate` for app-side callbacks
- `BADownloadManagerDelegate` protocol — `didReceiveChallenge`, `didFinishDownload`, `didFailDownload`, `downloadDidBegin`, `downloadDidProgress`
- `BAManifestURL` (Info.plist key) — URL for app manifest file pre-downloaded before extension launch
- `BAEssentialDownloadAllowance` (Info.plist key) **[NEW]** — byte budget for essential downloads
- `BAEssentialMaxInstallSize` (Info.plist key) **[NEW]** — max extracted size of essential assets
- `BADownloadAllowance` (Info.plist key) — byte budget for non-essential initial downloads
- `BAMaxInstallSize` (Info.plist key) — max extracted size of non-essential initial downloads
- `BADownloadDomainAllowList` (Info.plist key) — allowed CDN domains
- `BAInitialDownloadRestrictions` (Info.plist key) — dictionary containing download restrictions
- `xcrun backgroundassets-debug` **[NEW]** — CLI tool for simulating extension events

## Code Highlights

```swift
// Essential download (extension)
let download = BAURLDownload(
    identifier: session.identifier,
    request: URLRequest(url: session.url),
    essential: contentRequest.reason == .install,
    fileSize: session.fileSize,
    applicationGroupIdentifier: "group.com.example.app",
    priority: .default
)

// Re-enqueue failed essential as non-essential (extension)
func backgroundDownloadDidFail(_ failedDownload: BADownload) {
    guard let download = failedDownload as? BAURLDownload else { return }
    let nonEssential = download.removingEssential()
    try? BADownloadManager.shared.scheduleDownload(nonEssential)
}

// Promote background download to foreground (app)
BADownloadManager.shared.withExclusiveControl { granted, error in
    guard granted else { return }
    BADownloadManager.shared.fetchCurrentDownloads { downloads, error in
        if let existing = downloads.first(where: { $0.identifier == id }) {
            BADownloadManager.shared.promoteScheduledDownload(existing)
        } else {
            // Schedule new download
        }
    }
}
```

```bash
# Simulate an app-install event on a connected device
xcrun backgroundassets-debug --simulate app-install --device <UUID> com.example.app
```

## Takeaways
- Essential downloads (iOS 16.4+) integrate app asset downloads directly into the App Store/TestFlight installation progress, eliminating the "wait after first launch" problem for content-heavy apps.
- `BAURLDownload.removingEssential()` provides a clean path to recover from failed essential downloads by re-enqueueing as background non-essential.
- Use `BADownloadManager.withExclusiveControl` whenever both the app and extension may access shared resources (manifest, download state) simultaneously.
- The `backgroundassets-debug` CLI tool makes extension debugging practical — no App Store submission required to test installation-triggered downloads.

---
_Source: WWDC23 Session 10108 page (abstract, chapter summaries, code samples, transcript, and resource links)._
