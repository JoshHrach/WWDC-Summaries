# Boost Performance and Security with Modern Networking
**WWDC20 · Session 10111** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10111/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, tvOS 14, watchOS 7

## Overview
This session surveys the modern networking protocol stack on Apple platforms and provides concrete guidance for server-side configuration to ensure apps benefit from every available performance, security, mobility, and privacy improvement. The key message: if an app already uses `URLSession` or `Network.framework`, all the technologies discussed are available automatically — the primary work is enabling them on the server.

The session covers five protocol layers: IPv6 (lower latency, faster connection setup), HTTP/2 (multiplexing, connection coalescing, header compression), TLS 1.3 (faster handshake, better security), Multipath TCP (seamless network transitions), and Encrypted DNS (DNS-over-TLS / DNS-over-HTTPS). It also introduces two new features in iOS 14: Local Network Privacy protections and experimental HTTP/3/QUIC support.

## Key Topics
- **IPv6** — Apple platforms have native IPv6 support; IPv6 connections are 1.4x faster median setup than IPv4 (less NAT, better routing); 26% of all Apple device connections now use IPv6; 20% could but don't because the server lacks IPv6. Apps using `URLSession` or `Network.framework` get IPv6 support automatically via Happy Eyeballs. Test with NAT64 Internet Sharing on Mac (an App Store submission requirement).
- **HTTP/2** — Default in `URLSession`; multiplexes requests on a single connection; connection coalescing reuses connections for requests to the same server; header compression saves bandwidth; 79% of Safari requests now use HTTP/2; HTTP/2 tasks are 1.8x faster than HTTP/1.1 tasks. Enable HTTP/2 on server; client side is automatic.
- **TLS 1.3** — Default since iOS 13.4 in `URLSession` and `Network.framework`; one fewer round-trip in handshake; 49% of all TLS connections now use TLS 1.3; 1.3x faster handshake than TLS 1.2. Enable on server; client side is automatic.
- **Multipath TCP** — Allows a single TCP connection to continue as the device changes networks (e.g., Wi-Fi → cellular); client opts in via `multipathServiceType`; Apple Music adoption reduced stalls by 13% and stall duration by 22%. Requires server-side enablement (see multipath-tcp.org).
- **Encrypted DNS (iOS 14, macOS Big Sur)** **[NEW]** — System-level DNS-over-TLS and DNS-over-HTTPS support; configured system-wide via `NetworkExtension`; once configured, all apps benefit automatically. Apps can also require encrypted DNS per-connection. Covered in depth in the companion "Enable Encrypted DNS" session.
- **Local Network Privacy (iOS 14)** **[NEW]** — Accessing any local network resource (multicast, broadcast, or direct LAN access) now requires explicit user permission and a reason string in `Info.plist`. System services (AirPrint, AirPlay, HomeKit) are exempt. Details in companion "Support Local Network Privacy in Your App" session.
- **HTTP/3 / QUIC (experimental preview, iOS 14)** **[NEW]** — TLS 1.3 built in; multiplexed streams without head-of-line blocking; built-in mobility (network transitions don't interrupt in-progress operations). Enabled via Developer Settings (iOS) or `CFNetworkHTTP3Override` user default (macOS). Still IETF in-progress; submit feedback.
- **Encrypted SNI (upcoming)** — Currently TLS handshakes expose cleartext Server Name Indication; Apple is working with IETF to standardize Encrypted Client Hello (ECH) to hide SNI; no action required yet.

## APIs & Frameworks

### Foundation / URLSession
- **`URLSessionConfiguration.multipathServiceType`** — `URLSessionConfiguration.MultipathServiceType`; `.none` (default), `.handover`, `.interactive`, `.aggregate`; enables Multipath TCP
- **`URLSession`** — Automatically uses IPv6 (Happy Eyeballs), HTTP/2, TLS 1.3, and (experimentally) HTTP/3 when enabled on the server; no configuration required for these protocols

### Network.framework
- **`NWParameters.multipathServiceType`** — `NWParameters.MultipathServiceType`; enables Multipath TCP per connection
- **`NWParameters`** — Connections automatically negotiate IPv6, TLS 1.3; no explicit configuration needed

### NetworkExtension (Encrypted DNS — iOS 14 / macOS Big Sur)
- **`NEDNSSettingsManager`** **[NEW]** — Configure system-wide encrypted DNS resolvers (DNS-over-HTTPS or DNS-over-TLS) via a NetworkExtension app
- **`NEDNSSettings`** **[NEW]** — Base class for DNS settings
- **`NEDNSOverHTTPSSettings`** **[NEW]** — DNS-over-HTTPS configuration: `serverURL`
- **`NEDNSOverTLSSettings`** **[NEW]** — DNS-over-TLS configuration: `serverName`

### Info.plist (Local Network Privacy — iOS 14)
- **`NSLocalNetworkUsageDescription`** **[NEW]** — Required string key describing why the app needs local network access; triggers the system permission prompt when local network is first accessed

## Code Highlights

Enable Multipath TCP in URLSession:
```swift
let config = URLSessionConfiguration.default
config.multipathServiceType = .handover  // seamless Wi-Fi ↔ cellular
let session = URLSession(configuration: config)
```

Enable Multipath TCP in Network.framework:
```swift
let params = NWParameters.tcp
params.multipathServiceType = .handover
let connection = NWConnection(host: "example.com", port: 443, using: params)
```

Server-side checklist (no code — configuration only):
```
✅ Enable IPv6 on your server
✅ Enable HTTP/2 (e.g., nginx: http2 on; Apache: Protocols h2 http/1.1)
✅ Enable TLS 1.3 (OpenSSL/BoringSSL: ssl_protocols TLSv1.3)
✅ Enable Multipath TCP (see multipath-tcp.org for OS-specific instructions)
✅ Enable DNS-over-HTTPS or DNS-over-TLS at your DNS provider
```

Test HTTP/3 experimentally (iOS 14):
```
iOS: Settings > Developer > Enable HTTP/3
macOS: defaults write CFNetworkHTTP3Override 3
Safari macOS: Develop > Experimental Features > HTTP/3
```

## Takeaways
- Using `URLSession` or `Network.framework` is all the client-side work needed — IPv6, HTTP/2, TLS 1.3, and eventually HTTP/3 are automatic when the server supports them.
- Enable IPv6 on your server: 26% of Apple connections already use it and those connections set up 1.4x faster; 20% of connections would use it if the server supported it.
- TLS 1.3 (on by default since iOS 13.4) speeds up handshakes 1.3x; HTTP/2 (on by default in URLSession) speeds up task completion 1.8x vs HTTP/1.1 — both require only a server setting change.
- Opt into `multipathServiceType = .handover` for streaming or communication apps to maintain connections across Wi-Fi/cellular transitions; Apple Music saw a 13% stall reduction after adopting it.
- iOS 14 requires a `NSLocalNetworkUsageDescription` key for any app that accesses the local network; test and add this before submission to avoid rejection.

---
_Source: WWDC20 Session 10111 page (abstract and transcript)._
