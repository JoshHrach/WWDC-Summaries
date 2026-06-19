# Introducing Network.framework: A Modern Alternative to Sockets
**WWDC18 · Session 715** · [Watch](https://developer.apple.com/videos/play/wwdc2018/715/)

_Platforms:_ iOS 12, macOS Mojave 10.14, watchOS 5, tvOS 12

## Overview
Network.framework is a new Swift- and Objective-C-native networking API introduced in iOS 12 and macOS Mojave that replaces the BSD socket layer for most use cases. It brings direct access to the same transport layer used by URLSession — including TLS 1.3, QUIC (preview), Multipath TCP, ECN, and Happy Eyeballs — from a clean, callback-driven API with no socket file descriptors, no manual poll/select loops, and no thread management.

The session covers TCP and UDP connections (`NWConnection`), servers (`NWListener`), path monitoring (`NWPathMonitor`), and protocol stacking (`NWParameters`). All callbacks deliver results on a caller-supplied `DispatchQueue`, eliminating threading complexity inherent in socket-based code.

## Key Topics

### Why Network.framework

- BSD sockets require manual management of file descriptors, select/poll loops, multithreading, TLS handshakes, and reconnection logic. Even a "simple" TLS TCP client requires 200+ lines of C.
- `URLSession` wraps sockets internally and gets all Apple transport-layer optimizations for free; Network.framework exposes the same optimized layer to apps needing direct connection control (games, real-time messaging, custom protocols).
- Key guarantees over raw sockets:
  - Automatic TLS (via `NWProtocolTLS`) integrated into the connection state machine.
  - Built-in Happy Eyeballs (tries IPv6 and IPv4 simultaneously; uses whichever connects first).
  - Multipath TCP support — connections survive Wi-Fi → Cellular handoff.
  - ECN (Explicit Congestion Notification) awareness.
  - Coalescing with existing connections when possible (QUIC).

### NWConnection — Making Connections

- `NWConnection(host:port:using:)` — create a TCP or UDP connection to a host+port with a given protocol stack.
- `stateUpdateHandler: (NWConnection.State) -> Void` — callback fired on state changes: `.setup`, `.preparing`, `.ready`, `.failed(error:)`, `.cancelled`.
- `start(queue:)` — start the connection; provide a `DispatchQueue` for all callbacks.
- `send(content:contentContext:isComplete:completion:)` — send `Data`; completion block fires when data is handed to the network stack (not when the remote receives it).
- `receive(minimumIncompleteLength:maximumLength:completion:)` — async receive; fires when at least `minimumIncompleteLength` bytes arrive. Call again in the completion to loop.
- `cancel()` — cancel the connection cleanly (sends TCP FIN or TLS close-notify).
- `forceCancel()` — hard reset (TCP RST).

### NWListener — Building Servers

- `NWListener(using:on:)` — listen on a port for incoming connections of the given protocol stack.
- `newConnectionHandler: (NWConnection) -> Void` — called for each accepted connection.
- `stateUpdateHandler` — same states as `NWConnection`.
- `start(queue:)` / `cancel()` — same lifecycle as connections.
- For Bonjour services: set `listener.service = NWListener.Service(type: "_myapp._tcp")` and the system handles registration/browsing automatically.

### NWPathMonitor — Observing Network Availability

- `NWPathMonitor()` — observe all interfaces.
- `NWPathMonitor(requiredInterfaceType:)` — observe a specific interface type (`.wifi`, `.cellular`, `.wiredEthernet`, `.loopback`, `.other`).
- `pathUpdateHandler: (NWPath) -> Void` — fired on network path changes.
- `NWPath.status` — `.satisfied` (usable), `.unsatisfied`, `.requiresConnection`.
- `NWPath.isExpensive` — true when the path goes through a cellular or personal hotspot connection; use to defer large data transfers.
- `NWPath.supportsIPv4` / `.supportsIPv6`.
- `start(queue:)` / `cancel()`.
- Replaces `SCNetworkReachability` with a modern, queue-based callback API.

### NWParameters — Protocol Stack Configuration

- `NWParameters.tcp` — default TLS-over-TCP parameters.
- `NWParameters.udp` — default UDP parameters.
- `NWParameters(tls:tcp:)` — combine a `NWProtocolTLS.Options` and `NWProtocolTCP.Options` for custom TLS/TCP configuration.
- `NWProtocolTLS.Options` — set minimum/maximum TLS version, client certificates, server name override, custom verification via `sec_protocol_options_set_verify_block`.
- `NWProtocolTCP.Options` — `enableKeepalive`, `keepaliveIdle`, `keepaliveCount`, `keepaliveInterval`, `noDelay`, `connectionTimeout`.
- `NWProtocolUDP.Options` — `preferNoChecksum`.
- `parameters.requiredInterfaceType` — force a specific interface.
- `parameters.prohibitedInterfaceTypes` — exclude interface types.
- `parameters.multipathServiceType` — `.handover`, `.interactive`, `.aggregate` for Multipath TCP.
- `parameters.allowLocalEndpointReuse` — allow port reuse for UDP multicast.

### Endpoint Types (`NWEndpoint`)

- `NWEndpoint.hostPort(host:port:)` — explicit host (DNS name or IP) and port.
- `NWEndpoint.service(name:type:domain:interface:)` — Bonjour service endpoint; resolved automatically when the connection starts.
- `NWEndpoint.unix(path:)` — local Unix domain socket.

### Framing Protocols (Preview, iOS 12)

- `NWProtocolFramer` — custom message-framing protocol that runs in the network stack (not on the app thread), allowing zero-copy framing and interoperability with other protocol layers.
- Declare input/output messages; the system handles buffering and reassembly.
- Useful for length-prefixed, newline-delimited, or other fixed-structure wire formats.

## APIs & Frameworks

**Network.framework** **[NEW — iOS 12, macOS 10.14]**
- `NWConnection` — `init(host:port:using:)`, `init(to:using:)`, `stateUpdateHandler`, `start(queue:)`, `cancel()`, `forceCancel()`, `send(content:contentContext:isComplete:completion:)`, `receive(minimumIncompleteLength:maximumLength:completion:)`, `currentPath`, `viabilityUpdateHandler`, `betterPathUpdateHandler`
- `NWConnection.State` — `.setup`, `.preparing`, `.ready`, `.failed(NWError)`, `.cancelled`, `.waiting(NWError)`
- `NWListener` — `init(using:on:)`, `newConnectionHandler`, `stateUpdateHandler`, `start(queue:)`, `cancel()`, `service`, `serviceRegistrationUpdateHandler`
- `NWListener.Service` — `init(name:type:domain:)` for Bonjour
- `NWPathMonitor` — `init()`, `init(requiredInterfaceType:)`, `pathUpdateHandler`, `start(queue:)`, `cancel()`, `currentPath`
- `NWPath` — `status` (`.satisfied`, `.unsatisfied`, `.requiresConnection`), `isExpensive`, `isConstrained`, `supportsIPv4`, `supportsIPv6`, `usesInterfaceType(_:)`
- `NWParameters` — `tcp`, `udp`, `init(tls:tcp:)`, `init(dtls:udp:)`, `requiredInterfaceType`, `prohibitedInterfaceTypes`, `multipathServiceType`, `allowLocalEndpointReuse`, `includePeerToPeer`
- `NWProtocolTLS.Options` — `securityProtocolOptions` (maps to `sec_protocol_options_t`)
- `NWProtocolTCP.Options` — `enableKeepalive`, `noDelay`, `connectionTimeout`, `maximumSegmentSize`
- `NWProtocolUDP.Options` — `preferNoChecksum`
- `NWEndpoint` — `.hostPort(host:port:)`, `.service(name:type:domain:interface:)`, `.unix(path:)`
- `NWInterface.InterfaceType` — `.wifi`, `.cellular`, `.wiredEthernet`, `.loopback`, `.other`
- `NWError` — `.posix(POSIXErrorCode)`, `.dns(DNSServiceErrorType)`, `.tls(OSStatus)`
- `NWProtocolFramer` — custom framing protocol (preview)

## Code Highlights

Minimal TCP connection with TLS:
```swift
let connection = NWConnection(host: "example.com", port: 443, using: .tls)

connection.stateUpdateHandler = { state in
    switch state {
    case .ready:
        print("Connected")
        sendMessage(connection)
    case .failed(let error):
        print("Failed: \(error)")
        connection.cancel()
    default: break
    }
}
connection.start(queue: .global())

func sendMessage(_ conn: NWConnection) {
    let data = "GET / HTTP/1.1\r\nHost: example.com\r\n\r\n".data(using: .utf8)!
    conn.send(content: data, completion: .contentProcessed { error in
        if let error = error { print("Send error: \(error)"); return }
        receiveMessage(conn)
    })
}

func receiveMessage(_ conn: NWConnection) {
    conn.receive(minimumIncompleteLength: 1, maximumLength: 65536) { data, _, isComplete, error in
        if let data = data { print("Received: \(data.count) bytes") }
        if !isComplete { receiveMessage(conn) }  // loop
    }
}
```

Path monitoring to defer expensive transfers:
```swift
let monitor = NWPathMonitor()
monitor.pathUpdateHandler = { path in
    if path.status == .satisfied && !path.isExpensive {
        startBackgroundSync()
    }
}
monitor.start(queue: DispatchQueue.global(qos: .background))
```

UDP listener for a local game (Bonjour):
```swift
let params = NWParameters.udp
let listener = try! NWListener(using: params, on: 7777)
listener.service = NWListener.Service(type: "_mygame._udp")
listener.newConnectionHandler = { connection in
    connection.start(queue: .global())
    receive(from: connection)
}
listener.start(queue: .global())
```

## Takeaways
- Network.framework is the recommended replacement for BSD sockets and `SCNetworkReachability` on all Apple platforms as of iOS 12 / macOS 10.14. Use it for direct TCP/UDP connections, custom protocols, and servers.
- All callbacks run on a caller-supplied `DispatchQueue` — no thread management needed. Use a dedicated serial queue per connection/listener.
- `NWPathMonitor` with `path.isExpensive` is the correct way to detect cellular/hotspot and defer large transfers in iOS 12+.
- TLS and Happy Eyeballs are built into the connection state machine — there is no need to perform TLS handshakes manually.

---
_Source: WWDC18 Session 715 page (abstract, full transcript, and resource links)._
