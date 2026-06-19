# Add Support for Matter in Your Smart Home App
**WWDC21 · Session 10298** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10298/)

_Platforms:_ iOS 15, iPadOS 15, tvOS 15 (developer preview)

## Overview
Matter (formerly CHIP — Connected Home over IP) is a new open-source smart home protocol developed by Apple and industry partners to enable interoperability across all smart home ecosystems. This session explains how HomeKit in iOS 15 has been extended to transparently support Matter accessories alongside the existing HomeKit Accessory Protocol (HAP), with no code changes required for existing HomeKit apps.

The session covers two integration paths: using existing HomeKit APIs (which simply begin working with Matter accessories) and a new setup API for developers who need to connect Matter accessories directly to their own smart home hubs. It also tours the Matter data model (endpoints, clusters, attributes) and shows the open-source CHIP framework APIs used to interact with Matter devices directly.

## Key Topics

### Matter Integration via HomeKit
- Existing `HMHome.addAndSetupAccessories()` API now also scans and sets up Matter QR codes — no code changes needed
- Remote access, Siri control, automations, scenes, and notifications all work automatically for Matter accessories
- HomeKit calls into the open-source CHIP framework under the hood to communicate with Matter accessories
- All HomeKit APIs support the new Swift async/await concurrency model

### New Setup API for Third-Party Hubs
- New `HMAccessorySetupManager` **[NEW]** — system UI flow for pairing Matter accessories with non-Apple hubs
- `HMCHIPServiceTopology` **[NEW]** — describes the homes managed by the third-party app
- `HMCHIPServiceHome` **[NEW]**, `HMCHIPServiceRoom` **[NEW]** — represent homes and rooms in the topology
- New app extension type with `HMCHIPServiceRequestHandler` **[NEW]** principal class — three methods to implement:
  - `rooms(in:)` — return rooms for a selected home
  - `pairAccessory(in:onboardingPayload:)` — pair using CHIP framework
  - `configureAccessory(named:room:)` — apply name and room configuration
- System provides all UI (QR scanner, home/room selection, naming, error handling)

### Matter Data Model (Protocol Overview)
- **Endpoints** — logically separate features of an accessory (e.g., info endpoint, light endpoint)
- **Clusters** — capabilities within an endpoint (e.g., OnOff, LevelControl, ColorControl); equivalent to HomeKit Services
- **Attributes** — state values within a cluster (e.g., on/off state, brightness level); equivalent to HomeKit Characteristics
- **Actions** — read, write, report (subscribe for notifications)

### CHIP Framework (Open Source APIs)
- `CHIPDeviceController.shared()` — singleton controller for Matter interactions
- `CHIPDeviceController.getPairedDevice(_:)` — get handle to paired accessory by device ID
- Cluster objects (e.g., `CHIPOnOff`) initialized with device, endpoint, and queue
- Cluster methods: `toggle(_:)`, `readAttributeOnOff(responseHandler:)`, etc.

### Multi-Admin and Connected Services
- Matter allows multiple admins (e.g., Apple Home + third-party hub) to connect to the same accessory simultaneously
- New "Connected Services" section in Home app lists all connected admins
- Users can manage or remove admins and re-enable pairing mode

## APIs & Frameworks

### HomeKit
- `HMHome.addAndSetupAccessories(completionHandler:)` — extended to support Matter QR codes **[EXTENDED]**
- `HMAccessorySetupManager` **[NEW]** — manages the Matter accessory setup flow for third-party hubs
  - `addAndSetUpAccessories(for:)` — async/await API to launch setup with a topology
- `HMCHIPServiceTopology(homes:)` **[NEW]** — topology object listing app's homes
- `HMCHIPServiceHome(uuid:name:)` **[NEW]** — represents a home in the topology
- `HMCHIPServiceRoom` **[NEW]** — represents a room in a home
- `HMCHIPServiceRequestHandler` **[NEW]** — principal class for the CHIP service app extension
  - `rooms(in:) async throws -> [HMCHIPServiceRoom]` **[NEW]**
  - `pairAccessory(in:onboardingPayload:) async throws` **[NEW]**
  - `configureAccessory(named:room:) async throws` **[NEW]**

### CHIP Framework (Matter open source)
- `CHIPDeviceController` — singleton; `shared()`, `getPairedDevice(_:)`
- `CHIPDevicePairingDelegate` — protocol for pairing callbacks
- `CHIPOnOff(device:endpoint:queue:)` — OnOff cluster
  - `toggle(_:)` — toggle on/off state
  - `readAttributeOnOff(responseHandler:)` — read current state
- Additional cluster types follow same pattern (LevelControl, ColorControl, etc.)

## Code Highlights

Using existing HomeKit API to add a Matter accessory:
```swift
home.addAndSetupAccessories() { error in
    if let error = error {
        print("Error occurred in accessory setup \(error)")
    } else {
        print("Successfully added accessory to HomeKit")
    }
}
```

New async/await API for third-party hub setup:
```swift
let homes = proprietaryHomeStorage.homes.map { home in
    HMCHIPServiceHome(uuid: home.uuid, name: home.name)
}
let topology = HMCHIPServiceTopology(homes: homes)
let setupManager = HMAccessorySetupManager()
do {
    try await setupManager.addAndSetUpAccessories(for: topology)
} catch {
    print("Error occurred in accessory setup \(error)")
}
```

App extension request handler:
```swift
class RequestHandler: HMCHIPServiceRequestHandler, CHIPDevicePairingDelegate {
    override func rooms(in home: HMCHIPServiceHome) async throws -> [HMCHIPServiceRoom] { ... }
    override func pairAccessory(in home: HMCHIPServiceHome, onboardingPayload: String) async throws { ... }
    override func configureAccessory(named name: String, room: HMCHIPServiceRoom) async throws { ... }
}
```

## Takeaways
- Existing HomeKit apps automatically gain Matter accessory support in iOS 15 with no code changes — the same APIs now work for both HAP and Matter accessories.
- The new `HMAccessorySetupManager` API allows third-party smart home apps to pair Matter accessories directly to their own hubs through a system-provided, Apple-quality setup UI.
- Matter's data model (endpoints → clusters → attributes) maps closely to HomeKit's model (accessories → services → characteristics), making the transition natural for HomeKit developers.
- Matter's multi-admin capability requires new user-facing controls for transparency; the updated Home app surfaces this under "Connected Services."

---
_Source: WWDC21 Session 10298 page (abstract, chapter summaries, code samples, and resource links)._
