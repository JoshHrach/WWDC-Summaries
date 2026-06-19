# Accelerate Networking with HTTP/3 and QUIC
**WWDC21 · Session 10094** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10094/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, tvOS 15, watchOS 8

## Overview
HTTP/3 and QUIC are the next generation of networking protocols, standardized by the IETF, and available by default in iOS 15 and macOS Monterey. This session explains the evolution from HTTP/1 through HTTP/3, focusing on how QUIC's multiplexed streams, built-in TLS 1.3 security, and improved packet-loss recovery eliminate the head-of-line blocking problems of earlier HTTP versions.

URLSession apps automatically benefit from HTTP/3 without code changes once server-side support is enabled. The session also covers the new QUIC transport APIs in Network.framework — `NWProtocolQUIC` and `NWMultiplexGroup` — which allow custom application protocols to build on QUIC's advantages beyond HTTP.

For debugging, a new HTTP Traffic instrument in Instruments and support for qlog logging files allow developers to observe and verify HTTP/3 and QUIC usage in detail.

## Key Topics

### Evolution of HTTP
- HTTP/1: one request per connection (or head-of-line blocking on reused connections)
- HTTP/2: multiplexed streams on a single TCP connection — but packet loss affects all streams
- HTTP/3: independent streams, no shared TCP bottleneck; faster connection setup via QUIC

### QUIC Transport Protocol
- Replaces TCP as the underlying transport; standardized by IETF
- Built-in TLS 1.3 security — handshake in a single round trip
- Multiplexed streams without head-of-line blocking
- Improved loss recovery and connection migration (e.g., cellular to Wi-Fi)

### Using HTTP/3 with URLSession
- Enabled by default in iOS 15 and macOS Monterey — no client-side code changes needed
- HTTP/3 service discovery via:
  - DNS HTTPS resource record with `h3` ALPN string (first-connection capable)
  - `Alt-Svc` HTTP response header
- `assumesHTTP3Capable` property on `URLSessionConfiguration` to skip discovery for known-HTTP/3 servers
- HTTP priority via `priority` property and `prefersIncrementalDelivery`; can be changed dynamically

### Inspecting HTTP Traffic in Instruments
- New HTTP Traffic instrument in Xcode 13's Network profiling template
- No setup needed — taps URLSession directly
- Shows per-domain HTTP transactions, HTTP version used, response headers (including `Alt-Svc`)

### QUIC Transport API in Network.framework
- `NWProtocolQUIC` **[NEW]** — QUIC protocol for use with Network.framework
- `NWMultiplexGroup` **[NEW]** — connection group type for QUIC-multiplexed streams (tunnels)
- `NWConnectionGroup` extended to support multiplexing
- Unidirectional and bidirectional streams; streams created by either endpoint
- qlog support via environment variable for rich connection analysis

## APIs & Frameworks

### Network.framework
- `NWProtocolQUIC` **[NEW]** — QUIC protocol definition
- `NWProtocolQUIC.Options` **[NEW]** — configure QUIC transport parameters and per-stream options
- `NWProtocolQUIC.Metadata` **[NEW]** — per-stream metadata; `streamIdentifier`, `applicationError`
- `NWMultiplexGroup` **[NEW]** — descriptor for QUIC multiplex tunnel group
- `NWConnectionGroup` — extended with `newConnectionGroupHandler` for incoming tunnels **[NEW]**; `newConnectionHandler` for incoming streams **[NEW]**
- `NWConnection(from:)` — create outgoing stream from a connection group **[NEW]**
- `NWConnection.metadata(definition:)` — retrieve protocol-specific metadata
- `NWParameters.quic(alpn:)` **[NEW]** — convenience factory for QUIC parameters
- `NWListener.newConnectionGroupHandler` **[NEW]** — receive incoming QUIC tunnels
- `NWConnection.send(content:contentContext:isComplete:completion:)` — send data
- `NWConnection.receive(minimumIncompleteLength:maximumLength:completion:)` — receive data
- `NWConnection.stateUpdateHandler` — lifecycle callbacks
- `NWConnection.start(queue:)`, `NWConnectionGroup.start(queue:)`, `NWConnectionGroup.cancel()`

### URLSession
- `URLSessionConfiguration.assumesHTTP3Capable` **[NEW]** — skip HTTP/3 discovery
- `URLSessionTask.priority` — maps to QUIC/HTTP urgency (0.0–1.0)
- `URLSessionDataTask.prefersIncrementalDelivery` — controls HTTP incremental priority header

## Code Highlights

Creating a single QUIC connection:
```swift
let connection = NWConnection(host: "example.com", port: 443, using: .quic(alpn: ["myproto"]))
connection.stateUpdateHandler = { newState in
    switch newState {
    case .ready: print("Connected using QUIC!")
    default: break
    }
}
connection.start(queue: queue)
```

Establishing a QUIC tunnel with `NWMultiplexGroup`:
```swift
let descriptor = NWMultiplexGroup(to: .hostPort(host: "example.com", port: 443))
let group = NWConnectionGroup(with: descriptor, using: .quic(alpn: ["myproto"]))
group.stateUpdateHandler = { newState in ... }
group.newConnectionHandler = { newConnection in
    newConnection.stateUpdateHandler = { newState in ... }
    newConnection.start(queue: queue)
}
group.start(queue: queue)
```

Accessing QUIC stream metadata:
```swift
if let metadata = connection.metadata(definition: NWProtocolQUIC.definition) as? NWProtocolQUIC.Metadata {
    print("QUIC Stream ID is \(metadata.streamIdentifier)")
    metadata.applicationError = 0x100
    connection.cancel()
}
```

## Takeaways
- HTTP/3 is enabled by default in iOS 15 and macOS Monterey for all URLSession apps — enable it on your server to immediately benefit from faster connection setup and resilience to packet loss.
- QUIC's multiplexed streams eliminate head-of-line blocking and support connection migration, making it significantly more reliable than TCP on real-world networks.
- The new `NWProtocolQUIC` and `NWMultiplexGroup` APIs let custom application protocols build directly on QUIC's advantages in Network.framework.
- Use the new HTTP Traffic instrument in Xcode 13 and qlog environment-variable logging to verify and debug HTTP/3 and QUIC adoption.

---
_Source: WWDC21 Session 10094 page (abstract, chapter summaries, code samples, and resource links)._
