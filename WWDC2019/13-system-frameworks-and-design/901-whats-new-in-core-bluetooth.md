# What's New in Core Bluetooth
**WWDC19 · Session 901** · [Watch](https://developer.apple.com/videos/play/wwdc2019/901/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15, watchOS 6, tvOS 13

## Overview
Core Bluetooth in iOS 13 expands in four major directions: Bluetooth 5.0 performance improvements (LE 2 Mbps physical layer and advertising extensions for faster, more power-efficient connections), full GATT-over-BR/EDR support enabling Core Bluetooth apps to interact with classic Bluetooth devices like speakers, headphones, and car head units for the first time, dual-mode enhancements via Cross Transport Key Derivation (CTKD) and transport bridging for seamless LE + BR/EDR experiences, and mandatory user authorization for all Core Bluetooth APIs alongside new privacy controls for ANCS accessory notifications. Developer tooling gains Live Capture in PacketLogger for real-time Bluetooth packet analysis from iOS devices.

## Key Topics

**LE 2 Mbps Physical Layer**
Bluetooth 5.0 doubles the radio throughput from 1 to 2 Mbps between compatible devices. Core Bluetooth handles all link-layer PHY negotiations automatically — no API changes needed. Benefits: twice the data in the same airtime, improved power efficiency. Available on iPhone 8/8 Plus/X and later, Apple TV 4K, Apple Watch Series 4 and later. Requires the accessory to also support LE 2 Mbps.

**Advertising Extensions**
Bluetooth 5.0 advertising extensions reduce congestion on the three legacy advertising channels by sending a small pointer on the advertising channel and the full payload on the data channel. Payload grows from 31 to 255 bytes (up to 124 bytes supported by Core Bluetooth scan); rate can be LE 2 Mbps. Core Bluetooth scans for both legacy and extended advertisements automatically via the same scan API. Extended connections make connection establishment more robust: the advertiser sends an explicit connection response (vs. the scanner assuming success), and connections can start in LE 2 Mbps. A new API allows programmatic query of platform support. Available on iPhone XS and new iPad Pro (2018+).

**Core Bluetooth for BR/EDR (Classic Bluetooth) — NEW**
Core Bluetooth now supports GATT over BR/EDR, enabling apps to interact with classic Bluetooth peripherals (speakers, headphones, car head units, keyboards, game pads) using the same `CBPeripheral` APIs used for LE devices. `CBCentralManager` gains `registerForConnectionEvents(options:)` to receive a delegate callback when a system-connected BR/EDR device exposes a registered service UUID or known peripheral UUID — without the app having initiated the connection. Outgoing connections to known BR/EDR peripherals use the existing `connect(_:options:)` API. Available on iOS 13, watchOS 6, and tvOS 13. Accessories can add GATT support via a firmware update (no new hardware required).

**Dual-Mode: Cross Transport Key Derivation (CTKD)**
CTKD (Bluetooth 4.2) derives link keys for both LE and BR/EDR transports from a single pairing. Result: a dual-mode device appears as one `CBPeripheral` with a single identifier rather than two separate entries in Bluetooth settings. Enables in-app pairing flow: connect over LE, pair, and CTKD automatically derives the BR/EDR key — no user trip to Bluetooth settings required.

**Dual-Mode: Transport Bridging — NEW**
A new `CBConnectPeripheralOptionTransportBridgingKey` option in `CBCentralManager.connect(_:options:)` allows an LE connection to automatically trigger BR/EDR profile connections (A2DP, HFP, AVRCP). When the key is set, Core Bluetooth connects over LE and immediately pages over BR/EDR to bring up media profiles. Use case: a home audio device app can trigger music/podcast playback automatically when the user comes into proximity of a speaker.

**User Authorization — NOW REQUIRED**
Previously only CBPeripheralManager advertising required user authorization. In iOS 13, any use of Core Bluetooth APIs requires user authorization. Apps built on older SDKs are also affected. Requires `NSBluetoothAlwaysUsageDescription` in Info.plist — failure to add this key causes a crash at launch. Users can modify the permission in Settings → Privacy → Bluetooth or in the app-specific settings. On watchOS, authorization is shared between the paired iPhone app and watch extension for non-standalone watch apps. New `CBManager.authorization` property (replacing the deprecated `CBCentralManager.state` for auth checking) reports the current authorization status.

**ANCS Privacy**
Apple Notification Center Service (ANCS) now requires user authorization before an accessory can receive iOS notifications. When an iOS 13 device connects to an ANCS accessory, a permission prompt is shown. New `CBConnectPeripheralOptionNotifyOnNotificationKey` connect option triggers the ANCS permission prompt in-app at connection time (more contextually appropriate). The `CBPeripheral.ancsAuthorized` property reflects the current authorization state, and the delegate receives `peripheralDidUpdateANCSAuthorization(_:)` when it changes.

**PacketLogger Live Capture — NEW**
The Bluetooth PacketLogger app (in Xcode Additional Tools) gains Live Capture: connect an iOS 13 device to a Mac, install the developer logging profile, and capture live Bluetooth traffic in real time. Supports multiple simultaneous iOS devices. Previously only post-hoc sysdiagnose capture was available. Best performance on macOS Catalina.

## APIs & Frameworks

**Core Bluetooth**
- `CBCentralManager.registerForConnectionEvents(options:)` **[NEW]** — register service UUID or peripheral UUID to receive a delegate callback when a BR/EDR device with matching attributes connects
- `CBCentralManagerDelegate.centralManager(_:connectionEventDidOccur:for:)` **[NEW]** — delegate callback for registered connection events; includes `CBConnectionEvent` type
- `CBCentralManager.connect(_:options:)` — existing API; now supports BR/EDR peripherals
- `CBConnectPeripheralOptionTransportBridgingKey` **[NEW]** — connect option to bridge LE connection to BR/EDR media profiles (A2DP, HFP, AVRCP)
- `CBConnectPeripheralOptionNotifyOnNotificationKey` **[NEW]** — triggers ANCS permission prompt in-app during connection
- `CBPeripheral.ancsAuthorized: Bool` **[NEW]** — ANCS authorization state for this peripheral
- `CBPeripheralDelegate.peripheralDidUpdateANCSAuthorization(_:)` **[NEW]** — delegate callback when ANCS authorization changes
- `CBManager.authorization: CBManagerAuthorization` **[NEW]** — replaces checking `state == .unauthorized`; values: `.allowedAlways`, `.denied`, `.notDetermined`, `.restricted`
- `CBManagerDelegate.centralManagerDidUpdateState(_:)` — check `state == .unauthorized` then read `authorization` for `.denied` vs other unauthorized states
- `CBCentralManager.supportsFeatures(_:)` **[NEW]** — programmatic query for LE 2 Mbps, extended scan, extended connections support on current hardware
- `CBCentralManagerFeature.extendedScanAndConnect` **[NEW]** — feature flag for extended scanning and connections (iPhone XS / new iPad Pro+)

**Info.plist**
- `NSBluetoothAlwaysUsageDescription` **[NEW required key]** — usage description string; mandatory for all Core Bluetooth apps on iOS 13; crash on launch if absent

## Code Highlights

Registering for BR/EDR connection events:

```swift
let manager = CBCentralManager(delegate: self, queue: nil)

func centralManagerDidUpdateState(_ central: CBCentralManager) {
    guard central.state == .poweredOn else { return }
    // Register for connections to a known service UUID (e.g., Heart Rate)
    let heartRateUUID = CBUUID(string: "180D")
    manager.registerForConnectionEvents(options: [.serviceUUIDs: [heartRateUUID]])
}

func centralManager(_ central: CBCentralManager,
                    connectionEventDidOccur event: CBConnectionEvent,
                    for peripheral: CBPeripheral) {
    if event == .peerConnected {
        manager.connect(peripheral, options: nil)
    }
}
```

Checking authorization and handling all manager states:

```swift
func centralManagerDidUpdateState(_ central: CBCentralManager) {
    switch central.state {
    case .poweredOn:
        startScanning()
    case .unauthorized:
        if CBManager.authorization == .denied {
            showSettingsAlert()
        }
    case .poweredOff:
        showBluetoothOffMessage()
    case .resetting, .unknown, .unsupported:
        break
    @unknown default:
        break
    }
}
```

Using transport bridging to trigger BR/EDR media profiles:

```swift
// When user wants to stream audio to a known dual-mode speaker
manager.connect(speaker, options: [CBConnectPeripheralOptionTransportBridgingKey: true])
// Core Bluetooth connects over LE, then pages BR/EDR to bring up A2DP/HFP
```

Requesting ANCS authorization in-app:

```swift
manager.connect(peripheral, options: [CBConnectPeripheralOptionNotifyOnNotificationKey: true])
// Permission prompt shown in-app after pairing

func peripheralDidUpdateANCSAuthorization(_ peripheral: CBPeripheral) {
    if peripheral.ancsAuthorized {
        // Accessory can now receive notifications
    }
}
```

## Takeaways
- Core Bluetooth for BR/EDR opens up an entirely new class of accessories (speakers, headsets, car systems, keyboards) to GATT-based apps — the same `CBPeripheral` APIs apply with only one new registration API for incoming connections.
- CTKD eliminates the "two Bluetooth entries" UX problem for dual-mode accessories; pairing once over LE automatically establishes the BR/EDR bond.
- `NSBluetoothAlwaysUsageDescription` is now mandatory for every Core Bluetooth app on iOS 13 — add it immediately or existing apps will crash on update.
- PacketLogger Live Capture replaces the slow sysdiagnose-and-open workflow, making Bluetooth debugging dramatically more interactive; install the developer logging profile on test devices now.

---
_Source: WWDC19 Session 901 page (abstract, transcript, and resource links)._
