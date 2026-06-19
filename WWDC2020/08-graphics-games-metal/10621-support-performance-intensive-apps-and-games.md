# Support performance-intensive apps and games
**WWDC20 · Session 10621** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10621/)

_Platforms:_ iOS 14, iPadOS 14

## Overview
iOS 14 introduces a new `UIRequiredDeviceCapabilities` key — `iphone-performance-gaming-tier` — that allows apps and games with exceptional performance requirements to declare a minimum hardware requirement of the **A12 Bionic chip** (or newer). When declared, the App Store prevents the app from being installed on unsupported devices and surfaces the hardware requirement clearly to customers.

This is a last-resort mechanism intended for the narrowest class of apps — console-quality games and pro-level graphics tools — where optimization alone cannot deliver an acceptable experience on older hardware. Apple explicitly discourages using this key without first exhausting every performance optimization technique, since each device generation excluded represents customers who will never see the app.

## Key Topics

### Required Device Capabilities Background
- `UIRequiredDeviceCapabilities` (Info.plist array) — pre-existing mechanism for declaring device feature requirements
- Existing keys: `metal` (A7+), `arkit` (A9+), `arm64`, `opengles-2`, etc.
- The App Store uses this list to gate installation on incompatible devices
- Apps will not launch on devices that do not satisfy the declared capabilities

### New A12 Performance Key (NEW in iOS 14)
- Key: `iphone-performance-gaming-tier` **[NEW]**
- Declares that the app requires an **A12 Bionic chip** or higher
- A12 Bionic capabilities included by this requirement:
  - 6-core CPU (2 performance + 4 efficiency cores)
  - 4-core GPU
  - Second-generation Neural Engine
  - ARKit 3 support (people occlusion, motion capture)
  - Metal GPU Family Apple 5

### Supported Device Classes (A12 or later)
- iPhone XS / XS Max / XR
- iPhone 11 / 11 Pro / 11 Pro Max
- iPhone SE (2nd generation)
- iPad mini (5th generation)
- iPad Air (3rd generation)
- iPad Pro (3rd and 4th generation)

### When to Use This Key
Apple's guidance (in order of preference):
1. **First**: optimize aggressively — use Instruments, Metal System Trace, GPU Frame Debugger, and other profiling tools to bring performance up on older hardware
2. **Only if optimization fails**: after confirming the minimum viable experience genuinely requires A12 performance, add the key
3. The key is appropriate for: console-quality games (PC/console ports), apps requiring Neural Engine inference throughput, apps requiring ARKit 3 real-time people occlusion and motion capture
4. Not appropriate for: apps that could deliver a degraded-but-acceptable experience on older hardware with adaptive quality settings

### App Store Impact
- The App Store displays a hardware requirement badge on the app listing
- Users on incompatible devices see the app as "not compatible" and cannot download it
- Validate with TestFlight before submission to confirm behavior on target device generations

## APIs & Frameworks

**Info.plist**
- `UIRequiredDeviceCapabilities` (Array) — list of required device capability keys
  - `iphone-performance-gaming-tier` **[NEW in iOS 14]** — requires A12 Bionic or later
  - `metal` — requires Metal (A7+)
  - `arkit` — requires ARKit (A9+)

**Metal (A12 / Metal GPU Family Apple 5 features)**
- Metal GPU Family Apple 5 — TBDR improvements, tile shaders, raster order groups, improved memoryless textures (see session 10603 for GPU counters, 10632 for Apple silicon Metal optimization)
- Neural Engine — Core ML, Vision framework acceleration at full Neural Engine throughput

**ARKit (requires A12 for ARKit 3 features)**
- People occlusion — `ARWorldTrackingConfiguration.frameSemantics = .personSegmentation`
- Motion capture — `ARBodyTrackingConfiguration`

**Xcode**
- Info.plist editor — add `UIRequiredDeviceCapabilities` → `iphone-performance-gaming-tier` via the capability dropdown in Xcode 12
- TestFlight — use for validation across device generations before App Store submission

## Code Highlights

Info.plist entry for A12 Bionic requirement:
```xml
<key>UIRequiredDeviceCapabilities</key>
<array>
    <string>arm64</string>
    <string>iphone-performance-gaming-tier</string>
</array>
```

Adaptive quality fallback pattern (recommended before declaring the key):
```swift
// Check GPU family before committing to highest-quality rendering path
if device.supportsFamily(.apple5) {
    // Full A12-tier rendering: ray tracing, tile shaders, memoryless textures
    setupHighQualityRenderer()
} else {
    // Reduced quality for older hardware — scale resolution, simplify shaders
    setupCompatibilityRenderer()
}
```

## Takeaways

- The `iphone-performance-gaming-tier` Info.plist key is a last resort — add it only after exhausting Metal, CPU, and memory optimization, and only when the gap between older hardware and A12 cannot be bridged with adaptive quality settings.
- When declared, the key prevents installation on pre-A12 devices (older than iPhone XS/XR or iPad mini 5th gen) and displays a hardware requirement warning in the App Store — this permanently excludes a large fraction of the installed iOS base.
- A12 Bionic unlocks: Metal GPU Family Apple 5, second-generation Neural Engine, ARKit 3 people occlusion and motion capture — features that cannot be emulated via software on older silicon.
- Validate the restriction using TestFlight before submitting to the App Store to confirm the expected device gating behavior.

---
_Source: WWDC20 Session 10621 page (transcript)._
