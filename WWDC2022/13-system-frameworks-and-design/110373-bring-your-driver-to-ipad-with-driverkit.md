# Bring Your Driver to iPad with DriverKit
**WWDC22 · Session 110373** · [Watch](https://developer.apple.com/videos/play/wwdc2022/110373/)

_Platforms:_ iPadOS 16 (M1 iPads only), macOS Ventura 13

## Overview
DriverKit is coming to iPadOS 16, enabling USB, PCI, and Audio drivers on M1 iPads for the first time. This is the same DriverKit used on macOS — existing Mac DriverKit drivers can be brought to iPad without any source code changes. The session covers the path to bringing a macOS DriverKit driver to iPad, new AudioDriverKit real-time callback support, updated entitlements, user client communication security, and the update lifecycle for apps containing drivers.

DriverKit drivers on iPadOS run as bundled extensions inside App Store apps, just like on macOS. Driver approval is handled in the iOS Settings app (General > Drivers or within the app's Settings bundle). The `SystemExtensions` framework is not used on iPadOS — driver installation and approval flow entirely through Settings.

## Key Topics

### DriverKit on iPadOS
- Supported on all iPads with M1; requires iPadOS 16
- USB, PCI, and AudioDriverKit families supported **[NEW on iPadOS]**
- Same source code as macOS DriverKit — no changes needed for basic adoption
- Use Xcode 14 multiplatform app targets to build one app + driver for both Mac and iPad
- Automatic signing in Xcode 14 handles both Mac and iPadOS provisioning
- Distribution via App Store; add `DriverKit` to `UIRequiredDeviceCapabilities` to restrict install to supported devices

### Driver Approval on iPadOS (vs. macOS)
- No `SystemExtensions` framework on iPadOS
- Users approve drivers in **Settings > General > Drivers** (all apps) or within app's Settings bundle (requires embedding a Settings bundle in the app)
- App should use `UIApplication.open(_:)` to send user to Settings bundle for driver approval
- Driver launches on demand when hardware is plugged in (not on approval)
- Drivers can be toggled on/off per driver from Settings

### AudioDriverKit Updates
- New real-time IO operation callback via `IOUserAudioDevice.SetIOOperationHandler(_:)` **[NEW]**
- `IOOperationHandler` block: called from real-time context on IO operations for audio stream buffers
- Block receives: `IOUserAudioObjectID`, `IOUserAudioIOOperation` (e.g., `.writeEnd`, `.beginRead`), buffer frame size, sample time, host time
- New audio family entitlement: `com.apple.developer.driverkit.family.audio` replaces `allow-any-userclient-access` for AudioDriverKit drivers **[NEW, available since macOS 12.1]**

### User Client Communication & Entitlements
- Apps use `IOKit.framework` to open user clients to drivers
- **Communicates With Drivers** entitlement (iPadOS) **[NEW]**: `com.apple.developer.driverkit.communicates-with-drivers` — grants app ability to open user clients to drivers; replaces the macOS `driverkit.userclient-access` entitlement on iPadOS
- **Allow Third Party User Clients** entitlement (driver) **[NEW]**: `com.apple.developer.driverkit.allow-third-party-userclients` — allows apps from different team identifiers to communicate with the driver
- Both entitlements are public for development; request distribution entitlement via developer.apple.com

### Driver Update Lifecycle
- Automatic app update installs new app (and bundled driver) automatically
- Driver update takes effect only after hardware is **unplugged** (old driver keeps running until disconnect)
- App may need to communicate with an older driver version after an app update
- Driver approval state is preserved through app updates

## APIs & Frameworks

**DriverKit (iPadOS additions)** **[NEW]**
- USB, PCI, and Audio driver families on iPadOS 16 M1 **[NEW]**
- `UIRequiredDeviceCapabilities`: `"DriverKit"` value to restrict app install to DriverKit-capable iPads

**AudioDriverKit** **[NEW]**
- `IOUserAudioDevice.SetIOOperationHandler(_ handler: IOOperationHandler)` **[NEW]**
- `IOOperationHandler` block type: `(IOUserAudioObjectID, IOUserAudioIOOperation, uint32_t, uint64_t, uint64_t) -> kern_return_t`
- `IOUserAudioIOOperation` — `.writeEnd`, `.beginRead` and other operation types

**Entitlements** **[NEW]**
- `com.apple.developer.driverkit.family.audio` — AudioDriverKit family entitlement for drivers **[replaces allow-any-userclient-access]**
- `com.apple.developer.driverkit.communicates-with-drivers` — iPadOS app entitlement for user client access **[NEW]**
- `com.apple.developer.driverkit.allow-third-party-userclients` — driver entitlement for cross-team user client access **[NEW]**
- All DriverKit family entitlements now public for development

**Xcode 14 Support**
- Automatic signing for DriverKit drivers on both Mac and iPad **[NEW]**
- Multiplatform app targets — single app target delivers driver on Mac and iPad

## Code Highlights

AudioDriverKit real-time IO operation callback:
```cpp
// Declare and register an IOOperationHandler block on the IOUserAudioDevice
io_operation = ^kern_return_t(IOUserAudioObjectID in_device,
                              IOUserAudioIOOperation in_io_operation,
                              uint32_t in_io_buffer_frame_size,
                              uint64_t in_sample_time,
                              uint64_t in_host_time) {
    if (in_io_operation == IOUserAudioIOOperationWriteEnd) {
        // Process outgoing audio buffers
    } else if (in_io_operation == IOUserAudioIOOperationBeginRead) {
        // Process incoming audio buffers
    }
    return kIOReturnSuccess;
};
this->SetIOOperationHandler(io_operation);
```

Sending user to Settings for driver approval (iPadOS app):
```swift
// Open app's Settings bundle for driver approval
Button("Enable Driver in Settings") {
    if let url = URL(string: UIApplication.openSettingsURLString) {
        UIApplication.shared.open(url)
    }
}
```

## Takeaways
- Existing macOS DriverKit drivers run on iPadOS 16 M1 with zero code changes — just add iPad as a supported destination in Xcode 14.
- Driver approval on iPadOS goes through Settings (not `SystemExtensions` framework); apps should guide users with a Settings bundle and a button that opens it.
- Drivers only start running when hardware is plugged in — approval in Settings alone is not enough to start the driver.
- After an app update, the running driver version stays active until the hardware is unplugged; design your app to gracefully handle communicating with older driver versions.

---
_Source: WWDC22 Session 110373 page (abstract, chapter summaries, code samples, and resource links)._
