# Get to know Developer Mode
**WWDC22 · Session 110344** · [Watch](https://developer.apple.com/videos/play/wwdc2022/110344/)

_Platforms:_ iOS 16, iPadOS 16, watchOS 9

## Overview
Developer Mode is a new security feature introduced in iOS 16 and watchOS 9 that gates access to powerful development capabilities behind an explicit opt-in. The mode is disabled by default and must be manually enabled on each device. It persists across reboots and OS updates once enabled. The change was motivated by observation that developer APIs were being used in targeted attacks against non-developer users who had no need for these capabilities.

The session covers when Developer Mode is required, how to enable it manually on a single device, and how to automate enrollment across many devices in testing lab environments using the new `devmodectl` command-line tool that ships with macOS Ventura.

## Key Topics

**Why Developer Mode exists** — Powerful developer features (debug attach, instrument access, development-signed app launch) have been exploited in targeted attacks. Restricting them behind an explicit per-device opt-in reduces attack surface for the vast majority of users without impacting developer workflows for those who opt in.

**What requires Developer Mode** — Running or installing development-signed applications (including Personal Team signatures), debugging and instrumenting applications, and running testing automation on device. Distribution via TestFlight, Enterprise in-house distribution, and App Store does not require Developer Mode.

**Enabling Developer Mode manually** — Connect the device to a Mac running Xcode; Xcode will surface a prompt if Developer Mode is off. Alternatively, navigate to Settings > Privacy & Security > Developer Mode. A reboot is required to activate. After reboot, a confirmation prompt appears on device.

**Automation with `devmodectl`** — For testing labs with many devices, macOS Ventura ships `devmodectl` (included by default). Limitation: only devices without a passcode can be automatically enrolled (the device must be accessible immediately after reboot for automation to work).
- `devmodectl streaming` — monitors USB; automatically enables Developer Mode on every device plugged in that has no passcode.
- Single-device variant also available for already-connected devices.

## APIs & Frameworks

### Developer Mode (system feature — no SDK API)
- Settings > Privacy & Security > Developer Mode **[NEW in iOS 16 / watchOS 9]** — the on-device toggle
- Enrollment persists across reboots and OS updates
- Xcode 14 — surfaces a warning and prevents run/debug when Developer Mode is disabled on the target device

### devmodectl (macOS Ventura command-line tool — new)
- `devmodectl streaming` **[NEW]** — streaming mode; auto-enables Developer Mode on passcode-free devices as they connect via USB
- Ships by default with macOS Ventura; no separate download required
- Requirement: target device must have no passcode set (so automation can interact post-reboot)

### Xcode
- Run/debug destination picker — shows warning and blocks installation when Developer Mode is off on the target device
- Connecting a device to Xcode makes the Developer Mode menu item visible in Settings (even without running an app)

## Code Highlights

Automating Developer Mode enrollment for a testing lab:
```bash
# Enable Developer Mode automatically on all connected (passcode-free) devices
devmodectl streaming
```

## Takeaways
- Developer Mode is required on iOS 16 and watchOS 9 for any development, debug, or automation workflow; it is NOT required for TestFlight, Enterprise distribution, or App Store distribution.
- Enable it once via Settings > Privacy & Security > Developer Mode; enrollment survives reboots and OS updates.
- For testing labs with many devices, use `devmodectl streaming` on macOS Ventura to automatically enroll all connected passcode-free devices without manual interaction.
- Devices with a passcode cannot be enrolled via automation — they must be unlocked manually after the required reboot.

---
_Source: WWDC22 Session 110344 page (abstract, transcript, code samples, and resource links)._
