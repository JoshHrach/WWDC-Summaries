# Getting the Most Out of Simulator
**WWDC19 · Session 418** · [Watch](https://developer.apple.com/videos/play/wwdc2019/418/)

_Platforms:_ iOS, tvOS, watchOS (Simulator on macOS 10.15 Catalina, Xcode 11)

## Overview
This session covers the Simulator at three levels: a technical explanation of how it works (separate userspace running on the Mac kernel), a practical Q&A of tips and features most developers miss, and a command-line deep-dive into `xcrun simctl` for automation. It concludes with the headline new feature of Xcode 11 on macOS Catalina: native GPU acceleration via Metal, enabling full Metal API support in the Simulator for the first time.

The session makes clear that Simulator is not an emulator — it runs the real iOS/tvOS/watchOS userspace natively on the Mac's processor, sharing the kernel but with a separate home directory, daemon set, and process namespace. Memory/CPU limits of the target device are not enforced.

## Key Topics

**What Simulator Is (Technical)**
- Separate userspace: own launchd, daemons, notification domains, URL sessions, mach bootstrap
- Shares macOS kernel and filesystem root; separate Home directory per simulated device
- Built natively for the Mac's CPU; not an emulator
- Memory and CPU device limits not enforced; application sandbox not enforced
- Case-sensitive filesystem enforced for all Simulator processes
- Thread Sanitizer supported in Simulator (not available on device)

**Simulator App Tips (UI Features)**
- Pinch gesture: Option+drag
- Drag & drop: Control+click-hold, then drag; supports app bundles, photos, videos (live photo pairs auto-combined), location URLs, arbitrary URLs
- Safari Share Sheet includes Simulator targets
- Slow Animations: Debug menu → Slow Animations
- Rotate Device Automatically: Hardware menu toggle
- Dark mode: Settings app → Developer → Dark Appearance
- iCloud: Settings app; manual sync via Debug → Trigger iCloud Sync
- Shake Gesture: Hardware menu
- Siri: available with microphone permission; Hardware menu or keyboard shortcut
- External displays: adjustable resolution in Hardware menu
- tvOS Remote: Hardware menu software remote (Option+move to activate); pair hardware Apple TV Remote with Menu+Volume Up; game controllers and keyboard also work
- QuickPath swipe keyboard: supported
- Simulator sizes: Physical Size, Point Accurate, Pixel Accurate, or free-drag resize
- Install older runtimes: Xcode Preferences → Components
- Hide/delete simulators: Devices & Simulators window (Window menu or keyboard shortcut)
- Xcode 10 + iOS 13 Simulator: launch Xcode 11 first to get iOS 13 runtime, then leave Simulator open and switch to Xcode 10

**`xcrun simctl` Command-Line Interface**
- `simctl list` — list device types, runtimes, devices; `--json` for machine-readable output; filter by category (`devices`) or search string
- `simctl create <name> <device-type> <runtime>` — creates a simulator; returns identifier on stdout
- `simctl boot <device>` / `simctl shutdown <device|all>` — boot/shutdown; `all` shuts down all running simulators
- `simctl delete <device|unavailable>` — delete by ID or remove all with unavailable runtime (recover disk space)
- `simctl clone <source> <name>` — APFS-cloned copy of a device; fast parallel test setup without disk duplication
- `simctl spawn <device> <binary> [args]` — run a process inside the simulated environment (e.g., `defaults write`, `log stream`)
- `simctl launch <device> <bundle-id> [args]` — launch an installed app; supports user-defaults overrides (`-Key Value`); `--console-pty` connects stdin/stdout/stderr to terminal; signals (SIGINT, SIGUSR1, SIGUSR2) pass through
- `simctl install <device> <app-bundle>` — install an app from a bundle path
- `simctl diagnose [-l]` — collect logs and system state for bug reports; `-l` skips privacy warning for CI use
- `simctl addmedia <device> <files>` — add photos/videos/live photos programmatically
- `simctl get_app_container <device> <bundle-id> [app|data|groups]` — path to app's container or shared group on disk
- `simctl io <device> screenshot <path>` — automate screenshots
- `simctl pair/unpair/pair_activate` — manage Watch-iPhone pairings from command line
- `booted` alias: substitute for device UDID when exactly one simulator is booted

**Metal in Simulator (NEW in Xcode 11 / macOS Catalina)**
- All Apple frameworks migrated to Metal renderers — GPU acceleration is automatic with no code changes
- Metal API fully available in Simulator; renders via translation from iOS Metal to macOS Metal (not emulation)
- `MTLGPUFamilyCommon1` and `MTLGPUFamilyApple2` supported
- Performance reflects the Mac GPU, not a target device GPU — always profile and optimize on real hardware before shipping
- Storage mode caveats: multisampled, depth stencil, and linear textures must use `.private` storage in Simulator (device supports `.shared` for these); use `BlitCommandEncoder` to copy from intermediate shared buffer/texture if CPU access is needed

## APIs & Frameworks

**Metal (Xcode 11 / macOS Catalina)** **[NEW in Simulator]**
- `MTLDevice.supportsFamily(_:)` **[NEW]** — query GPU family support (replaces older feature query API)
- `MTLGPUFamily` **[NEW]**: `.common1`, `.apple2`, `.mac1` — Simulator supports `.common1` and `.apple2`
- `MTLTextureDescriptor.storageMode` — must be `.private` for multisampled/depth stencil/linear textures in Simulator
- `MTLBlitCommandEncoder` — use to copy between shared and private textures; `copy(from:sourceSlice:sourceLevel:sourceOrigin:sourceSize:to:destinationSlice:destinationLevel:destinationOrigin:)`

**xcrun simctl** (command-line tool, ships with Xcode)
- All subcommands listed above; `xcrun simctl help [subcommand]` for full documentation

## Code Highlights

Simulator-conditional storage mode for textures that need CPU access:
```swift
let desc = MTLTextureDescriptor.texture2DDescriptor(...)
#if targetEnvironment(simulator)
desc.storageMode = .private
#else
desc.storageMode = .shared
#endif
let texture = device.makeTexture(descriptor: desc)!

#if targetEnvironment(simulator)
// Initialize via intermediate shared buffer + blit
let buffer = device.makeBuffer(bytes: data, length: dataSize, options: .storageModeShared)!
let blit = commandBuffer.makeBlitCommandEncoder()!
blit.copy(from: buffer, sourceOffset: 0, sourceBytesPerRow: ..., ...)
blit.endEncoding()
#else
// Initialize directly on CPU
texture.replace(region: ..., mipmapLevel: 0, withBytes: data, bytesPerRow: ...)
#endif
```

Clone-based parallel test setup with simctl:
```bash
# Set up base simulator
BASE=$(xcrun simctl create "BaseTest" "iPhone 11" "com.apple.CoreSimulator.SimRuntime.iOS-13-0")
xcrun simctl boot "$BASE"
xcrun simctl install booted /path/to/MyApp.app
xcrun simctl shutdown "$BASE"

# Clone for parallel runs
CLONE1=$(xcrun simctl clone "$BASE" "Test-1")
CLONE2=$(xcrun simctl clone "$BASE" "Test-2")
xcrun simctl boot "$CLONE1" && xcrun simctl boot "$CLONE2"
```

## Takeaways
- Simulator is the real OS userspace on your Mac kernel — memory/CPU limits are Mac limits, not device limits; always verify on real hardware.
- `xcrun simctl clone` with APFS cloning is the key to fast, disk-efficient parallel test environments in CI.
- `xcrun simctl diagnose` should be attached to every Simulator-related bug report filed via Feedback Assistant.
- Metal GPU acceleration in Xcode 11 on macOS Catalina is automatic for framework-based UI; Metal API apps need only handle the `.private` storage mode caveat for specific texture types.

---
_Source: WWDC19 Session 418 page (abstract, chapter summaries, code samples, and resource links)._
