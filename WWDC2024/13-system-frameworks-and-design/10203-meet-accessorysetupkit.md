# Meet AccessorySetupKit
**WWDC24 · Session 10203** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10203/)

_Platforms:_ iOS 18, iPadOS 18

## Overview
AccessorySetupKit is a new framework in iOS 18 that gives apps a streamlined, privacy-preserving way to discover and pair Bluetooth and Wi-Fi accessories. Instead of sending users to the Settings app, AccessorySetupKit presents an in-app picker sheet with a custom accessory image, name, and description — matching the accessory's physical appearance so users can confidently identify what they're pairing.

The framework improves privacy by granting the app access only to accessories it explicitly declares, rather than giving blanket access to all nearby Bluetooth devices. Apps can also migrate accessories previously paired via Core Bluetooth into AccessorySetupKit management.

## Key Topics

**The Accessory Picker**
- Replaces the flow of: prompt user → open Settings → find accessory → grant Bluetooth access
- Presents a sheet with a custom image (from the app's asset catalog), name, and product information for each discoverable accessory
- Users select which specific accessory to pair; the app only receives access to the chosen device — not all nearby Bluetooth peripherals
- The picker is invoked programmatically; no separate permission prompt is shown because the framework scopes access inherently

**Session and Event Model**
- `ASAccessorySession` — the central object; must be activated before use
- `session.activate(on:eventHandler:)` — activates the session on a dispatch queue and registers a callback for all events
- Events delivered via `ASAccessoryEvent`; `event.eventType` distinguishes `accessoryAdded`, `accessoryChanged`, `accessoryRemoved`, and `activated`
- `event.accessory` — returns the `ASAccessory` involved in the event

**Discovery Descriptors and Display Items**
- `ASDiscoveryDescriptor` — describes what to scan for (e.g., a specific Bluetooth service UUID via `bluetoothServiceUUID`)
- `ASPickerDisplayItem` — pairs a `ASDiscoveryDescriptor` with a display name and `UIImage`; one display item per accessory variant shown in the picker
- Multiple display items can be passed to a single `showPicker(for:)` call (e.g., pink and blue variants of the same product)

**Showing the Picker**
- `session.showPicker(for: [displayItems]) { error in ... }` — presents the accessory picker; completion handler called with an error if the user cancels or pairing fails
- After the user selects an accessory, the `.accessoryAdded` event fires in the session's event handler
- Communication with the paired accessory then proceeds via Core Bluetooth's `CBCentralManager`, which now receives the `.poweredOn` state only when a paired accessory from AccessorySetupKit is available

**Migrating Existing Accessories**
- Apps with existing Core Bluetooth accessories can migrate them into AccessorySetupKit without requiring users to re-pair
- `ASMigrationDisplayItem` — like `ASPickerDisplayItem` but also holds a reference to an existing `CBPeripheral` (via `.peripheral`) so the framework can match it to the system record
- Pass migration items to `showPicker(for:)` alongside regular display items; already-paired accessories appear as pre-selected in the picker UI

## APIs & Frameworks

**AccessorySetupKit** **[NEW]**
- `ASAccessorySession` **[NEW]** — manages the accessory pairing lifecycle
- `ASAccessorySession.activate(on:eventHandler:)` **[NEW]** — activates the session; takes a `DispatchQueue` and event-handler closure
- `ASAccessoryEvent` **[NEW]** — event object passed to the event handler
- `ASAccessoryEvent.eventType` **[NEW]** — `ASAccessoryEventType`: `.activated`, `.accessoryAdded`, `.accessoryChanged`, `.accessoryRemoved`
- `ASAccessoryEvent.accessory` **[NEW]** — the `ASAccessory` involved in the event
- `ASAccessory` **[NEW]** — represents a discovered or paired accessory
- `ASDiscoveryDescriptor` **[NEW]** — describes a Bluetooth (or Wi-Fi) accessory to scan for
- `ASDiscoveryDescriptor.bluetoothServiceUUID` **[NEW]** — `UUID` of the Bluetooth GATT service to match
- `ASPickerDisplayItem` **[NEW]** — pairs a descriptor with a display name and image for the picker UI
- `ASPickerDisplayItem(name:productImage:descriptor:)` **[NEW]** — initializer
- `ASAccessorySession.showPicker(for:completionHandler:)` **[NEW]** — presents the accessory picker sheet
- `ASMigrationDisplayItem` **[NEW]** — display item for migrating an existing Core Bluetooth peripheral
- `ASMigrationDisplayItem.peripheral` **[NEW]** — `CBPeripheral` to migrate into AccessorySetupKit management

**Core Bluetooth**
- `CBCentralManager` — used after pairing to communicate with the accessory; state transitions to `.poweredOn` only when AccessorySetupKit-managed accessories are available
- `CBCentralManager(delegate:queue:)` — standard initializer
- `CBCentralManagerDelegate.centralManagerDidUpdateState(_:)` — handle Bluetooth state changes
- `CBCentralManagerDelegate.centralManager(_:didConnect:)` — handle successful peripheral connection

## Code Highlights

Activate a session and handle events:
```swift
var session = ASAccessorySession()
session.activate(on: .main, eventHandler: handleSessionEvent(event:))

func handleSessionEvent(event: ASAccessoryEvent) {
    switch event.eventType {
    case .activated:
        print("Session is activated and ready to use")
    case .accessoryAdded:
        let newAccessory = event.accessory
        // store and connect to accessory
    default:
        print("Received event type \(event.eventType)")
    }
}
```

Create display items and show the picker:
```swift
let pinkDescriptor = ASDiscoveryDescriptor()
pinkDescriptor.bluetoothServiceUUID = pinkUUID

let pinkItem = ASPickerDisplayItem(
    name: "Pink Dice",
    productImage: UIImage(named: "pink")!,
    descriptor: pinkDescriptor)

session.showPicker(for: [pinkItem, blueItem]) { error in
    if let error { /* handle cancellation or failure */ }
}
```

Migrate an existing Core Bluetooth peripheral:
```swift
let pinkMigration = ASMigrationDisplayItem(
    name: "Pink Dice",
    productImage: UIImage(named: "pink")!,
    descriptor: pinkDescriptor)
pinkMigration.peripheral = existingPeripheral
session.showPicker(for: [pinkMigration]) { error in ... }
```

## Takeaways
- Adopt AccessorySetupKit to replace ad-hoc Bluetooth permission flows — users get a branded picker with a photo of the accessory instead of a generic permission alert.
- Declare one `ASPickerDisplayItem` per accessory variant (e.g., colors/sizes) so the picker shows exactly what's available.
- Use `ASMigrationDisplayItem` to silently upgrade existing Core Bluetooth pairings without forcing users to re-pair.
- After pairing, continue using Core Bluetooth (`CBCentralManager`) for data communication — AccessorySetupKit only handles discovery and authorization.

---
_Source: WWDC24 Session 10203 page (abstract, chapter summaries, code samples, and resource links)._
