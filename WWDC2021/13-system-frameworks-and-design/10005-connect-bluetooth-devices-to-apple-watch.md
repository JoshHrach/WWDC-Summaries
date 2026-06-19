# Connect Bluetooth Devices to Apple Watch
**WWDC21 · Session 10005** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10005/)

_Platforms:_ watchOS 8

## Overview
watchOS 8 introduces the ability to connect to Bluetooth Low Energy accessories during Background App Refresh, enabling Apple Watch complications to display up-to-date data from accessories like health monitors, sports sensors, or environmental sensors — without requiring the user to open the app.

Previously, BLE connections on watchOS were only possible while the app was in the foreground or during a background session. Now, with a single `Info.plist` key and updated background task handling, apps can connect to known peripherals, retrieve fresh data, and update complications during scheduled Background App Refresh windows.

## Key Topics

### Background Bluetooth Connection (New in watchOS 8)
Add `bluetooth-central` to `UIBackgroundModes` in `Info.plist` to enable Bluetooth during Background App Refresh. During a `WKApplicationRefreshBackgroundTask`, the app can call `CBCentralManager.connect(_:options:)` on a previously discovered peripheral. When the Apple Watch receives the peripheral's advertisement, the BLE connection is established.

### Connection Lifecycle
1. **Foreground (first setup)**: Scan for new peripherals with `scanForPeripherals(withServices:options:)`. Discover, pair if needed, and store the peripheral reference.
2. **Background App Refresh**: Initiate connection to known peripheral (no scanning). Retrieve data. Disconnect explicitly via `cancelPeripheralConnection(_:)`.
3. **Expiration handler**: Called when the background task is about to expire. Cancel any pending connection and call `setTaskCompletedWithSnapshot(false)`.
4. **Auto-disconnect**: If the app does not disconnect before Background App Refresh ends, Core Bluetooth automatically terminates the connection. The next refresh will receive a `didDisconnectPeripheral` callback.

### Accessory Design Best Practices
- Accessories should advertise at least every 2 seconds in ideal RF conditions; more frequently in challenging conditions.
- Buffer sensor data and increase advertisement rate when nearing buffer capacity.
- Background App Refresh is not guaranteed to occur at specific times — frequent advertising maximizes the chance of a successful connection window.

### Best Practices for watchOS Apps
- Always disconnect explicitly when done to preserve battery.
- Only scan for new peripherals in the foreground.
- Use `expirationHandler` to cancel connections and complete tasks cleanly.
- First-time pairing and device discovery must happen in the foreground.

## APIs & Frameworks

**Core Bluetooth** (`import CoreBluetooth`)
- `CBCentralManager` — manages BLE central role
  - `scanForPeripherals(withServices:options:)` — discover new peripherals (foreground only)
  - `connect(_:options:)` — initiate connection to a known peripheral
  - `cancelPeripheralConnection(_:)` — disconnect from peripheral
- `CBCentralManagerDelegate` — delegate for central manager events
  - `centralManager(_:didDiscover:advertisementData:rssi:)` — peripheral discovered during scan
  - `centralManager(_:didConnect:)` — peripheral connected
  - `centralManager(_:didDisconnectPeripheral:error:)` — peripheral disconnected
- `CBPeripheral` — represents a remote BLE device
- `UIBackgroundModes` `Info.plist` key — add `bluetooth-central` to enable background BLE

**WatchKit** (`import WatchKit`)
- `WKApplicationRefreshBackgroundTask` — background refresh task
  - `expirationHandler` **[NEW watchOS 8]** — closure called just before Background App Refresh terminates
  - `setTaskCompletedWithSnapshot(_:)` — marks the task as complete
- `WKExtensionDelegate.handle(_:)` — entry point for background tasks
- `WKRefreshBackgroundTask` — base type for all background tasks

## Code Highlights

Handling Background App Refresh with Bluetooth:
```swift
func handle(_ backgroundTasks: Set<WKRefreshBackgroundTask>) {
    for task in backgroundTasks {
        if let refreshTask = task as? WKApplicationRefreshBackgroundTask {
            // Connect to previously discovered peripheral
            central.connect(peripheral, options: nil)
            refreshTask.expirationHandler = {
                // Cancel connection before expiry
                if let p = self.bluetoothReceiver.connectedPeripheral {
                    self.central.cancelPeripheralConnection(p)
                }
                refreshTask.setTaskCompletedWithSnapshot(false)
            }
        }
    }
}
```

Completing the task on disconnection:
```swift
func centralManager(_ central: CBCentralManager,
                    didDisconnectPeripheral peripheral: CBPeripheral,
                    error: Error?) {
    connectedPeripheral = nil
    delegate?.didCompleteDisconnection(from: peripheral)
}

// In WatchKit Extension delegate:
func didCompleteDisconnection(from peripheral: CBPeripheral) {
    if let refreshTask = currentRefreshTask {
        refreshTask.setTaskCompletedWithSnapshot(false)
        currentRefreshTask = nil
    }
}
```

## Takeaways
- watchOS 8 enables BLE connections during Background App Refresh by adding `bluetooth-central` to `UIBackgroundModes` and using the new `expirationHandler` on `WKApplicationRefreshBackgroundTask`.
- Accessories must advertise frequently (at least every 2 seconds) to maximize the chance of being discovered during short background windows.
- Apps should always explicitly disconnect via `cancelPeripheralConnection(_:)` when done rather than relying on automatic disconnect at task expiry.
- First-time peripheral discovery and pairing still require the app to be in the foreground; background connects only work with previously discovered peripherals.

---
_Source: WWDC21 Session 10005 page (abstract, chapter summaries, code samples, and resource links)._
