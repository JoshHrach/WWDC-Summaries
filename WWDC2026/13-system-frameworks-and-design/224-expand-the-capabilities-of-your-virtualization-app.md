# Expand the capabilities of your Virtualization app
**WWDC26 · Session 224** · [Watch](https://developer.apple.com/videos/play/wwdc2026/224/)

_Platforms:_ macOS

## Overview
macOS 27 brings five major capability expansions to the Virtualization framework. The session walks through each in turn: automating the first-boot Setup Assistant for virtual Macs, passing USB accessories directly into VMs via the new Accessory Access framework, building complex multi-VM network topologies with the vmnet framework, managing layered disk images with the new DiskImageKit framework, and implementing custom paravirtualized devices using the standard Virtio protocol.

Together these additions make it practical to build fully automated CI/testing pipelines (provision VMs headlessly, share a base disk image across many VMs, pass through hardware under test), developer tools (persistent Linux VMs with direct USB access), and specialized embedded-systems simulators (custom Virtio devices that bridge host and guest).

DiskImageKit is entirely new in macOS 27 and enables space-efficient layered disk images — a base layer shared read-only across all VMs, an optional cache layer, and a per-VM writable overlay — reducing both storage and I/O overhead in multi-VM scenarios.

## Key Topics

### macOS Guest Provisioning
`VZMacGuestProvisioningOptions` lets you supply a username, full name, password, auto-login preference, and SSH (Remote Login) enablement directly in `VZMacOSVirtualMachineStartOptions` at first boot, replacing the need for an interactive Setup Assistant.

### Accessory Access (USB Passthrough)
The `AccessoryAccess` framework lets a host app register for USB attachment events and pass a matching `AAUSBAccessory` directly into a running VM as a `VZUSBPassthroughDevice`. The user retains explicit control over which physical devices are passed through.

### Advanced Network Topologies
Integrating the `vmnet` framework with `Virtualization` allows full control over network architecture — creating shared-mode or custom networks, bridging multiple VMs, and configuring port forwarding — rather than using the default NAT attachment.

### DiskImageKit (NEW in macOS 27)
A new high-performance framework for creating, opening, and composing layered disk images. Layers can be stacked (base → cache → overlay) and the resulting `DiskImage` is directly consumable as a `VZDiskImageStorageDeviceAttachment`.

### Custom Virtio Devices
`VZCustomVirtioDeviceConfiguration` lets you define a custom paravirtualized device by specifying a Virtio device ID, PCI class/subclass IDs, and queue count. A delegate receives notifications when the guest enqueues work items, enabling high-performance host↔guest communication without a full-blown USB or network stack.

## APIs & Frameworks

### Virtualization
- `VZMacGuestProvisioningOptions` **[NEW]** — carries username, password, full name, auto-login, SSH flags
- `VZMacOSVirtualMachineStartOptions.setGuestProvisioning(_:)` **[NEW]** — attaches provisioning options to a VM start
- `VZVirtualMachine.start(options:)` — async start with options
- `VZUSBPassthroughDeviceConfiguration(device:)` **[NEW]** — wraps an `AAUSBAccessory` for VM attachment
- `VZUSBPassthroughDevice` **[NEW]** — the live device handle; attach via USB controller
- `VZVirtualMachine.usbControllers` — array of USB controller objects
- `VZVmnetNetworkDeviceAttachment(network:)` **[NEW]** — creates a network attachment from a vmnet network handle
- `VZVirtioNetworkDeviceConfiguration` — standard Virtio NIC configuration
- `VZDiskImageStorageDeviceAttachment(diskImage:)` **[NEW overload]** — accepts a `DiskImage` from DiskImageKit
- `VZVirtioBlockDeviceConfiguration(attachment:)` — block device wrapping the attachment
- `VZCustomVirtioDeviceConfiguration` **[NEW]** — configure a custom Virtio device (deviceID, pciClassID, pciSubclassID, virtioQueueCount)
- `VZCustomVirtioDeviceDelegateProvider` **[NEW]** — wraps a queue and delegate for device I/O
- `VZCustomVirtioDeviceConfigurationDelegate` protocol **[NEW]** — `customVirtioConfiguration(_:didCreateDevice:)` called when VM starts
- `VZCustomVirtioDevice` **[NEW]** — live device handle; set `delegate` to receive queue notifications
- `VZCustomVirtioDeviceDelegate` protocol **[NEW]** — `customVirtioDevice(_:didReceiveNotificationFor:)`
- `VZVirtioQueue` **[NEW]** — `nextElement()` returns a `VZVirtioQueueElement`; call `returnToQueue()` when processed
- `VZVirtualMachine.customVirtioDevices` **[NEW]** — array property to register custom devices

### DiskImageKit (NEW framework, macOS 27)
- `DiskImage` **[NEW]** — primary type; open with `DiskImage(opening:)` or compose with `.appending(_:)`
- `DiskImage.OpenDescriptor` — `.open(url:mode:)` factory; modes include `.readOnly`, `.readWrite`
- `DiskImage.AppendDescriptor` — `.asifLayer(url:type:)` for cache layers
- Layer types: base layer (read-only), cache layer (`.cache`), overlay layer (writable, per-VM)

### AccessoryAccess (NEW framework)
- `AAUSBAccessoryManager.shared` — singleton manager
- `AAUSBAccessoryManager.registerListener(_:matchingCriteria:)` **[NEW]** — async; returns currently connected accessories
- `AAUSBAccessoryMatchingCriteria` **[NEW]** — filter by vendor ID, product ID, etc.
- `AAUSBAccessoryListener` protocol **[NEW]** — `usbAccessoryDidConnect(_:)` / `usbAccessoryDidDisconnect(_:)`
- `AAUSBAccessory` **[NEW]** — represents a connected USB device

### vmnet
- `vmnet_network_configuration_create(_:_:)` — create a network configuration (e.g., `.VMNET_SHARED_MODE`)
- `vmnet_network_create(_:_:)` — instantiate the network, returns opaque handle
- `vmnet_return_t` — status enum (`.VMNET_SUCCESS`, `.VMNET_FAILURE`, etc.)

## Code Highlights

Provision a macOS guest at first boot:
```swift
let opts = VZMacGuestProvisioningOptions()
opts.username = "admin"; opts.password = "secret"
opts.logsInAutomatically = true; opts.enablesRemoteLogin = true
let startOpts = VZMacOSVirtualMachineStartOptions()
try startOpts.setGuestProvisioning(opts)
try await virtualMachine.start(options: startOpts)
```

Pass a USB device into a running VM:
```swift
func usbAccessoryDidConnect(_ usbAccessory: AAUSBAccessory) {
    let config = VZUSBPassthroughDeviceConfiguration(device: usbAccessory)
    let device = try VZUSBPassthroughDevice(configuration: config)
    vm.usbControllers.first?.attach(device: device) { error in ... }
}
```

Build a layered disk image:
```swift
let base = try DiskImage(opening: .open(url: baseURL, mode: .readOnly))
let cache = try base.appending(.asifLayer(url: cacheURL, type: .cache))
let overlay = try DiskImage(opening: .open(url: overlayURL))
let stacked = try cache.appending(overlay)
let attachment = try VZDiskImageStorageDeviceAttachment(diskImage: stacked)
```

## Takeaways
- `VZMacGuestProvisioningOptions` eliminates the interactive Setup Assistant, enabling fully automated macOS VM provisioning.
- `DiskImageKit` is a new macOS 27 framework for layered, space-efficient disk images — essential for multi-VM workloads like CI farms.
- USB passthrough via `AccessoryAccess` gives users explicit control and apps a clean API for hardware testing scenarios.
- `VZCustomVirtioDeviceConfiguration` enables high-performance host↔guest channels without needing a network or USB stack.

---
_Source: WWDC26 Session 224 page (abstract, chapter summaries, code samples, and resource links)._
