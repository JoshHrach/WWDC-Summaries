# Use Structured Concurrency with Network Framework
**WWDC25 · Session 250** · [Watch](https://developer.apple.com/videos/play/wwdc2025/250/)

_Platforms:_ iOS 26, iPadOS 26, macOS Tahoe 26, tvOS 26, visionOS 26, watchOS 26

## Overview
Network framework gains a new set of Swift-native, structured-concurrency-based APIs in iOS 26 and macOS Tahoe 26: `NetworkConnection`, `NetworkListener`, and `NetworkBrowser`. These replace the callback-heavy `NWConnection`, `NWListener`, and `NWBrowser` APIs for new Swift code, enabling clean `async`/`await` data I/O and `AsyncSequence`-based event streams. The session also introduces a TLV framer (length-value message framing) and a `Coder` protocol for type-safe encode/decode, and demonstrates Wi-Fi Aware integration for peer-to-peer networking.

## Key Topics

### NetworkConnection
`NetworkConnection` is the new structured-concurrency replacement for `NWConnection`. It provides `async` methods for sending and receiving data, and an `AsyncSequence`-based stream of incoming messages. Connections can be established to TCP, UDP, TLS, QUIC, or Wi-Fi Aware endpoints. The connection manages its own state and lifecycle within Swift's structured concurrency task hierarchy.

### NetworkListener
`NetworkListener` replaces `NWListener` with an `AsyncSequence` of incoming connections. Iterating over the sequence yields `NetworkConnection` objects as clients connect. The listener automatically handles TLS negotiation when a TLS identity is provided.

### NetworkBrowser
`NetworkBrowser` replaces `NWBrowser` for Bonjour/DNS-SD service discovery. It provides an `AsyncSequence` of discovered services. Discovered endpoints can be directly connected via `NetworkConnection`.

### TLV Framer
A new `TLVFramer` provides built-in type-length-value message framing over a byte stream connection. It segments a raw TCP/UDP stream into discrete typed messages without requiring custom framing logic. Individual TLV frames carry a type tag and length-prefixed payload.

### Coder Protocol
The new `Coder` protocol allows apps to define type-safe serialization/deserialization for messages sent over `NetworkConnection`. Conforming types implement `encode(to:)` and `decode(from:)` methods. The Coder integrates with `NetworkConnection` so apps can send and receive strongly-typed values directly without manual Data marshaling.

### Wi-Fi Aware Integration
`NetworkConnection` and `NetworkListener` can use Wi-Fi Aware endpoints (from the Wi-Fi Aware framework) as their underlying transport, giving the same structured-concurrency data path to peer-to-peer Wi-Fi connections discovered via `WASubscribableService` or `WAPairedDevice`.

## APIs & Frameworks

**Network framework** (iOS/macOS 26)
- `NetworkConnection` **[NEW]** — Swift async/await replacement for `NWConnection`; async `send(_:)`, `receive()`, `AsyncSequence` of messages
- `NetworkListener` **[NEW]** — async replacement for `NWListener`; `AsyncSequence<NetworkConnection>` of incoming clients
- `NetworkBrowser` **[NEW]** — async replacement for `NWBrowser`; `AsyncSequence` of discovered Bonjour/DNS-SD services
- `TLVFramer` **[NEW]** — built-in type-length-value message framing for stream-based connections
- `Coder` protocol **[NEW]** — type-safe encode/decode protocol for `NetworkConnection` messages; conforming types are directly sendable/receivable

**Wi-Fi Aware framework** (see Session 228)
- Integration: `WASubscribableService` discoveries and `WAPairedDevice` connections provide endpoints usable with `NetworkConnection` / `NetworkListener`

## Code Highlights

```swift
// Client: connect and send/receive with NetworkConnection
let connection = try NetworkConnection(to: endpoint, using: .tls)
try await connection.start()

// Send a Codable message
try await connection.send(MyMessage(text: "Hello"))

// Receive messages as an AsyncSequence
for try await message in connection.messages(MyMessage.self) {
    print(message.text)
}
```

```swift
// Server: listen for incoming connections
let listener = try NetworkListener(using: .tls, identity: tlsIdentity)
try await listener.start()

for try await incomingConnection in listener.connections {
    Task {
        for try await message in incomingConnection.messages(MyMessage.self) {
            let response = MyMessage(text: "Echo: \(message.text)")
            try await incomingConnection.send(response)
        }
    }
}
```

```swift
// TLV framing: send a typed frame
let framer = TLVFramer()
let framedConnection = NetworkConnection(to: endpoint, using: .tcp, framer: framer)
try await framedConnection.send(TLVFrame(type: 1, payload: data))
```

```swift
// Discovery with NetworkBrowser
let browser = NetworkBrowser(for: "_myapp._tcp")
for try await service in browser.services {
    let conn = try NetworkConnection(to: service.endpoint, using: .tcp)
    try await conn.start()
}
```

## Takeaways
- Migrate new networking code from `NWConnection`/`NWListener`/`NWBrowser` to `NetworkConnection`/`NetworkListener`/`NetworkBrowser` to use native Swift structured concurrency.
- Use `TLVFramer` to add message framing to TCP connections without writing custom framing code.
- Define a `Coder`-conforming type to send and receive strongly-typed messages over connections, eliminating manual `Data` marshaling.
- Pair Network framework with Wi-Fi Aware: use `WASubscribableService` to discover peers, then `NetworkConnection` for data I/O — the APIs compose cleanly.

---
_Source: WWDC25 Session 250 page (abstract, chapter summaries, code samples, and resource links)._
