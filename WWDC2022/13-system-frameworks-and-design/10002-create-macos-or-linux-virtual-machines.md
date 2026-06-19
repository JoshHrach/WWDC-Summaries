# Create macOS or Linux Virtual Machines
**WWDC22 · Session 10002** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10002/)

_Platforms:_ macOS Ventura 13

## Overview
This session covers the Virtualization framework's new capabilities in macOS Ventura for running both virtual Macs and full unmodified Linux distributions on Apple silicon. The framework sits above Hypervisor framework (the low-level CPU/memory virtualization layer) and provides a high-level Swift/Objective-C API for configuring and operating virtual machines with just a few dozen lines of code.

For macOS VMs, the session introduces the Mac-specific platform configuration (`VZMacPlatformConfiguration`), the automated restore image installer API, GPU acceleration via Metal, the new Mac trackpad device, and a Virtio file-system share that automounts in the guest. For Linux, the session covers booting unmodified ISO distributions via EFI, Virtio GPU 2D for rendering Linux UI in a macOS window, USB mass storage for installing from an ISO, and a new Rosetta 2 directory share that lets x86-64 Linux binaries run at near-native speed inside an ARM Linux VM.

## Key Topics

### Virtualization Framework Architecture
Apple silicon hardware virtualizes CPUs and memory at the SoC level. The macOS kernel exposes this through Hypervisor framework (low-level C API). Virtualization framework wraps Hypervisor with a high-level API supporting complete macOS and Linux VMs. Two object categories: **configuration objects** (define hardware) and **virtual machine objects** (operate the VM).

### macOS Virtual Machines
- `VZMacPlatformConfiguration` holds three unique properties: `hardwareModel`, `auxiliaryStorage`, and `machineIdentifier`
- `VZMacOSBootLoader` provides macOS boot support
- `VZMacOSRestoreImage.latestSupported` downloads the latest macOS installer; `.mostFeaturefulSupportedConfiguration` returns compatible hardware requirements
- `VZMacOSInstaller` automates installation from a restore image
- Metal GPU acceleration via `VZMacGraphicsDeviceConfiguration` with configurable display resolution and pixel density
- New `VZMacTrackpadConfiguration` (macOS Ventura) enables gesture support (pinch, rotate, etc.) in the virtual Mac
- `VZVirtioFileSystemDeviceConfiguration` with `macOSGuestAutomountTag` automounts a shared host directory in the guest

### Linux Virtual Machines
- Boot from ISO via `VZUSBMassStorageDeviceConfiguration` (virtual USB drive)
- EFI boot loader (`VZEFIBootLoader` + `VZEFIVariableStore`) enables booting unmodified Linux distributions
- `VZVirtioGraphicsDeviceConfiguration` with `VZVirtioGraphicsScanoutConfiguration` renders Linux UI in `VZVirtualMachineView`
- `VZLinuxRosettaDirectoryShare` exposes the Rosetta 2 translator to the Linux VM; Linux uses `update-binfmts` to register Rosetta as the handler for x86-64 ELF binaries

## APIs & Frameworks

### Virtualization — Core
- `VZVirtualMachineConfiguration` — root configuration object; holds `cpuCount`, `memorySize`, `bootLoader`, `storageDevices`, `graphicsDevices`, `pointingDevices`, `directorySharingDevices`
- `VZVirtualMachine(configuration:)` — creates a VM instance
- `VZVirtualMachine.start() async` — starts the VM
- `VZVirtualMachineView` — `NSView` subclass; set `virtualMachine` property to display guest UI

### Virtualization — macOS
- `VZMacPlatformConfiguration` **[NEW]** — platform object for virtual Macs; properties: `hardwareModel`, `auxiliaryStorage`, `machineIdentifier`
- `VZMacHardwareModel(dataRepresentation:)` — restores a saved hardware model
- `VZMacAuxiliaryStorage(contentsOf:)` — attaches auxiliary storage file
- `VZMacMachineIdentifier(dataRepresentation:)` — restores a saved machine identifier
- `VZMacOSBootLoader` — boot loader for macOS guests
- `VZMacOSRestoreImage.latestSupported async` **[NEW]** — fetches metadata for the latest supported macOS restore image
- `VZMacOSRestoreImage.url` — download URL for the restore image
- `VZMacOSRestoreImage.mostFeaturefulSupportedConfiguration` — returns `VZMacOSConfigurationRequirements`
- `VZMacOSConfigurationRequirements.hardwareModel` — compatible hardware model
- `VZMacOSConfigurationRequirements.minimumSupportedCPUCount` / `minimumSupportedMemorySize`
- `VZMacOSInstaller(virtualMachine:restoringFromImageAt:)` — installer object
- `VZMacOSInstaller.install() async` — runs the installation
- `VZMacGraphicsDeviceConfiguration` — GPU-accelerated display for macOS guests; `displays: [VZMacGraphicsDisplayConfiguration]`
- `VZMacGraphicsDisplayConfiguration(widthInPixels:heightInPixels:pixelsPerInch:)` — defines display resolution
- `VZMacTrackpadConfiguration` **[NEW]** — enables multi-touch trackpad gestures in macOS guests (requires macOS 13 host + guest)
- `VZSharedDirectory(url:readOnly:)` — wraps a host directory for sharing
- `VZSingleDirectoryShare(directory:)` — shares a single directory
- `VZMultipleDirectoryShare(directories:)` — shares multiple named directories
- `VZVirtioFileSystemDeviceConfiguration(tag:)` — Virtio FS device; use `macOSGuestAutomountTag` for automatic guest mounting
- `VZVirtioFileSystemDeviceConfiguration.macOSGuestAutomountTag` **[NEW]** — special tag that triggers automount in macOS guests

### Virtualization — Linux
- `VZEFIBootLoader` **[NEW]** — EFI-based boot loader for unmodified Linux distributions
- `VZEFIVariableStore(creatingVariableStoreAt:options:)` **[NEW]** — creates a new EFI variable store file
- `VZEFIVariableStore(url:)` — opens an existing EFI variable store
- `VZDiskImageStorageDeviceAttachment(url:readOnly:)` — attaches a disk image (e.g., ISO) as storage
- `VZUSBMassStorageDeviceConfiguration(attachment:)` **[NEW]** — virtual USB mass storage device (for booting ISO installers)
- `VZVirtioGraphicsDeviceConfiguration` **[NEW]** — Virtio GPU 2D for Linux guests; `scanouts: [VZVirtioGraphicsScanoutConfiguration]`
- `VZVirtioGraphicsScanoutConfiguration(widthInPixels:heightInPixels:)` **[NEW]** — virtual display for Virtio GPU
- `VZLinuxRosettaDirectoryShare` **[NEW]** — exposes Rosetta 2 translator to ARM Linux VMs for x86-64 binary support
- `VZUSBScreenCoordinatePointerDeviceConfiguration` — USB pointer device for Linux mouse input

## Code Highlights

Starting a VM and displaying its screen:
```swift
let virtualMachine = VZVirtualMachine(configuration: configuration)
try await virtualMachine.start()
let view = VZVirtualMachineView()
view.virtualMachine = virtualMachine
```

Installing macOS into a new VM:
```swift
let restoreImage = try await VZMacOSRestoreImage.latestSupported
let requirements = restoreImage.mostFeaturefulSupportedConfiguration!
platform.hardwareModel = requirements.hardwareModel
let installer = VZMacOSInstaller(virtualMachine: virtualMachine, restoringFromImageAt: imageURL)
try await installer.install()
```

Enabling Rosetta 2 in a Linux VM:
```swift
let rosettaShare = try VZLinuxRosettaDirectoryShare()
let device = VZVirtioFileSystemDeviceConfiguration(tag: "RosettaShare")
device.share = rosettaShare
configuration.directorySharingDevices = [device]
```

## Takeaways
- The Virtualization framework provides a complete, high-level Swift API to create and run macOS and Linux VMs in fewer than 50 lines of code, with no kernel extensions required.
- macOS Ventura adds `VZMacTrackpadConfiguration` (gesture support), `macOSGuestAutomountTag` (auto-mounting shared folders), and GPU acceleration improvements.
- Linux support is significantly enhanced: EFI boot enables unmodified distro installation from ISO, Virtio GPU 2D renders full Linux desktop UI in a window, and `VZLinuxRosettaDirectoryShare` brings Rosetta 2 performance to x86-64 Linux binaries.
- DAL plug-ins are deprecated; this is unrelated to Virtualization but shares the macOS Ventura context.

---
_Source: WWDC22 Session 10002 page (abstract, chapter summaries, code samples, and resource links)._
