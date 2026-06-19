# Supercharge Device Connectivity with Wi-Fi Aware
**WWDC25 · Session 228** · [Watch](https://developer.apple.com/videos/play/wwdc2025/228/)

_Platforms:_ iOS 26, iPadOS 26

## Overview
Wi-Fi Aware is a new framework in iOS 26 and iPadOS 26 that enables peer-to-peer device discovery and direct data connections without requiring an internet connection, a shared Wi-Fi router, or Bluetooth. Built on the Wi-Fi Aware standard, it allows devices to discover one another, exchange service metadata, and establish encrypted data channels over direct Wi-Fi links at up to tens of meters range. The session covers the framework APIs, integration with DeviceDiscoveryUI and AccessorySetupKit for user-facing pairing flows, and how Wi-Fi Aware connections integrate with Network framework's new Swift concurrency APIs.

## Key Topics

### Core Wi-Fi Aware Concepts
Wi-Fi Aware uses publish/subscribe semantics. A device publishing a service broadcasts metadata; subscribing devices discover publishers and can initiate a connection. All traffic is encrypted. Range is significantly greater than Bluetooth and works without infrastructure Wi-Fi.

### Capabilities Check
`WACapabilities` is queried at app launch to determine whether the current device and OS support Wi-Fi Aware. Apps should gracefully handle unsupported devices.

### Publishing a Service
`WAPublishableService` defines the service type, service-specific metadata, and optional connection parameters. The framework handles broadcasting; the app can update metadata at any time.

### Subscribing and Discovering
`WASubscribableService` defines the service type to look for. The app receives discovered peers, including their metadata, and can initiate connections.

### Paired Devices
`WAPairedDevice` represents a previously paired device. The app can connect directly using a stored pairing record without repeating the discovery flow.

### User-Facing Discovery UI
`DeviceDiscoveryUI` (new in iOS 26) presents a system sheet for users to choose among discovered devices, consistent with system UI. Reduces the need for apps to build custom discovery UI.

### AccessorySetupKit Integration
`AccessorySetupKit` (introduced in iOS 18) can now initiate pairing flows that include Wi-Fi Aware device setup alongside Bluetooth and other protocols, enabling a single unified accessory onboarding experience.

### Network Framework Integration
Once a Wi-Fi Aware connection is established, data transfer uses `NetworkConnection`, `NetworkListener`, and `NetworkBrowser` from Network framework (new Swift-concurrency-based APIs introduced in iOS/macOS 26). This means apps can use structured concurrency (`async`/`await`, `AsyncSequence`) for all data I/O over Wi-Fi Aware connections.

## APIs & Frameworks

**Wi-Fi Aware** (new framework) **[NEW]**
- `WACapabilities` **[NEW]** — check device/OS support for Wi-Fi Aware
- `WAPublishableService` **[NEW]** — define and broadcast a Wi-Fi Aware service; set service type and metadata
- `WASubscribableService` **[NEW]** — define and scan for a Wi-Fi Aware service type; receive discovered peers
- `WAPairedDevice` **[NEW]** — represent and connect to a previously paired device

**DeviceDiscoveryUI** **[NEW]**
- System-provided device discovery sheet for user selection of nearby Wi-Fi Aware peers

**AccessorySetupKit** (updated)
- Wi-Fi Aware pairing flow integration **[NEW]** — unified accessory setup for Bluetooth + Wi-Fi Aware devices

**Network framework** (iOS/macOS 26)
- `NetworkConnection` **[NEW]** — Swift concurrency-based data connection (used for Wi-Fi Aware data channels)
- `NetworkListener` **[NEW]** — accept incoming connections
- `NetworkBrowser` **[NEW]** — discover network services

## Code Highlights

```swift
// Check Wi-Fi Aware capability
let capabilities = WACapabilities.current
guard capabilities.isSupported else { return }

// Publish a service
let service = WAPublishableService(serviceType: "com.example.myapp")
service.metadata = customMetadata
try await service.start()

// Subscribe to discover peers
let subscriber = WASubscribableService(serviceType: "com.example.myapp")
for await peer in subscriber.discoveries {
    // peer contains metadata; initiate connection
    let connection = try await peer.connect()
    // Use NetworkConnection APIs for data transfer
}
```

## Takeaways
- Use Wi-Fi Aware for local peer-to-peer connections that need greater range than Bluetooth, work without internet access, and don't require a shared Wi-Fi network.
- Check `WACapabilities` at startup and degrade gracefully on unsupported devices.
- Integrate `DeviceDiscoveryUI` for a consistent, system-provided peer selection UI rather than building custom discovery lists.
- Leverage `AccessorySetupKit` to bundle Wi-Fi Aware pairing into a single unified onboarding flow alongside Bluetooth.
- Once connected, all data I/O uses Network framework's new Swift concurrency `NetworkConnection` APIs — no callback-based code required.

---
_Source: WWDC25 Session 228 page (abstract, chapter summaries, code samples, and resource links)._
