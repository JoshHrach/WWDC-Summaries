# Meet Background Assets
**WWDC22 · Session 110403** · [Watch](https://developer.apple.com/videos/play/wwdc2022/110403/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13

## Overview
Background Assets is a new framework introduced in WWDC22 that enables apps to schedule and download large asset files from a developer-controlled CDN before and between app launches. The framework introduces a dedicated app extension — the Background Downloader extension — that wakes automatically when the app is first installed, whenever the app is updated in the background, and periodically based on app usage frequency. This allows games and content-heavy apps to have their required assets present and ready the first time a user opens the app, eliminating the frustrating "downloading after launch" experience.

The framework is designed to complement existing asset management systems and does not require content to be hosted by Apple. Developers retain full control over CDN choice, compression, and delivery, while the OS manages scheduling, prioritization, and mutual exclusion between the app and its extension.

## Key Topics
- **Core motivation** — eliminate post-launch asset downloads by pre-downloading content during app install, app update, and periodic background wakeups; the extension runs before the user has ever launched the app
- **BADownloadManager** — singleton interface for scheduling, observing, promoting, and canceling downloads; used identically in both the app and its extension (foreground download API is app-only)
- **BAURLDownload** — the primary download object type; carries a unique identifier, `URLRequest`, and an app group identifier that determines where the downloaded file lands
- **Download lifecycle** — scheduled downloads run at system-determined times; the app can promote background downloads to foreground (higher priority, immediate start) via `startForegroundDownload(_:)`; promotes without re-downloading already-received bytes
- **BADownloaderExtension protocol** — the extension's entry points: `applicationDidInstall(metadata:)`, `applicationDidUpdate(metadata:)`, `checkForUpdates(metadata:)`, `backgroundDownloadDidFail(failedDownload:)`, `backgroundDownloadDidFinish(finishedDownload:fileURL:)`, and `extensionWillTerminate()`; extension runtime is short-lived — schedule work quickly
- **Delegate vs. extension routing** — if the app has an active `BADownloadManagerDelegate`, download callbacks go to the app; if not, the extension is woken to service them; only shared protocol methods can wake the extension
- **App group requirement** — both the app and extension must belong to the same app group; the group identifier is passed to each `BAURLDownload` and determines the destination container
- **Info.plist configuration** — required keys: `BAInitialDownloadRestrictions` dictionary (containing `BADownloadAllowance` in bytes and `BADownloadDomainAllowList` array of hostnames; enforced only at first install) and `BAMaximumDownloadSize` (uncompressed final size shown on the App Store)
- **Mutual exclusion** — `BADownloadManager.withExclusiveControl(_:)` provides a mutual exclusion block ensuring only the app or extension executes the closure at a time; critical when both processes may handle the same downloaded file

## APIs & Frameworks
**Background Assets framework** **[NEW]**
- `import BackgroundAssets` **[NEW]**
- `BAURLDownload` **[NEW]** — download object; init with `identifier:`, `request: URLRequest`, `applicationGroupIdentifier:`
- `BADownload` **[NEW]** — base class for download objects; carries `identifier` and error state on failure
- `BADownloadManager` **[NEW]** — singleton (`BADownloadManager.shared`); primary interface for all download operations
  - `BADownloadManager.shared` — shared singleton
  - `manager.delegate: BADownloadManagerDelegate?` — set to receive download callbacks in the app
  - `manager.schedule(_ download: BADownload) throws` **[NEW]** — enqueues a download for background execution
  - `manager.startForegroundDownload(_ download: BADownload) throws` **[NEW]** — promotes a download to foreground priority; app-only (not available in extension)
  - `manager.fetchCurrentDownloads() async throws -> [BADownload]` **[NEW]** — returns all currently scheduled/in-flight downloads
  - `manager.cancel(_ download: BADownload) throws` **[NEW]** — cancels a download
  - `manager.withExclusiveControl(_ handler: (Error?) -> Void)` **[NEW]** — executes handler with mutual exclusion between app and extension
- `BADownloadManagerDelegate` protocol **[NEW]**
  - `downloadDidBegin(_ download: BADownload)`
  - `downloadDidPause(_ download: BADownload)`
  - `download(_:bytesWritten:totalBytesWritten:totalExpectedBytes:)`
  - `download(_:didReceive challenge: URLAuthenticationChallenge) async -> (URLSession.AuthChallengeDisposition, URLCredential?)`
  - `download(_:failedWithError error: Error)`
  - `download(_:finishedWithFileURL fileURL: URL)`
- `BADownloaderExtension` protocol **[NEW]** — extension entry point protocol
  - `applicationDidInstall(metadata: BAApplicationExtensionInfo)`
  - `applicationDidUpdate(metadata: BAApplicationExtensionInfo)`
  - `checkForUpdates(metadata: BAApplicationExtensionInfo)`
  - `download(_:didReceive challenge: URLAuthenticationChallenge) async -> (URLSession.AuthChallengeDisposition, URLCredential?)`
  - `backgroundDownloadDidFail(failedDownload: BADownload)`
  - `backgroundDownloadDidFinish(finishedDownload: BADownload, fileURL: URL)`
  - `extensionWillTerminate()`
- `BAApplicationExtensionInfo` **[NEW]** — metadata passed to extension entry points
- `NSBundleResourceRequest` — existing alternative for Apple-hosted on-demand resources

**Info.plist keys**
- `BAInitialDownloadRestrictions` (dictionary) — `BADownloadAllowance` (Int, bytes), `BADownloadDomainAllowList` (Array of String); enforced on first install only
- `BAMaximumDownloadSize` (Int, bytes) — uncompressed total storage requirement; displayed on the App Store

## Code Highlights
Scheduling a background download:
```swift
import BackgroundAssets

let url = URL(string: "https://cdn.example.com/large-asset.bin")!
let appGroupIdentifier = "group.WWDC.AssetContainer"
let download = BAURLDownload(identifier: "Large-Asset",
                             request: URLRequest(url: url),
                             applicationGroupIdentifier: appGroupIdentifier)

let manager = BADownloadManager.shared
manager.delegate = self  // BADownloadManagerDelegate

do {
    try manager.schedule(download)
} catch {
    print("Failed to schedule download. \(error)")
}
```

Promoting background downloads to foreground:
```swift
do {
    for download in try await manager.fetchCurrentDownloads() {
        try manager.startForegroundDownload(download)
    }
} catch {
    print("Failed to promote downloads to foreground \(error)")
}
```

Mutual exclusion when handling a finished download:
```swift
func download(_ download: BADownload, finishedWithFileURL fileURL: URL) {
    let manager = BADownloadManager.shared
    manager.withExclusiveControl { error in
        guard error == nil else { return }
        do {
            let data = try Data(contentsOf: fileURL, options: .mappedIfSafe)
            // process data...
            try FileManager.default.removeItem(at: fileURL)
        } catch {
            print("Unable to read/cleanup file data. \(error)")
        }
    }
}
```

## Takeaways
- Background Assets lets apps pre-download CDN-hosted content before first launch by running a lightweight extension during app install, app update, and periodic background wakeups.
- Use `BAURLDownload` + `BADownloadManager` in both the app and the extension; promote background downloads to foreground as soon as the user opens the app.
- Always use `withExclusiveControl` when the app and extension may both respond to a completed download — it prevents data races on the shared file.
- Configure `BAInitialDownloadRestrictions` and `BAMaximumDownloadSize` in the app's Info.plist accurately; these values are reviewed by App Review and displayed to users on the App Store.

---
_Source: WWDC22 Session 110403 page (abstract, chapter summaries, code samples, and resource links)._
