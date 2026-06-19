# Get timely alerts from Bluetooth devices on watchOS
**WWDC22 · Session 10135** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10135/)

_Platforms:_ watchOS 9, Apple Watch Series 6 or later

## Overview
watchOS 9 expands Core Bluetooth background capabilities on Apple Watch in two key ways: background GATT characteristic monitoring (timely alerts) and background peripheral scanning. Both features require Apple Watch Series 6 or later and build on the Background App Refresh Bluetooth foundation introduced in watchOS 8.

The session walks through the full pattern for each capability with live code, discusses the rate limits that protect Apple Watch battery life, and closes with hardware accessory design guidance covering advertisement intervals, connection intervals, and the Bluetooth 5.3 connection sub-rating feature coming to Apple Watch.

## Key Topics

**Background GATT characteristic monitoring (watchOS 9 new)** — While an app is backgrounded, Core Bluetooth keeps the peripheral connection alive and continues to receive GATT notification/indication events. When a monitored characteristic value changes, the app is given a short burst of background runtime to post a local notification, make a network request, or perform other work. Example use case: a food thermometer app that alerts the user when the meat temperature approaches the target.

**Background limits** — Two notifications track usage: `CBCentralManagerDelegate` error codes `LeGattNearBackgroundNotificationLimit` and `LeGattExceededBackgroundNotificationLimit`. After the limit is exceeded the app reverts to watchOS 8 behavior (no background connection; Background App Refresh only). In watchOS 9 the limit is 5 runtime events. Limits reset on user interaction with the app or 24 hours after the limit is reached.

**Background peripheral discovery (watchOS 9 new)** — Call `scanForPeripherals(withServices:)` while in the foreground and Core Bluetooth continues scanning in the background. When a matching advertisement is detected, the app gets background runtime to connect. Useful for devices that only advertise when a critical event occurs (e.g., a medical sensor that advertises only on alarm). The background scan runtime budget is shared with characteristic-change runtime. `allowDuplicatesKey` scanning is foreground-only.

**Bluetooth reconnection** — If a peripheral goes out of range, the app briefly gets background runtime to call `connectPeripheral(_:options:)`. Core Bluetooth reconnects automatically when the device returns to range. Repeated out-of-range events within a 24-hour window reduce the reconnection range to conserve battery.

**Accessory design guidance**
- Two topology options: (1) deep-sleep peripheral that advertises only on alert (lower power, higher latency) and (2) always-connected peripheral using GATT indications (lower latency, higher power).
- Recommend long connection intervals (≥ 150 ms) for always-connected peripherals.
- Recommend transmitting only event-driven data, not continuous sensor streams, to stay within runtime limits.
- Bluetooth 5.3 connection sub-rating (coming to Apple Watch) will allow fast switching between idle and active connection intervals.

## APIs & Frameworks

### Core Bluetooth
- `CBPeripheral.setNotifyValue(_:for:)` — subscribe to GATT characteristic notifications/indications **[now honored in background on watchOS 9]**
- `CBPeripheralDelegate.peripheral(_:didDiscoverCharacteristicsFor:error:)` — called after characteristic discovery; subscribe here
- `CBPeripheralDelegate.peripheral(_:didUpdateValueFor:error:)` — called when characteristic value changes (foreground and background) **[new background delivery on watchOS 9]**
- `CBCentralManager.scanForPeripherals(withServices:options:)` — scan for peripherals by service UUID **[background scan continues on watchOS 9]**
- `CBCentralManager.connectPeripheral(_:options:)` — connect to a discovered peripheral; also used for background reconnection
- `CBCentralManagerDelegate.centralManager(_:didConnect:)` — connection established callback
- `LeGattNearBackgroundNotificationLimit` **[NEW]** — error code posted to `didUpdateValueFor` when approaching the background runtime limit
- `LeGattExceededBackgroundNotificationLimit` **[NEW]** — error code posted when the limit is exceeded; app reverts to Background App Refresh only
- `allowDuplicatesKey` — duplicate advertisement scanning option; foreground-only in watchOS 9

### Info.plist
- `UIBackgroundModes` → `bluetooth-central` — required in the Watch app's Info.plist to enable background BLE connections ("App communicates using CoreBluetooth")
- Must be set directly in the Watch extension's Info.plist; cannot rely on iOS Signing & Capabilities

### UserNotifications
- `UNUserNotificationCenter` — post local notifications from the `didUpdateValueFor` background callback to alert the user

### watchOS Background Execution
- Background App Refresh — periodic background execution for complication updates (introduced watchOS 8)
- Background BLE connection / timely alerts — event-driven background runtime on GATT characteristic change **[NEW in watchOS 9]**
- Background peripheral scan — scan continues after app backgrounds **[NEW in watchOS 9]**
- Both new modes require Apple Watch Series 6 or later

## Code Highlights

Subscribing to characteristic notifications (triggers background delivery in watchOS 9):
```swift
func peripheral(_ peripheral: CBPeripheral,
                didDiscoverCharacteristicsFor service: CBService,
                error: Error?) {
    peripheral.setNotifyValue(true, for: characteristic)
}

func peripheral(_ peripheral: CBPeripheral,
                didUpdateValueFor characteristic: CBCharacteristic,
                error: Error?) {
    if let newData = characteristic.value {
        // Post a local notification or send a network request.
        // Called in both foreground and background — handle each case appropriately.
    }
}
```

Scanning for peripherals in the foreground (continues in background):
```swift
func centralManager(_ central: CBCentralManager, didConnect peripheral: CBPeripheral) {
    central.scanForPeripherals(withServices: [myCustomUUID])
}
```

## Takeaways
- watchOS 9 adds two new background BLE modes for Apple Watch Series 6+: characteristic monitoring for timely alerts and peripheral scanning; both require `bluetooth-central` in the Watch app's Info.plist.
- Background runtime is limited (5 events in watchOS 9 per 24-hour window); monitor `LeGattNearBackgroundNotificationLimit` and `LeGattExceededBackgroundNotificationLimit` errors and fall back gracefully.
- Design accessories to transmit only event-driven data rather than continuous sensor streams; this keeps both peripheral and Apple Watch battery consumption low.
- Use long Bluetooth connection intervals (≥ 150 ms) for always-connected accessories to preserve Apple Watch battery life.

---
_Source: WWDC22 Session 10135 page (abstract, transcript, code samples, and resource links)._
