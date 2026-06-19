# System Extensions and DriverKit
**WWDC19 · Session 702** · [Watch](https://developer.apple.com/videos/play/wwdc2019/702/)

_Platforms:_ macOS Catalina 10.15

## Overview
macOS Catalina introduces System Extensions and DriverKit as modern, user-space replacements for Kernel Extensions (Kexts). Kernel Extensions run inside the kernel, making them difficult to develop/debug, and a single bug can compromise the security or reliability of the entire system. System Extensions run in user space outside the kernel, enabling full use of macOS frameworks, Swift, modern debugging tools, and safe crash recovery — all without requiring a kernel restart.

This session introduces three types of System Extensions (Network, Driver, Endpoint Security), demonstrates building a USB driver with the new DriverKit SDK, shows live LLDB debugging of a driver without rebooting, and explains how to package, sign, and install System Extensions within a macOS app.

## Key Topics

### Why Kernel Extensions Are Being Deprecated
- Kexts run inside the kernel: above the system's own security rules, with access to everything on the machine.
- Any bug in a Kext can be a critical security or reliability problem — crashes cause kernel panics and full system restart.
- Kext development requires C/C++, forbids most system frameworks, and requires a second machine for debugging.
- macOS 10.15 Catalina is the **last release to fully support Kernel Extensions without compromises** for device families and capabilities covered by System Extensions. Future releases will not load deprecated Kext types.

### Three Types of System Extensions **[NEW]**
1. **Network Extensions** — replace Network Kernel Extensions; filter/reroute traffic, connect to VPNs.
2. **Driver Extensions (Dext)** — replace IOKit device driver Kexts; built with DriverKit SDK.
3. **Endpoint Security Extensions** — replace KAUTH-based security Kexts; support EDR and DLP.

### DriverKit SDK **[NEW]**
- A separate SDK (not the macOS SDK) with new frameworks based on IOKit but updated for user space.
- Limited API surface by design (no direct filesystem, networking, or Mach messaging) — enables elevated process priority and security.
- Must be written in C or C++17 (no Swift).
- Interfaces defined in `.iig` files, processed by the IIG tool (similar to `.defs` for MIG).
- IIG files compile with Clang and can import C/C++ headers; class/method attributes indicate kernel vs. user implementations.

### DriverKit Core Classes **[NEW]**
- `IOService` — lifecycle methods (`Start`, `Stop`, `Terminate`); each instance has a default `IODispatchQueue`.
- `IODispatchQueue` — built on pre-run GCD, optimized for DriverKit's restricted runtime.
- `IOMemoryDescriptor` / `IOBufferMemoryDescriptor` — memory management.
- `OSAction` — encapsulates asynchronous callbacks (replaces C function pointers); type-checked at compile and runtime.
- Dispatch sources for interrupts and timers (similar to `IOWorkLoop`/`IOEventSource` in IOKit).

### Available DriverKit Families (macOS Catalina) **[NEW]**
- `NetworkingDriverKit` — create network interfaces.
- `HIDDriverKit` — create HID devices.
- `USBSerialDriverKit` — expose USB serial devices to the OS.
- `USBDriverKit` — use USB device providers in drivers.

### Driver Extension Lifecycle
- When a matching device appears, IOKit matching creates a kernel proxy service, then starts a user-space process hosting the Driver Extension.
- Dexts appear in `IORegistryExplorer` and respond to IOKit Framework APIs just like Kexts.
- On crash: the process restarts automatically without affecting the rest of the system.
- Instance variables in a Dext must be allocated at init time in an `IVars` struct (no class-level storage in IIG).
- Use the `IMPL` macro on `Start`/`Stop` implementations to enable IPC between the user process and kernel proxy.

### Packaging and Distribution **[NEW]**
- System Extensions are always part of an app — not standalone; distributed via Developer ID or the Mac App Store (first time for device drivers).
- Embedded at: `<App>.app/Contents/Library/SystemExtensions/`.
- **Driver Extension bundles:** `.dext` suffix, package type `DEXT`, flat bundle, `OSBundle*` keys in Info.plist.
- **Other System Extensions:** `.systemextension` suffix, package type `SYSX`.
- Xcode provides templates and auto-embeds extensions via Copy Files phase.
- Sign with the same Developer ID certificate as the app (no special Kext certificate needed).
- Must be notarized before running on user systems.

### Entitlements **[NEW]**
- All Driver Extensions need a base DriverKit entitlement.
- Transport entitlement (per device type): grants control of a specific hardware family.
- Family entitlement: grants ability to expose a service type to the OS.
- App containing the extension needs `com.apple.developer.system-extension.install`.
- Cross-team packaging (e.g., a USB chip driver included by multiple vendors) is supported via a special entitlement.
- Request entitlements at `developer.apple.com/system-extensions`.

### Installation and Lifecycle Management **[NEW]**
- Apps use the new `SystemExtensions` framework to submit activation requests.
- Activation requests should be submitted at app launch; if already activated and unchanged, they return immediately with success.
- The system manages the extension's runtime lifecycle (starts on device connect, etc.).
- Updates: when the app updates, the next activation request detects version change and upgrades the extension automatically.
- Uninstall: moving the app to Trash deactivates all its System Extensions automatically — no separate uninstaller needed.
- `OSSystemExtensionRequest` — the activation request API.
- `OSSystemExtensionRequest.deactivationRequest(forExtensionWithIdentifier:queue:delegate:)` — for manual deactivation.

### Debugging
- Attach LLDB to the driver process by PID — no second machine, no special cables.
- The kernel keeps running if the driver crashes; the driver process restarts automatically.
- Full LLDB capabilities: expression evaluation, object printing, thread inspection.
- System Integrity Protection can be disabled during development to bypass code signing/entitlement checks (re-enable for testing release builds).

## APIs & Frameworks

### DriverKit SDK (user-space) **[NEW]**
- `IOService` — base driver class; `Start()`, `Stop()`, `Terminate()`, `init()`
- `IODispatchQueue` — GCD-based queue for driver events
- `IOMemoryDescriptor` — DMA-capable memory
- `IOBufferMemoryDescriptor` — allocated buffer memory
- `OSAction` — typed async callback wrapper
- `IOUSBHostInterface` — USB interface provider
- `IOUSBHostPipe` — USB pipe for I/O
- `IOSharedDataQueueDispatchSource` — shared memory ring buffer for fast IPC
- IIG tool — generates IPC stubs from `.iig` class definitions

### SystemExtensions Framework (macOS app) **[NEW]**
- `OSSystemExtensionManager.shared` — shared extension manager
- `OSSystemExtensionRequest.activationRequest(forExtensionWithIdentifier:queue:delegate:)` — activate an extension **[NEW]**
- `OSSystemExtensionRequest.deactivationRequest(forExtensionWithIdentifier:queue:delegate:)` — deactivate **[NEW]**
- `OSSystemExtensionRequestDelegate` — callbacks for activation result **[NEW]**

### Info.plist Keys **[NEW]**
- `OSBundleUsageDescription` — user-facing description for Driver Extensions
- `NSSystemExtensionUsageDescription` — description for other System Extensions
- `CFBundleDisplayName` — user-visible extension name
- `OSBundleIdentifier` — Driver Extension bundle ID

## Code Highlights

IIG class definition (driver interface):
```cpp
// MyDriver.iig
class KERNEL MyDriver : public IOService {
public:
    virtual kern_return_t Start(IOService* provider) override;
    virtual void Stop(IOService* provider) override;
    
    virtual void ReadComplete(OSAction* action,
                              IOReturn status,
                              uint64_t actualByteCount) TYPE(IOUSBHostPipe::CompleteAsyncIO);
};
```

Driver Start implementation:
```cpp
kern_return_t IMPL(MyDriver, Start) {
    kern_return_t ret = Start(provider, SUPERDISPATCH);
    if (ret != kIOReturnSuccess) return ret;
    // open USB interface, allocate pipe, enqueue read...
    return kIOReturnSuccess;
}
```

Activating a System Extension from the app:
```swift
let request = OSSystemExtensionRequest.activationRequest(
    forExtensionWithIdentifier: "com.example.app.driver",
    queue: .main,
    delegate: self
)
OSSystemExtensionManager.shared.submitRequest(request)
```

## Takeaways
- Kernel Extensions are deprecated in macOS Catalina; begin migrating to System Extensions and DriverKit immediately.
- DriverKit drivers run in user space, can be debugged with LLDB on a single machine, crash safely without a kernel panic, and can be distributed through the Mac App Store for the first time.
- System Extensions are embedded in the app bundle and managed by the `SystemExtensions` framework — no installer or uninstaller required.
- Request required entitlements (transport, family, install) at `developer.apple.com/system-extensions` before submission.

---
_Source: WWDC19 Session 702 page (abstract, chapter summaries, code samples, and resource links)._
