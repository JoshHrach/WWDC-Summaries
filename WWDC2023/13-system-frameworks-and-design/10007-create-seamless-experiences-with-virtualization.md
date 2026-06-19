# Create Seamless Experiences with Virtualization
**WWDC23 · Session 10007** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10007/)

_Platforms:_ macOS Sonoma 14

## Overview
The Virtualization framework in macOS Sonoma gains six new capabilities that make virtual machines more seamless, flexible, and performant. The session covers: a resizable display that auto-adjusts VM resolution to match the host window; save and restore for suspending a running VM to disk and resuming it later; Network Block Device (NBD) attachment for remote/custom storage backends; NVMe controller emulation for Linux VMs without virtio drivers; the Mac keyboard for Apple-specific key mapping; and a Rosetta 2 translation cache daemon that speeds up x86_64 workloads in Linux VMs.

The session is structured as a practical guide with code snippets for each feature, all building on the foundation established in "Create macOS or Linux virtual machines" (WWDC22).

## Key Topics

### Resizable Display
A single new property—`VZVirtualMachineView.automaticallyReconfiguresDisplay`—enables the VM's display to dynamically resize when the host window is resized. Previously, display resolution was fixed at boot. Setting the property to `true` causes the framework to automatically communicate the new resolution to the guest OS, making use of all available space without user intervention.

### Save and Restore
VMs can now be suspended to disk and resumed later, similar to closing a laptop lid:
1. Pause the running VM with `VZVirtualMachine.pause()`.
2. Call `VZVirtualMachine.saveMachineStateTo(url:)` with a `.vzvmsave` file URL to write all runtime state (CPU registers, memory, device state) to disk.
3. External resources (disk images, auxiliary storage) must be copied separately—they are not included in the save file.
4. To restore: create a new `VZVirtualMachine` from the same configuration, call `restoreMachineStateFrom(url:)` with the save file URL, then call `resume()`.

Security: save files are hardware-encrypted; only the same Mac and user account that saved them can restore them. Files are versioned, and if a format change prevents restoration, the framework returns specific error codes so the app can fall back to a cold boot.

Use cases include: pausing work, rewinding to a previous checkpoint, or backing up VM state at a known-good point.

### Network Block Device (NBD) Storage
Virtualization framework now implements the NBD client protocol, enabling storage to be served from any NBD-compatible server—local or remote:
- Initialize a `VZNetworkBlockDeviceStorageDeviceAttachment` with an `nbd://` URL specifying host, port, and disk name.
- Use the attachment with any supported device type (e.g., `VZVirtioBlockDeviceConfiguration`).
- Attach a delegate to `VZNetworkBlockDeviceStorageDeviceAttachment` to receive connection state change notifications (useful for pausing the VM or reconnecting when the server is unreachable).

NBD is especially valuable for data center deployments where disk images live on storage servers, or for apps that need custom I/O (custom image formats, physical drive passthrough) fully transparent to the guest OS.

### NVMe Controller (Linux VMs)
macOS Sonoma adds `VZNVMExpressControllerDeviceConfiguration` for Linux VMs that lack virtio drivers. NVMe is a standardized controller widely supported in stock Linux kernels. This enables running unmodified Linux kernel builds in virtual machines. NVMe also supports the NBD attachment for remote storage. For VMs that do have virtio drivers, virtio block storage remains the recommended choice for performance.

### Mac Keyboard
A new Mac keyboard device passes Apple-specific keys—including the Globe key—through to a virtual Mac guest, enabling language switching, emoji panel, and other Globe-key features to work natively inside the VM.

### Rosetta 2 Caching Daemon
Rosetta 2 translates x86_64 binaries on demand in Linux VMs. In macOS Sonoma, a new caching daemon runs alongside the Rosetta 2 runtime inside the VM. Pre-translated binaries are stored in a shared cache; subsequent processes that load the same libraries or executables fetch translations from the cache instead of retranslating. This eliminates redundant translation overhead on exec-heavy tasks like compilation and package installation.

Setup: configure the Rosetta 2 communication channel via the new API, then launch the caching daemon inside the Linux VM. The runtime and daemon negotiate a connection through Virtualization framework automatically.

## APIs & Frameworks

**Virtualization (macOS Sonoma — New)**
- `VZVirtualMachineView.automaticallyReconfiguresDisplay` **[NEW]** — auto-resize VM display to match host window
- `VZVirtualMachine.pause()` — suspend execution (prerequisite for save)
- `VZVirtualMachine.saveMachineStateTo(url:)` **[NEW]** — serialize running VM state to `.vzvmsave` file
- `VZVirtualMachine.restoreMachineStateFrom(url:)` **[NEW]** — restore serialized VM state
- `VZVirtualMachine.resume()` — resume a paused/restored VM
- `VZNetworkBlockDeviceStorageDeviceAttachment(url:)` **[NEW]** — NBD client storage attachment
- `VZNetworkBlockDeviceStorageDeviceAttachment.delegate` **[NEW]** — connection state change notifications
- `VZNVMExpressControllerDeviceConfiguration` **[NEW]** — NVMe controller for Linux VMs
- Mac keyboard device configuration **[NEW]** — pass Apple-specific keys to virtual Mac
- Rosetta 2 caching daemon API **[NEW]** — shared translation cache channel configuration

## Code Highlights

Enable resizable display:
```swift
let virtualMachineView = VZVirtualMachineView()
virtualMachineView.virtualMachine = virtualMachine
virtualMachineView.automaticallyReconfiguresDisplay = true
```

Save a running VM:
```swift
try await virtualMachine.pause()
let saveFileURL = URL(filePath: "SaveFile.vzvmsave", directoryHint: .notDirectory)
try await virtualMachine.saveMachineStateTo(url: saveFileURL)
```

Restore a VM from a save file:
```swift
let configuration = VZVirtualMachineConfiguration()
// Customize configuration (must match saved configuration)
let virtualMachine = VZVirtualMachine(configuration: configuration)
let saveFileURL = URL(filePath: "SaveFile.vzvmsave", directoryHint: .notDirectory)
try await virtualMachine.restoreMachineStateFrom(url: saveFileURL)
try await virtualMachine.resume()
```

Configure a virtio block device with NBD attachment:
```swift
let url = URL(string: "nbd://localhost:10809/myDisk")!
let attachment = try VZNetworkBlockDeviceStorageDeviceAttachment(url: url)
let blockDevice = VZVirtioBlockDeviceConfiguration(attachment: attachment)
```

Add a delegate to respond to NBD connection changes:
```swift
let attachment = try VZNetworkBlockDeviceStorageDeviceAttachment(url: url)
let delegate = NetworkBlockDeviceAttachmentDelegate()
attachment.delegate = delegate
let blockDevice = VZVirtioBlockDeviceConfiguration(attachment: attachment)
```

## Resources
- [Virtualization framework documentation](https://developer.apple.com/documentation/Virtualization)
- [Running macOS in a virtual machine on Apple silicon](https://developer.apple.com/documentation/Virtualization/running-macos-in-a-virtual-machine-on-apple-silicon)
- Related: "Create macOS or Linux virtual machines" (WWDC22 10002)

## Takeaways
- `automaticallyReconfiguresDisplay = true` is a one-liner that dramatically improves the VM window experience; apply to both virtual Mac and Linux VMs.
- Save/restore enables backup, rewind, and suspend workflows; save files are hardware-encrypted and versioned, so handle restoration errors gracefully with a cold-boot fallback.
- NBD opens up remote and custom-format storage backends with no guest-side changes; attach a delegate to respond to connection drops in networked deployments.
- NVMe controller support means unmodified stock Linux kernels can run in Virtualization framework VMs without virtio drivers.

---
_Source: WWDC23 Session 10007 page (abstract, transcript, chapter summaries, code samples, and resource links)._
