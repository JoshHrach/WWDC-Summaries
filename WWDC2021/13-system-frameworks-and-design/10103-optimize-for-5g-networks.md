# Optimize for 5G Networks
**WWDC21 · Session 10103** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10103/)

_Platforms:_ iOS 15, iPadOS 15

## Overview
5G networks bring dramatically higher bandwidth (up to 4 Gbps theoretical, ~3 Gbps observed at Apple Park), lower latency (as low as 7 ms ping), and the ability to handle more concurrent devices — enabling new app experiences that were previously impractical over cellular. The session covers the evolution from 2G through 5G, explains how iOS intelligently manages network selection to balance performance and battery life, and provides practical guidance for developers to ensure their apps take full advantage of 5G.

iOS manages 5G via two system-level technologies: Automatic Switch to 5G and Smart Data Mode. These evaluate network performance, security characteristics, and power consumption to transparently select the best interface (Wi-Fi, LTE, or 5G) for a given task. Developers should not check network type directly — instead, use the `expensive` and `constrained` path properties exposed by Apple's high-level networking frameworks to adapt behavior appropriately.

The session emphasizes relying on Apple's frameworks (URLSession, Network framework, AVFoundation, CallKit) to automatically benefit from 5G optimizations, rather than writing custom network-type detection logic that can break as network conditions evolve.

## Key Topics
- **5G Network Landscape** — Two deployment modes: Non-Standalone (NSA, built on LTE core, sub-7 GHz + mmWave) and Standalone (SA, new 5G core, improved latency over LTE).
- **Automatic Switch to 5G and Smart Data Mode** — iOS system-level features that transparently select the best network; evaluate performance, security, and power; detect congestion and switch interfaces.
- **Data Mode Settings** — Three user-selectable tiers affecting expensive/constrained path values: Allow More Data on 5G (inexpensive/Wi-Fi-like), Standard (expensive), Low Data Mode (constrained).
- **Best Practice 1: Don't Check Network Type** — Checking for Wi-Fi vs. cellular blocks 5G optimization. Use expensive/constrained APIs only.
- **Best Practice 2: Use Apple Frameworks** — AVFoundation for audio/video streaming; CallKit for VoIP enhancements; URLSession and Network framework for general networking.
- **Best Practice 3: Tune for Constrained and Expensive Paths** — Use `allowsExpensiveNetworkAccess`, `allowsConstrainedNetworkAccess` in URLSession; check `NWPath.isExpensive`, `NWPath.isConstrained` in Network framework; use `AVURLAsset` resource loading hints.

## APIs & Frameworks
- **Foundation / URLSession**
  - `URLSessionConfiguration.allowsExpensiveNetworkAccess` — **[NEW emphasis]** Allow or deny requests on expensive (non-inexpensive 5G / LTE) paths
  - `URLSessionConfiguration.allowsConstrainedNetworkAccess` — Allow or deny requests on constrained (Low Data Mode) paths
- **Network framework**
  - `NWPath` — Represents a current network path
  - `NWPath.isExpensive` — Bool; true when path uses a cellular or expensive interface
  - `NWPath.isConstrained` — Bool; true when Low Data Mode is active
  - `NWPathMonitor` — Monitor path changes to adapt in real time
- **AVFoundation**
  - `AVURLAsset` — Use for adaptive audio/video streaming optimized for the current network
  - `AVAssetResourceLoader` — Intercept resource loading for custom bandwidth-aware behavior
- **CallKit** — VoIP call enhancements on 5G (quality improvements handled by framework)
- **Smart Data Mode** (OS-level) — Automatic network interface selection; no developer API required
- **Automatic Switch to 5G** (OS-level) — Transparent switch between LTE and 5G based on task characteristics

## Code Highlights
No code samples were included in the session transcript. The session's guidance is framework-selection and configuration-based:

```swift
// Prefer this over network type checks:
let config = URLSessionConfiguration.default
config.allowsExpensiveNetworkAccess = true   // allow on non-inexpensive 5G/LTE
config.allowsConstrainedNetworkAccess = false // block on Low Data Mode
let session = URLSession(configuration: config)
```

```swift
// In Network.framework, inspect path properties:
let monitor = NWPathMonitor()
monitor.pathUpdateHandler = { path in
    if path.isConstrained {
        // reduce data usage
    } else if !path.isExpensive {
        // inexpensive path (Allow More Data on 5G or Wi-Fi) — full quality OK
    }
}
monitor.start(queue: .main)
```

## Takeaways
- Never gate behavior on specific network types (Wi-Fi vs. cellular); use `isExpensive` and `isConstrained` path properties instead.
- iOS automatically selects the best available interface via Smart Data Mode; apps get 5G benefits without manual network detection.
- Use high-level frameworks (URLSession, Network, AVFoundation, CallKit) to get system-managed 5G optimizations with minimal code changes.
- Provide a user-facing option to override data policy based on the expensive/constrained signal, since users can manually change Data Mode in Settings.

---
_Source: WWDC21 Session 10103 page (abstract, chapter summaries, code samples, and resource links)._
