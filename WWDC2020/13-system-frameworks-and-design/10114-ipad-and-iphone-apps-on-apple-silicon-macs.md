# iPad and iPhone apps on Apple silicon Macs
**WWDC20 · Session 10114** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10114/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11 (Apple silicon)

## Overview
macOS Big Sur on Apple silicon Macs can run existing iPhone and iPad apps natively without recompilation, using the same Mac Catalyst infrastructure that underlies all unified frameworks on the Mac. The same binary that ships on the iOS App Store is delivered to Mac users through the Mac App Store — no source changes required. Compatible apps automatically receive a significant set of macOS behaviors (window chrome, menu bar, keyboard/mouse input mapping, scroll bars, etc.) by virtue of running on the shared UIKit layer.

Developers who want deeper Mac integration should use Mac Catalyst (checkbox in Xcode), which allows full customization and distribution on all Macs, not just Apple silicon. For the "as-is" path, compatibility is automatic for most apps; the key development task is testing on Apple silicon Mac hardware to identify any gaps caused by hardware, UI, or system-software differences between iOS devices and the Mac.

## Key Topics

### How It Works
- Apple silicon Macs run the exact same iOS binary through the Mac Catalyst infrastructure (unified frameworks, same iOS SDK)
- No recompilation required; compatible apps automatically appear in the Mac App Store
- iOS apps on Apple silicon get macOS window chrome, menu bar integration, pointer/keyboard mapping, and system UI adaptations automatically
- Apps that need deeper Mac customization should target Mac Catalyst (macOS SDK build)

### Hardware Differences to Consider
- Multi-touch vs. cursor interaction: system maps common gestures, but custom touch handling needs testing
- Sensors unavailable on Mac: accelerometer, gyroscope, magnetometer, depth camera, GPS
  - Use conditional API availability checks (e.g., `CLLocationManager` still works with reduced precision)
  - Use `AVCaptureDeviceDiscoverySession` instead of enumerating all capture devices to find the best available camera
- Screen resolution varies dramatically across Macs; do not hardcode assumptions matching iOS device sizes

### UI Differences to Consider
- Alerts and popovers may appear at different screen positions on Mac
- Open/Save panels appear in separate windows
- Do not hardcode assumptions about system UI placement
- iPad apps that support multitasking become fully resizable windows on Mac (including live resize)
- Apps without multitasking support or iPhone apps run in a fixed-size, non-resizable window
- Use Auto Layout to support any window size and improve live resize performance

### System Software Differences
- App's data containers are in a different location on Mac; use Foundation APIs (not hardcoded paths) to locate files
- Device model strings will not be "iPhone" / "iPad" — server-side or app-side code that checks for specific device identifiers must handle unexpected values gracefully
- Some APIs that depend on unavailable hardware still function in limited form (e.g., Core Location)

### Testing and Debugging in Xcode
- New run destination: "My Mac (Designed for iPad)" **[NEW]** — builds against the iOS SDK, not macOS SDK
- Full Xcode debugging support: breakpoints, variable inspection, LLDB expression evaluation
- View Debugger and Memory Debugger work for iOS apps running on Mac
- Instruments supported for performance profiling
- XCTest unit tests run natively on Apple silicon Mac (built against iOS SDK)
- No Simulator involved — the app runs natively

### Distribution
- Compatible apps are automatically made available in the Mac App Store after signing the updated Developer Agreement
- Same Xcode Organizer workflow: export via App Store Connect, ad hoc, enterprise, or development
- All App Store features work: in-app purchases, subscriptions (StoreKit), on-demand resources
- App thinning extended to Apple silicon Macs; a Mac is treated as a "very capable iPad" for thinning purposes
- New virtual thinning destination "Mac" in Xcode Organizer produces an install variant optimized for Apple silicon Macs
- OTA (Over-the-Air) installation manifests automatically serve an appropriate Mac variant — no changes needed
- Managed device MDM deployment works for Apple silicon Macs
- TestFlight is NOT available for iPad/iPhone apps on Mac; use ad hoc or development distribution for pre-release testing
- App Store Connect provides opt-out controls per app (Pricing and Availability page; uncheck the Mac availability checkbox)

## APIs & Frameworks

- **Mac Catalyst** — underlying infrastructure enabling iOS apps on Apple silicon Macs **[NEW path]**
- **UIKit** — runs on Mac via Mac Catalyst layer; system provides automatic adaptations
- **Core Location** (`CLLocationManager`) — works on Mac with reduced precision (no GPS)
- **AVFoundation**
  - `AVCaptureDeviceDiscoverySession` — preferred API to find the best available camera, works correctly when multiple cameras may be present on Mac
- **Foundation** file-system APIs — use `FileManager` / URL-based APIs (not hardcoded paths) to locate app containers
- **Auto Layout** — required for resizable window support when app supports multitasking / Mac
- **StoreKit** — in-app purchases and subscriptions work on Mac identically to iOS
- **On-Demand Resources** — supported identically on Mac
- **App Thinning** — extended with a new Mac thinning target **[NEW]**
- **XCTest** — unit tests run natively on Apple silicon Mac against iOS SDK
- **Xcode run destination: "My Mac (Designed for iPad)"** **[NEW]**

## Code Highlights

No new Swift/Objective-C API code was introduced in this session. The key guidance is defensive coding patterns:

```swift
// Check for hardware availability instead of assuming
if CLLocationManager.locationServicesEnabled() {
    // use location; will work on Mac with less precision
}

// Use discovery session instead of enumerating all devices
let discoverySession = AVCaptureDevice.DiscoverySession(
    deviceTypes: [.builtInWideAngleCamera],
    mediaType: .video,
    position: .unspecified
)
let bestCamera = discoverySession.devices.first
```

Opt out of Mac availability in App Store Connect via the Pricing and Availability page (no code change).

## Takeaways
- Existing iOS apps run on Apple silicon Macs as the exact same binary with zero code changes — the main developer task is testing to find behavioral issues.
- Use `AVCaptureDeviceDiscoverySession`, Foundation URL APIs, dynamic feature checks, and Auto Layout to make apps resilient across the hardware differences between iOS devices and Macs.
- Opt into deeper Mac customization via Mac Catalyst (Xcode checkbox) to gain access to macOS SDK and all Mac customization APIs, and to support distribution on Intel Macs as well.
- TestFlight is unavailable for this path; use ad hoc or development distribution for pre-release Mac testing.

---
_Source: WWDC20 Session 10114 page (abstract and transcript)._
