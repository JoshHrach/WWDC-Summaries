# Discover Apple-Hosted Background Assets
**WWDC25 · Session 325** · [Watch](https://developer.apple.com/videos/play/wwdc2025/325/)

_Platforms:_ iOS 26, iPadOS 26, macOS Tahoe 26, tvOS 26, visionOS 26

## Overview
This session introduces Managed Background Assets and Apple-Hosted Background Assets — a major evolution of the Background Assets framework that replaces the deprecated On-Demand Resources technology. Developers can now package asset files into typed "asset packs," choose from three download policies (Essential, Prefetch, On-Demand), and either self-host or let Apple host them (200 GB included in Apple Developer Program membership).

A new system-provided downloader extension means most apps need zero custom extension code. A new cross-platform packaging CLI (`ba-package`) and mock server (`ba-serve`) streamline development and testing.

## Key Topics

### Asset Pack Concepts
- **Asset packs** group related files (textures, audio, shaders, etc.) that the system manages as a unit.
- **Download policies**:
  - **Essential** — downloaded as part of app installation; contributes to App Store/TestFlight download progress bar.
  - **Prefetch** — starts during installation but may complete in the background after the app is available.
  - **On-Demand** — only downloaded when `ensureLocalAvailability(of:)` is explicitly called.
- Essential and Prefetch policies can be scoped to `firstInstallation` and/or `subsequentUpdate` installation events.

### Managed Background Assets API (new Swift/ObjC APIs)
- `AssetPackManager.shared` — the central access point for all managed asset pack operations.
- `ensureLocalAvailability(of:)` — async method; returns when the pack is ready locally.
- `statusUpdates(forAssetPackWithID:)` — AsyncSequence of download progress updates.
- `contents(at:searchingInAssetPackWithID:options:)` — returns memory-mapped `Data` for a file inside a pack.
- `descriptor(for:searchingInAssetPackWithID:)` — returns a file descriptor for low-level access (caller must close).
- `remove(assetPackWithID:)` — frees storage when a pack is no longer needed.
- `BAManagedAssetPackDownloadDelegate` — Objective-C delegate for download lifecycle events.
- `StoreDownloaderExtension` / `shouldDownload(_:)` — extension protocol; new system implementation requires no custom code by default.

### Tooling
- `ba-package template` — generates a JSON manifest template (ID, download policy, platforms, file selectors).
- `ba-package` — packages files into a `.aar` compressed archive for upload.
- `ba-serve` — local mock HTTPS server for testing downloads before TestFlight/App Store submission.
- Available via `xcrun` on macOS; standalone binaries for Linux/Windows coming.

### Apple Hosting & App Store Connect
- Upload `.aar` archives via **Transporter** (drag-and-drop) or the **App Store Connect REST API**.
- REST API endpoints: `backgroundAssets`, `backgroundAssetVersions`, `backgroundAssetUploadFiles`.
- Asset pack versions are independent of app build versions; one live version per context (App Store, external beta, internal beta).
- Asset pack updates apply to **all** installed app versions in that context — ensure backwards compatibility.
- Review/distribution flow mirrors app review: internal beta → external beta → App Store.

### Info.plist Keys
- `BAAppGroupID` — shared app group between main app and downloader extension.
- `BAHasManagedAssetPacks` — opt-in flag for Managed Background Assets.
- `BAUsesAppleHosting` — opt-in flag for Apple hosting.
- `BAManifestURL` — required for self-hosting (not needed with Apple hosting).

## Code Highlights

```swift
// Download an asset pack
let assetPack = try await AssetPackManager.shared.assetPack(withID: "Tutorial")
let statusUpdates = AssetPackManager.shared.statusUpdates(forAssetPackWithID: "Tutorial")
Task {
    for await statusUpdate in statusUpdates { /* update progress UI */ }
}
try await AssetPackManager.shared.ensureLocalAvailability(of: assetPack)

// Read a file
let videoData = try AssetPackManager.shared.contents(at: "Videos/Introduction.m4v")

// Remove when done
try await AssetPackManager.shared.remove(assetPackWithID: "Tutorial")
```

```swift
// Minimal downloader extension (system implementation — no custom code needed)
@main
struct DownloaderExtension: StoreDownloaderExtension {
    func shouldDownload(_ assetPack: AssetPack) -> Bool { return true }
}
```

## Takeaways
- Migrate from On-Demand Resources now — ODR is deprecated; Managed Background Assets is the successor.
- Use the **Essential** policy for content required at first launch (e.g., tutorial level) and scope it to `firstInstallation` only to avoid re-downloading on updates.
- Apple-Hosted Background Assets eliminates the need for your own CDN — 200 GB is included at no extra cost.
- Always ensure asset packs remain backwards-compatible with older installed app versions before updating a live pack on the App Store.

---
_Source: WWDC25 Session 325 page (abstract, chapter summaries, code samples, and resource links)._
