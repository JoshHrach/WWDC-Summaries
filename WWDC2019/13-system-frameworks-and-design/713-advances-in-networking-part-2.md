# Advances in Networking, Part 2
**WWDC19 · Session 713** · [Watch](https://developer.apple.com/videos/play/wwdc2019/713/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15, watchOS 6, tvOS 13

## Overview
Part 2 of the networking advances series covers four advanced topics: wide-area Bonjour discovery via a new `NWBrowser` API and discovery proxies, custom protocol framing handlers in Network.framework (`NWProtocolFramerImplementation`), expanded metrics collection in both URLSession and Network.framework, and platform-level updates including TLS 1.3 defaults, Wi-Fi access restrictions, Catalyst networking behavior, watchOS direct networking, and deprecations (PAC file/FTP URL schemes, SPDY, Secure Transport TLS 1.3 support).

The session uses a peer-to-peer tic-tac-toe game as a running example, demonstrating NWBrowser for service discovery, NWParameters with CryptoKit-derived pre-shared keys for TLS, and a custom TLV framing protocol for game messages.

## Key Topics

### NWBrowser — Native Bonjour Browsing **[NEW]**
- `NWBrowser` **[NEW]** — completes the Network.framework trio: `NWListener` (advertise) + `NWBrowser` (discover) + `NWConnection` (connect). Previously, discovery required a separate Bonjour API followed by manual endpoint conversion.
- `NWBrowser.init(for:using:)` — browse for a Bonjour service type using NWParameters (same object used for connections/listeners).
- `NWBrowser.browseResultsChangedHandler` — delivers endpoint updates as either a full change list (added/removed/changed with flags) or the latest complete results list.
- Browse results can be passed directly to `NWConnection.init(to:using:)` — no address resolution step needed.
- **Discovery Proxy** **[NEW]**: enables Bonjour discovery across subnets; client sends unicast to proxy, proxy multicasts on the target subnet and proxies results back. Client-side code is unchanged (specify `nil` for domain); server implementation available on GitHub.
- Always specify `nil` for domain (not `"local."`) when browsing — specifying `"local."` blocks discovery of proxied/wide-area services.
- `NWParameters.includePeerToPeer = true` — enables discovery and connection over peer-to-peer links (Bluetooth, Wi-Fi Direct) when devices are not on the same network.

### Custom Protocol Framing **[NEW]**
- `NWProtocolFramerImplementation` **[NEW]** — protocol that custom classes conform to to define message framing logic running on the same thread as TCP/TLS in the user-space networking stack.
- Two required methods: `handleOutput` (encode outgoing application messages as wire bytes) and `handleInput` (parse incoming bytes into discrete messages).
- `NWProtocolFramer.Instance` — handle passed to the framing methods:
  - `writeOutput(data:)` — queue header bytes for sending.
  - `writeOutputNoCopy(length:)` — queue application bytes directly (zero-copy).
  - `parseInput(minimumIncompleteLength:maximumLength:parse:)` — inspect incoming bytes; returns `false` if insufficient bytes; closure returns bytes consumed.
  - `deliverInput(data:message:isComplete:)` / `deliverInputNoCopy(length:message:isComplete:)` — deliver parsed message (or a byte range) to the application layer.
- `NWProtocolFramer.Message` — key-value store attached to each send/receive; lets the framer return metadata (e.g., message type enum) to the application without changing the send/receive API.
- `NWProtocolFramer.Definition` — registered handle to the protocol class; used when building protocol stack options.
- `NWProtocolOptions` subclass created from a `Definition` is added to `NWParameters.defaultProtocolStack.applicationProtocols`.
- WebSocket (new in iOS 13) is implemented internally as a framing protocol using this same API.
- STARTTLS pattern: implement a framing protocol that performs the plaintext handshake, then dynamically inserts TLS into the stack above itself before calling `ready`.

### Network Metrics **[NEW / Expanded]**
**URLSession:**
- `URLSessionTaskTransactionMetrics` — new properties **[NEW]**:
  - `localAddress`, `localPort`, `remoteAddress`, `remotePort` — connection endpoints
  - `negotiatedTLSProtocolVersion`, `negotiatedTLSCipherSuite` — confirm TLS 1.3 usage
  - `isConstrained`, `isExpensive` — path property flags at time of request
  - `countOfRequestBodyBytesBeforeEncoding`, `countOfRequestBodyBytesSent`, `countOfResponseBodyBytesReceived`, `countOfResponseBodyBytesAfterDecoding` — granular byte counts for data-saving validation

**Network.framework:**
- `NWConnection.requestEstablishmentReport(queue:completion:)` **[NEW]** — available once connection is `.ready`; delivers `NWConnection.EstablishmentReport` with:
  - `duration` — total connection establishment time
  - `resolutions: [NWConnection.EstablishmentReport.Resolution]` — DNS/Bonjour resolution steps with `source` (`.dns`, `.optimisticDNS` / expired cache, `.bonjour`)
  - `connectionAttempts: [NWConnection.EstablishmentReport.ConnectionAttempt]` — per-attempt TCP, TLS timing, proxy info, RTT
- `NWConnection.startDataTransferReport() -> NWConnection.PendingDataTransferReport` **[NEW]** — begin a data transfer measurement window.
- `NWConnection.PendingDataTransferReport.collect(queue:completion:)` **[NEW]** — stop the window and deliver `NWConnection.DataTransferReport` with:
  - Per-path (or aggregate) packet/byte counts sent/received; RTT details
  - Multipath breakdown when using `.handover`/`.interactive` service type.
- **Optimistic DNS** — now on by default for URLSession and Network.framework connections. Connects to cached IP in parallel with fresh DNS query; `EstablishmentReport.Resolution.source == .dns` with `isExpiredCache == true` indicates optimistic DNS was used.
- **Network Link Conditioner** — now accessible directly in Xcode's Devices and Simulators panel (Device Conditions) for simulating realistic network conditions.

### TLS 1.3 **[NEW Default]**
- TLS 1.3 is now the default for URLSession and Network.framework connections.
- Benefits: one round-trip handshake (vs. two for TLS 1.2); removes weak algorithms (ECDSA, RSA key exchange without forward secrecy); encrypts certificates and most header fields; all algorithms provide AEAD and forward secrecy.
- Secure Transport will never support TLS 1.3 — another reason to migrate to URLSession or Network.framework.
- Ensure servers are updated to support TLS 1.3.

### Wi-Fi Information Access Restriction **[NEW]**
- Accessing Wi-Fi SSID/BSSID now requires the same location privileges used for other location data — can be used to infer location.
- Requirement: add the "Access Wi-Fi Information" capability (entitlement) in Xcode **AND** at least one of:
  - User has granted location access to the app.
  - App is the currently active VPN app.
  - App is a Hotspot Configuration app (for networks it configured only).

### watchOS Direct Networking **[NEW]**
- Apps using AVFoundation audio streaming can now use direct networking via URLSession or Network.framework on watchOS.
- Raw sockets remain unavailable on watchOS.

### Catalyst Networking
- By default, outgoing network connections are allowed for Catalyst apps.
- Incoming connections require explicitly checking the "Incoming Connections (Server)" checkbox in the Xcode entitlements for Mac.

### Deprecations
- PAC files using `file://` or `ftp://` URL schemes — no longer supported.
- SPDY — replaced by HTTP/2; Apple only supports HTTP/2.
- Secure Transport — does not and will not support TLS 1.3; migrate to URLSession or Network.framework.

## APIs & Frameworks

**Network.framework**
- `NWBrowser` **[NEW]** — `init(for:using:)`, `browseResultsChangedHandler`, `stateUpdateHandler`, `start(queue:)`, `cancel()`
- `NWBrowser.Result` **[NEW]** — `endpoint`, `metadata`
- `NWBrowser.Result.Change` **[NEW]** — `.added(_:)`, `.removed(_:)`, `.changed(old:new:flags:)`
- `NWBrowser.Result.Change.Flags` **[NEW]** — `.interfaceAdded`, `.interfaceRemoved`, `.metadataChanged`
- `NWParameters.includePeerToPeer: Bool` — existing property, critical for peer-to-peer discovery
- `NWProtocolFramerImplementation` **[NEW]** — protocol: `start(framer:)`, `handleOutput(framer:message:messageLength:isComplete:)`, `handleInput(framer:)`, `wakeup(framer:)`, `stop(framer:)`, `cleanup(framer:)`, `static var definition: NWProtocolFramer.Definition`
- `NWProtocolFramer.Instance` **[NEW]** — `writeOutput(data:)`, `writeOutputNoCopy(length:)`, `parseInput(minimumIncompleteLength:maximumLength:parse:)`, `deliverInput(data:message:isComplete:)`, `deliverInputNoCopy(length:message:isComplete:)`, `passThroughOutput()`, `passThroughInput()`, `markReady()`, `markFailed(error:)`, `scheduleWakeup(wakeupTime:)`, `async(execute:)`
- `NWProtocolFramer.Definition` **[NEW]** — `init(implementation:)`
- `NWProtocolFramer.Message` **[NEW]** — `init(definition:)`, subscript for custom key-value metadata
- `NWConnection.requestEstablishmentReport(queue:completion:)` **[NEW]**
- `NWConnection.EstablishmentReport` **[NEW]** — `duration`, `resolutions`, `connectionAttempts`
- `NWConnection.startDataTransferReport() -> NWConnection.PendingDataTransferReport` **[NEW]**
- `NWConnection.PendingDataTransferReport.collect(queue:completion:)` **[NEW]**
- `NWConnection.DataTransferReport` **[NEW]** — `paths`, `sentPacketCount`, `sentByteCount`, `receivedPacketCount`, `receivedByteCount`, `roundTripTime`

**Foundation — URLSession**
- `URLSessionTaskTransactionMetrics.localAddress/localPort/remoteAddress/remotePort` **[NEW]**
- `URLSessionTaskTransactionMetrics.negotiatedTLSProtocolVersion` **[NEW]** — `tls_protocol_version_t`
- `URLSessionTaskTransactionMetrics.negotiatedTLSCipherSuite` **[NEW]**
- `URLSessionTaskTransactionMetrics.isConstrained: Bool` **[NEW]**
- `URLSessionTaskTransactionMetrics.isExpensive: Bool` **[NEW]**
- `URLSessionTaskTransactionMetrics.countOfRequestBodyBytesBeforeEncoding` **[NEW]**
- `URLSessionTaskTransactionMetrics.countOfRequestBodyBytesSent` **[NEW]**
- `URLSessionTaskTransactionMetrics.countOfResponseBodyBytesReceived` **[NEW]**
- `URLSessionTaskTransactionMetrics.countOfResponseBodyBytesAfterDecoding` **[NEW]**
- `URLSessionConfiguration.waitsForConnectivity = true` — existing; combined with `allowsExpensiveNetworkAccess = false` lets system wait for non-cellular path; `urlSession(_:taskIsWaitingForConnectivity:)` delegate for UI prompt

## Code Highlights

```swift
// NWBrowser: discover peer-to-peer Bonjour services
let params = NWParameters()
params.includePeerToPeer = true  // discover over Bluetooth/Wi-Fi Direct too

let browser = NWBrowser(for: .bonjour(type: "_tictactoe._tcp", domain: nil), using: params)
browser.browseResultsChangedHandler = { results, changes in
    // results: Set<NWBrowser.Result> — pass to UI for display
    delegate?.updateResults(results)
}
browser.start(queue: .main)

// Connect directly from a browse result endpoint:
let connection = NWConnection(to: selectedResult.endpoint, using: secureParams)
```

```swift
// Custom TLV framing protocol
class GameProtocol: NWProtocolFramerImplementation {
    static let definition = NWProtocolFramer.Definition(implementation: GameProtocol.self)

    func start(framer: NWProtocolFramer.Instance) -> NWProtocolFramer.StartResult { .ready }

    func handleOutput(framer: NWProtocolFramer.Instance,
                      message: NWProtocolFramer.Message, messageLength: Int, isComplete: Bool) {
        var header = GameHeader(type: message.gameMessageType, length: UInt32(messageLength))
        let headerData = Data(bytes: &header, count: 8)
        framer.writeOutput(data: headerData)           // write 8-byte header
        framer.writeOutputNoCopy(length: messageLength) // zero-copy body
    }

    func handleInput(framer: NWProtocolFramer.Instance) -> Int {
        // Parse 8-byte header
        let parsed = framer.parseInput(minimumIncompleteLength: 8, maximumLength: 8) { buffer, _ in
            guard buffer.count >= 8 else { return 0 }
            let header = buffer.withUnsafeBytes { $0.load(as: GameHeader.self) }
            // Save header values, deliver body
            let msg = NWProtocolFramer.Message(definition: GameProtocol.definition)
            msg.gameMessageType = GameMessageType(rawValue: header.type)!
            _ = framer.deliverInputNoCopy(length: Int(header.length), message: msg, isComplete: true)
            return 8  // consumed 8 header bytes
        }
        return parsed ? 0 : 8  // if failed, wait for 8 bytes
    }

    static var label = "GameProtocol"
    func wakeup(framer: NWProtocolFramer.Instance) {}
    func stop(framer: NWProtocolFramer.Instance) -> Bool { true }
    func cleanup(framer: NWProtocolFramer.Instance) {}
}

// Add custom framing protocol to connection parameters
let options = NWProtocolFramer.Options(definition: GameProtocol.definition)
params.defaultProtocolStack.applicationProtocols.insert(options, at: 0)

// Send a typed message
let msg = NWProtocolFramer.Message(definition: GameProtocol.definition)
msg.gameMessageType = .selectCharacter
let context = NWConnection.ContentContext(identifier: "select", metadata: [msg])
connection.send(content: characterData, contentContext: context, isComplete: true, completion: .idempotent)

// Receive a typed message
connection.receiveMessage { data, context, isComplete, error in
    let msg = context?.protocolMetadata(definition: GameProtocol.definition) as? NWProtocolFramer.Message
    let type = msg?.gameMessageType
}
```

```swift
// Network.framework: collect establishment metrics
connection.requestEstablishmentReport(queue: .main) { report in
    guard let report else { return }
    print("Total time: \(report.duration)")
    for resolution in report.resolutions {
        print("DNS source: \(resolution.source)")  // .dns, expired cache = optimistic DNS
    }
    for attempt in report.connectionAttempts {
        print("TLS handshake: \(attempt.tlsHandshake)")
    }
}

// Collect data transfer metrics for a burst
let pending = connection.startDataTransferReport()
// ... send/receive data ...
pending.collect(queue: .main) { report in
    let path = report.paths.first
    print("Sent \(path?.sentByteCount ?? 0) bytes, RTT \(path?.roundTripTime ?? 0)")
}
```

## Takeaways
- `NWBrowser` completes the Network.framework service workflow; pass browse results directly to `NWConnection` without any manual Bonjour-to-endpoint conversion.
- Always specify `nil` (not `"local."`) for the domain when browsing — `"local."` silently blocks wide-area and proxied discovery.
- `NWProtocolFramerImplementation` enables custom message framing running inside the networking stack thread — applications call `receiveMessage` and get exactly one callback per complete message, eliminating all manual buffer management.
- Optimistic DNS is on by default; verify it is benefiting your app with `EstablishmentReport.Resolution.source` and use network link conditioner (now in Xcode's Devices panel) for realistic performance testing.
- TLS 1.3 is now the default; verify with `URLSessionTaskTransactionMetrics.negotiatedTLSProtocolVersion` and update servers accordingly. Do not use Secure Transport for new code.

---
_Source: WWDC19 Session 713 page (transcript, abstract, and resource links)._
