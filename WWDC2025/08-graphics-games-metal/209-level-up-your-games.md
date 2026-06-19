# Level up your games
**WWDC25 · Session 209** · [Watch](https://developer.apple.com/videos/play/wwdc2025/209/)

_Platforms:_ iOS 26, iPadOS 26, macOS 26

## Overview
This session is a comprehensive tour of the 2025 game developer story on Apple platforms. It covers performance modes, cloud save, new input frameworks, background asset delivery, Metal 4 rendering features, and the revamped Game Center / Apple Games app ecosystem — all in one place to help developers plan their adoption priorities.

Game Mode and Sustained Execution Mode have matured: iOS 26 extends Game Mode to iPhone with LSSupportsGameMode, and the new entitlement-gated Sustained Execution Mode lets games request predictable, throttle-free CPU and GPU headroom. Background Assets now has a managed tier that handles delta updates and integrity checking, dramatically reducing first-launch wait times.

On the rendering side, Metal 4 brings new GPU timeline primitives, and MetalFX gains frame interpolation and denoising as first-class upscaling companions — giving developers a full latency and quality budget management toolkit.

## Key Topics

### Game Mode and Sustained Execution
`LSSupportsGameMode` in the Info.plist enables Game Mode on iPhone (previously iPad/Mac only). A new entitlement activates Sustained Execution Mode, which requests that the system hold CPU/GPU clocks steady. Apps observe power-state changes via `NSProcessInfoPowerStateDidChange` to gracefully step down when the device exits sustained mode.

### GameSave Framework
`GSSyncedDirectory` is the new cross-device cloud save primitive. It presents a local directory whose contents are transparently synced via iCloud, with conflict resolution callbacks for games that need custom merge logic.

### Touch Controls Framework
A new framework ships for designing and displaying on-screen virtual gamepads and control overlays, built to integrate directly with the existing Game Controller framework event model.

### Managed Background Assets
Background Assets gains a managed tier where the App Store CDN handles differential updates and hash verification. Games declare asset manifests; the system downloads only changed chunks. This integrates with the existing `BGAssetPack` model but removes manual diffing logic.

### Metal 4 and MetalFX
Metal 4 introduces new GPU work submission primitives for tighter CPU–GPU synchronization. MetalFX adds frame interpolation (inserts synthetic frames between rendered frames) and AI-based denoising alongside the existing spatial upscaling, enabling developers to trade render resolution for latency headroom.

### Game Center and Apple Games
The Apple Games app is a rebranded, richer hub for multiplayer, achievements, and leaderboards. Game Center APIs are unchanged but surface more prominently in the system.

## APIs & Frameworks

- **LSSupportsGameMode** (Info.plist key) — enables Game Mode on iPhone 26 **[NEW on iPhone]**
- **Sustained Execution Mode** (entitlement) **[NEW]** — requests stable clock speeds
- **NSProcessInfoPowerStateDidChange** — notification for power-state transitions
- **GameSave framework** **[NEW]** — `GSSyncedDirectory` for cloud-synced save data
- **Game Controller framework** — unchanged API, now works with Touch Controls
- **Touch Controls framework** **[NEW]** — on-screen virtual gamepad overlay system
- **Background Assets (Managed tier)** **[NEW]** — App Store-managed delta asset downloads
- **Metal 4** — new GPU timeline and work submission primitives
- **MetalFX upscaling** — spatial upscaling (existing)
- **MetalFX frame interpolation** **[NEW]** — synthetic frame insertion for higher perceived frame rate
- **MetalFX denoising** **[NEW]** — AI-based denoising pass
- **Metal Performance HUD** — runtime overlay for GPU counters (existing, improved)
- **Game Center** — leaderboards, achievements, multiplayer (existing APIs)

## Code Highlights

```swift
// Observe power state to adapt quality settings
NotificationCenter.default.addObserver(
    forName: .NSProcessInfoPowerStateDidChange,
    object: nil, queue: .main
) { _ in
    let isLowPower = ProcessInfo.processInfo.isLowPowerModeEnabled
    renderQuality = isLowPower ? .reduced : .full
}
```

```swift
// GameSave: open a synced directory
let syncedDir = try GSSyncedDirectory(name: "SaveData")
let saveURL = syncedDir.url.appendingPathComponent("slot1.dat")
try gameData.write(to: saveURL)
```

## Takeaways

- Adopt `LSSupportsGameMode` in your Info.plist immediately — it costs nothing and users get lower-latency input on iPhone 26.
- `GSSyncedDirectory` replaces iCloud Document boilerplate for game saves; the system handles sync and conflict UI.
- MetalFX frame interpolation can recover significant frame-budget headroom without reducing render resolution — profile before choosing upscaling vs. interpolation.
- The Managed Background Assets tier is the new recommended path; remove custom delta-download logic and let the platform handle it.

---
_Source: WWDC25 Session 209 page (abstract, chapter summaries, code samples, and resource links)._
