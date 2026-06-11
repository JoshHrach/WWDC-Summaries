# Get the Most Out of Device Hub

**WWDC26 · Session 260** · [Watch](https://developer.apple.com/videos/play/wwdc2026/260/)

_Platforms:_ Xcode 27 · iOS · macOS · iPadOS · tvOS · watchOS · visionOS

## Overview

Device Hub is a new standalone app shipping with Xcode 27 that provides a unified environment for managing, configuring, and interacting with both physical devices and simulators. It replaces the device management surface that was previously scattered across Xcode's Devices and Simulators window, the Simulator app, and various command-line tools, consolidating them into a single consistent interface.

The session walks through Device Hub's four primary capabilities — Control, Organize, Configure, and bug reproduction — using a realistic scenario where one developer captures diagnostics on a physical device and another reproduces the issue on a simulator by mirroring the device's configuration. This end-to-end workflow demonstrates how Device Hub bridges the gap between device-specific bug reports and simulator-based debugging.

The session closes with a look at `devicectl`, the command-line companion tool that exposes Device Hub's capabilities for scripting, CI integration, and automation workflows. For teams that build automation pipelines around Xcode Cloud or custom CI systems, `devicectl` provides the same device management primitives as the GUI without requiring a display.

## Key Topics

### Device Hub Overview
- New app bundled with Xcode 27; available from the Xcode menu and from the toolbar.
- Two layout modes: **compact window** (minimal footprint, quick access) and **full window** (complete feature set).
- Consistent experience across physical devices and simulators.

### Control
- **Canvas**: live display of the device or simulator screen.
- Touch input passthrough — interact directly with the device from the canvas.
- Hardware controls: volume, lock, home button equivalents.
- Zoom and resize mode (test different screen sizes, relevant for iPhone Mirroring).
- Keyboard capture — type on the Mac keyboard into the device.

### Organize
- **Sidebar**: full inventory of connected devices and simulators.
- Filters, sorting, and grouping (by OS, device type, platform).
- Compact windows per device for keeping multiple devices visible simultaneously.
- Pairing new physical devices directly from Device Hub.

### Configure (Five Inspector Panels)
1. **Appearance** — Dark/Light mode, text size, accessibility settings (Dynamic Type, Bold Text, Reduce Motion, etc.)
2. **Conditions** — simulate network conditions, location (GPS coordinates), and sensor overrides.
3. **Diagnostics** — capture sysdiagnose, device logs, console output, crash reports.
4. **Device** — device info, OS version, UDID; install and manage apps; manage configuration profiles and provisioning profiles.
5. **Apps** — install, launch, and remove apps; manage app data (export/import sandbox contents).

### Reproducing a Bug
- Developer A: pairs device, installs logging profile, captures diagnostics, exports app container data.
- Developer B: creates a simulator matching the device's OS, imports the app data, installs the same logging profile, reproduces the issue.
- Device Hub makes the configuration mirroring workflow explicit and guided.

### devicectl
- Command-line tool for scripting and CI integration.
- Key commands: `devicectl list devices`, `devicectl device install-app`, `devicectl device capture-sysdiagnose`, `devicectl device info`.
- Can pipe results as JSON for machine-readable CI output.

## APIs & Frameworks

**Device Hub (Xcode 27)**
- **[NEW]** Device Hub app — bundled with Xcode 27
- Compact window mode
- Full window mode
- Canvas — live display + touch input passthrough
- Resize mode — test different screen sizes (iPhone Mirroring support)
- Keyboard capture
- Device/simulator sidebar with filters, sort, group
- Appearance inspector panel (Dark/Light mode, Dynamic Type, Bold Text, Reduce Motion, etc.)
- Conditions inspector panel (network, location, sensors)
- Diagnostics inspector panel (sysdiagnose, logs, console, crash reports)
- Device inspector panel (device info, app install/remove, profiles)
- Apps inspector panel (app data export/import — sandbox container)
- Logging profile install (from Device Hub UI)
- Device pairing workflow

**devicectl (Command-Line Tool)**
- **[NEW/REVISED]** `devicectl` — unified device management CLI
- `devicectl list devices` — enumerate connected devices
- `devicectl device install-app --bundle-id <id> --device-id <udid> <path>` — install app
- `devicectl device capture-sysdiagnose --device-id <udid>` — capture diagnostics
- `devicectl device info --device-id <udid>` — query device metadata
- JSON output mode for CI pipeline integration

**Related Documentation**
- [Device Hub](https://developer.apple.com/documentation/Xcode/device-hub)

**Related Sessions (WWDC26)**
- What's new in Xcode 27 (258) — Device Hub introduction
- Modernize your UIKit app (278) — resizability testing with Device Hub

## Code Highlights

No Swift/Objective-C code samples in this session. Primary usage is via the Device Hub GUI and `devicectl` CLI.

Key `devicectl` patterns for CI:

```bash
# List all connected devices as JSON
devicectl list devices --json-output devices.json

# Install an app on a specific device
devicectl device install-app --device-id <UDID> /path/to/MyApp.app

# Capture a sysdiagnose for sharing with another developer
devicectl device capture-sysdiagnose --device-id <UDID> --output ./diagnostics/
```

## Takeaways

- Device Hub consolidates device management, simulator control, and diagnostic capture into a single app, eliminating the need to switch between Xcode's Devices window, Simulator.app, and Terminal for common tasks.
- The five inspector panels — Appearance, Conditions, Diagnostics, Device, Apps — cover the full surface of configuration needed to reproduce most device-specific bugs on a simulator.
- The app data export/import workflow (sandbox container sharing) is the key enabler for bug reproduction across developer machines without requiring physical device access.
- `devicectl` makes every Device Hub capability scriptable, enabling CI pipelines that need to install apps, capture diagnostics, or verify device state without a GUI.

---
_Source: WWDC26 Session 260 page (abstract, chapter summaries, and resource links)._
