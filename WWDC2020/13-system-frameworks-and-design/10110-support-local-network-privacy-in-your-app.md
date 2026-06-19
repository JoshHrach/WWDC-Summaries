# Support local network privacy in your app
**WWDC20 · Session 10110** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10110/)

_Platforms:_ iOS 14, iPadOS 14, tvOS 14

## Overview
iOS 14 introduces a new privacy permission governing apps that access the local network — directly communicating with nearby devices via Bonjour, unicast, multicast, or broadcast protocols. Any app that previously interacted with the local network without restriction now requires explicit user approval and two new `Info.plist` keys. Apps that interact with the local network only through system services (AirPrint, AirPlay, AirDrop, HomeKit) are exempt and need no changes.

The motivation is privacy: a device's home network contains a unique constellation of devices that can function as a precise location fingerprint — and any device reachable on the local network could be silently probed by a malicious app. Local network privacy closes this gap by making access opt-in, transparent, and revocable from Settings.

## Key Topics

### What Triggers the Permission
- **Requires permission** (apps must update): direct Bonjour browsing/advertising, `NWBrowser`/`NWListener`, unicast connections to local IPs, multicast group joins, raw socket sends to the local subnet
- **Exempt** (no changes needed): AirPrint, AirPlay, AirDrop, HomeKit — these system services discover devices without exposing the full device list to the app
- Apps compiled against iOS 13 SDK and run on iOS 14 show a **default reason text** prompt automatically when they first trigger local network access; update to the iOS 14 SDK to supply a custom reason

### Permission Flow
- The system blocks all local network communication until the user responds to the permission prompt — no partial access
- Bonjour browsing returns zero results (looks like no devices nearby) until permission is granted; no error is surfaced to the user
- `NWConnection` to a local host stays in the `.waiting` state until permission is granted
- `URLSession` tasks that require local network access stall until permission is granted — set `waitsForConnectivity = true` on the session configuration
- Direct BSD socket calls return `ENOAUTH` / `errno = 1` when blocked
- Bonjour `NetService` / `NWBrowser` returns a `NoAuth` error in the debug console when blocked — this is the diagnostic signal that the Info.plist update is missing

### Required `Info.plist` Updates (NEW)
Two new keys under the root level of `Info.plist`:

1. **`NSLocalNetworkUsageDescription`** (String) — human-readable description of why the app needs local network access; shown in the permission alert and in Settings → Privacy → Local Network
2. **`NSBonjourServices`** (Array of Strings) — list of Bonjour service types the app browses or advertises (e.g., `_tictactoe._tcp`, `_myapp._udp`); required if the app uses any Bonjour API

### Entitlement for Exceptional Cases
- Apps that perform multicast/broadcast discovery **without** Bonjour, or that enumerate **all** Bonjour service types on a network (not just their own), require a special entitlement
- Request `com.apple.developer.networking.multicast` via the Apple Developer portal

### Best Practices for Permission UX
- **Delay the prompt**: do not browse the local network at app launch; defer until the user takes an action that genuinely requires it (e.g., tapping "Search for games" rather than triggering on `viewDidAppear`)
- **Write a clear usage description**: describe the specific feature being enabled (e.g., "TicTacToe uses the local network to discover players around you"), not a generic phrase
- Handle the "no results" state gracefully — while awaiting permission or if permission is denied, Bonjour browsers return empty results; display appropriate empty-state UI
- Handle `NWConnection` `.waiting` state explicitly; retry or inform the user when connectivity is restored after permission grant

## APIs & Frameworks

**Network framework**
- `NWBrowser` — browse for Bonjour services; returns `.waiting` / empty results if local network access is blocked
- `NWListener` — advertise a Bonjour service
- `NWConnection` — connect to a local host; stays in `.waiting` state until permission granted
- `NWParameters` — configure connection parameters including Bonjour service types

**Foundation**
- `URLSessionConfiguration.waitsForConnectivity` — set `true` so `URLSession` tasks automatically retry after permission grant
- `NetService` (deprecated but still in use) / `NetServiceBrowser` — Bonjour browsing; returns `NoAuth` error when blocked

**BSD Sockets (low level)**
- Direct `socket()` / `sendto()` calls return `ENOAUTH` (errno) when local network access is blocked

**Info.plist Keys (NEW)**
- `NSLocalNetworkUsageDescription` **[NEW]** — local network usage description string
- `NSBonjourServices` **[NEW]** — array of Bonjour service type strings (e.g., `_tictactoe._tcp`)

**Entitlements**
- `com.apple.developer.networking.multicast` **[NEW]** — required for non-Bonjour multicast/broadcast, or for enumerating all Bonjour services on a network; request via the developer portal

**System Services (exempt — no changes needed)**
- AirPrint (`UIPrintInteractionController`)
- AirPlay (`AVRoutePickerView`, `MPVolumeView`)
- AirDrop (`UIActivityViewController`)
- HomeKit (`HMHomeManager`)

## Code Highlights

Required Info.plist additions for a Bonjour-based app:
```xml
<key>NSLocalNetworkUsageDescription</key>
<string>TicTacToe uses the local network to discover players around you.</string>

<key>NSBonjourServices</key>
<array>
    <string>_tictactoe._tcp</string>
</array>
```

Handle the waiting state in NWConnection:
```swift
let connection = NWConnection(host: localHost, port: .http, using: .tcp)
connection.stateUpdateHandler = { state in
    switch state {
    case .waiting(let error):
        // Local network permission not yet granted — show empty state UI
        print("Waiting for local network permission: \(error)")
    case .ready:
        // Permission granted, connection active
        startCommunication()
    default: break
    }
}
connection.start(queue: .main)
```

URLSession with waitsForConnectivity for local resource:
```swift
let config = URLSessionConfiguration.default
config.waitsForConnectivity = true  // task resumes automatically after permission grant
let session = URLSession(configuration: config)
let task = session.dataTask(with: localResourceURL) { data, response, error in
    // handle response
}
task.resume()
```

## Takeaways

- Every app using Bonjour, `NWBrowser`, `NWListener`, or raw multicast/broadcast on iOS 14 must add `NSLocalNetworkUsageDescription` and (for Bonjour) `NSBonjourServices` to `Info.plist` — omitting them causes all local network communication to be silently blocked.
- Trigger the permission prompt from a user-initiated action rather than at launch — this context makes users far more likely to grant access and avoids a confusing cold-start prompt.
- Apps using only AirPrint, AirPlay, AirDrop, or HomeKit are fully exempt; those system frameworks handle device discovery internally without exposing the network device list to the app.
- Handle the transitional states gracefully: Bonjour browsers silently return zero results, `NWConnection` enters `.waiting`, and `URLSession` tasks stall — all of these recover automatically once the user grants permission.

---
_Source: WWDC20 Session 10110 page (transcript and resource links)._
