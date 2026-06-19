# Get Ready for iCloud Private Relay
**WWDC21 · Session 10096** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10096/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12

## Overview
iCloud Private Relay is a new iCloud+ service that prevents networks and servers from monitoring user internet activity. It routes traffic through two separate secure proxies — one operated by Apple and one by a content provider — so that no single entity can see both the user's IP address and what they are accessing.

This session explains how Private Relay works, what traffic it affects, what developers need to do (almost nothing for most apps), how to prepare servers and websites, and how network administrators should adapt their infrastructure.

## Key Topics

**How iCloud Private Relay Works**
Without Private Relay, DNS queries expose visited hostnames to local network observers, and destination servers see the client's IP address (enabling geolocation and cross-site fingerprinting). Private Relay uses two-hop proxying: the first proxy (Apple-operated ingress proxy) sees the client IP but not the destination; the second proxy (third-party egress proxy) sees the destination but not the client IP. The client connects to the ingress proxy via QUIC (HTTP/3, UDP port 443). No entity in the chain — including Apple — can see both the client IP and the destination.

**What Traffic Private Relay Affects (iOS 15 / macOS 12)**
- All web browsing in Safari
- All DNS name resolution queries
- Insecure HTTP traffic from apps (TCP port 80)

**What Traffic Is NOT Affected**
- Local network connections
- Private domain name connections
- Traffic routed through VPN or app-proxy Network Extensions
- Traffic using an explicit proxy configuration

**App Developer Preparation**
Most apps need no changes. Best practices:
- Use modern networking APIs (`URLSession`, `NWConnection` from Network.framework) — they work correctly through Private Relay and expose metrics for diagnosing relay usage
- Migrate from `http://` to `https://` — insecure HTTP will be proxied but end-to-end security requires HTTPS
- Remove App Transport Security exceptions for insecure HTTP
- Use Core Location instead of server-side IP geolocation (IP-based location becomes unreliable/approximate with relay egress IPs)
- Check `URLSessionTaskTransactionMetrics.proxyConnection` to detect relay usage
- Use `NWConnection.EstablishmentReport` to inspect timing of DNS and proxy connection stages

**Server and Website Preparation**
- Relay egress proxy IP addresses map to city/region (not individual users) — geo IP databases must be updated to correctly map these ranges
- Region-based access restrictions remain enforceable (Private Relay prevents region spoofing)
- Stop relying on client IP for user identity — require explicit login instead
- Test that servers work correctly when receiving connections from proxy IP addresses

**Network Management**
- Networks will see increased UDP port 443 traffic (QUIC/HTTP/3 to ingress proxy)
- Fewer cleartext DNS queries visible on local network
- Enterprise/school networks that must inspect all traffic can block the iCloud Private Relay proxy hostname; users will then see a prompt to disable Private Relay on that network or switch networks
- Parental controls should use `NetworkExtension` content filter APIs (which see traffic before Private Relay)

## APIs & Frameworks

- **iCloud Private Relay** **[NEW]** — iCloud+ service, no developer API required
- `URLSession` — recommended for app networking; works correctly with Private Relay
- `URLSessionTaskTransactionMetrics` — networking metrics
  - `var proxyConnection: Bool` — indicates if task used a proxy (including Private Relay)
- `NWConnection` (Network.framework) — recommended low-level networking
  - `NWConnection.EstablishmentReport` — connection timing including DNS and each proxy hop
- `NWPathMonitor` — network path monitoring (existing API)
- **App Transport Security (ATS)** — review and remove insecure HTTP exceptions
- **Core Location** — preferred over IP-based geolocation **[NEW GUIDANCE]**
- **NetworkExtension** — `NEContentFilterProvider` — content filters see traffic before Private Relay
- QUIC / HTTP/3 — transport protocol used between device and ingress proxy
- UDP port 443 — used for QUIC connections to Private Relay ingress proxy

## Code Highlights

No new Swift/Objective-C API introduced in this session. Guidance is architectural. To detect Private Relay usage in URLSession:
```swift
// URLSessionTaskTransactionMetrics
let metrics: URLSessionTaskTransactionMetrics = ...
if metrics.proxyConnection {
    // Connection used a proxy (may include Private Relay)
}
```

To check NWConnection establishment stages including proxy timing:
```swift
// NWConnection EstablishmentReport
connection.pathUpdateHandler = { path in
    // path.usesInterfaceType(.cellular)
}
// Use NWConnection metrics callback for EstablishmentReport
```

## Takeaways

- Most apps work with iCloud Private Relay with zero code changes; the primary action is migrating from HTTP to HTTPS and removing ATS exceptions.
- IP-based geolocation and user fingerprinting by destination servers are broken by design — migrate to explicit location permission (Core Location) and explicit authentication (login).
- Server operators should update geo IP databases to correctly map Private Relay egress IP ranges to city/region, and not use client IP as a user identity signal.
- Network administrators can block the Private Relay proxy hostname to force users to choose between disabling Private Relay on that network or switching networks — this is the supported enterprise/school control mechanism.

---
_Source: WWDC21 Session 10096 page (abstract, chapter summaries, code samples, and resource links)._
