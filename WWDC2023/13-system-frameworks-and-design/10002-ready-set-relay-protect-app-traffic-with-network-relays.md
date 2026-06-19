# Ready, set, relay: Protect app traffic with network relays
**WWDC23 · Session 10002** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10002/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, tvOS 17

## Overview
This session introduces first-party support for MASQUE and Oblivious HTTP network relays as a modern, standards-based privacy tool for Apple platform apps. Relays already underpin iCloud Private Relay, Mail Privacy Protection, and Safari IP address hiding; in iOS/macOS 2023 they are opened to third-party developers through two new paths: in-app proxy configuration via `ProxyConfiguration` (Network, URLSession, WebKit), and system-wide relay configuration via `NERelayManager` or MDM profiles.

The session positions relays as a lighter-weight, more performant alternative to VPNs for enterprise access to private resources. Relays use TLS 1.3, QUIC/HTTP/3 (with HTTP/2 fallback), avoid tunnel/virtual-interface overhead, and support multiple simultaneous configurations targeting different internal domains — none of which traditional VPNs handle gracefully.

## Key Topics

### MASQUE Relays
MASQUE (Multiplexed Application Substrate over QUIC Encryption) is an IETF standard. It proxies any TCP or UDP connection without modifying the back-end server. Traffic is encrypted to the relay with TLS 1.3. Relays can be chained (multi-hop) so no single entity can correlate a client IP address with browsing activity — the basis of iCloud Private Relay.

### Oblivious HTTP
A simpler, single-hop relay for HTTP requests where server support is required. Ideal for anonymous metrics, DNS queries, or database lookups where requests must not be linkable to each other. Covered in depth in the companion "What's new in Privacy" session (10053).

### ProxyConfiguration for In-App Relays
The new `ProxyConfiguration` class is a unified way to define proxies (MASQUE, Oblivious HTTP, HTTP CONNECT with TLS, SOCKSv5) across Network framework, URLSession, and WebKit. All three APIs consume the same `ProxyConfiguration` object.

- **Network framework:** attach to an `NWParameters.PrivacyContext`
- **URLSession:** assign to `URLSessionConfiguration.proxyConfigurations`
- **WebKit:** assign to `WKWebsiteDataStore.proxyConfigurations`

### NERelayManager for Device-Wide Relays
`NERelayManager` (NetworkExtension) allows an app to install a relay configuration that applies to specific domains or the entire device — without requiring a VPN tunnel or virtual interface. Configurations appear in System Settings and persist across reboots. Requires the `com.apple.developer.networking.networkextension` entitlement.

### Enterprise Relay via MDM
A new `com.apple.relay.managed` MDM payload type installs relay configurations on managed devices. Supports client certificate authentication (via `PayloadCertificateUUID`), domain matching, and HTTP/3 or HTTP/2 relay URLs. Cisco Secure Edge provides a compatible enterprise relay service.

### tvOS Network Extension Support
Network Extension support (including VPN) is newly added to tvOS 17.

## APIs & Frameworks

- **Network framework**
  - `ProxyConfiguration` — unified proxy/relay configuration object **[NEW]**
    - `ProxyConfiguration(relayHops:)` — init with MASQUE relay hops **[NEW]**
    - `ProxyConfiguration.RelayHop(http3RelayEndpoint:)` — single relay hop, HTTP/3 **[NEW]**
    - `ProxyConfiguration.RelayHop(http3RelayEndpoint:http2RelayEndpoint:)` — dual-protocol hop **[NEW]**
    - `ProxyConfiguration(socksv5Proxy:)` — SOCKSv5 **[NEW]**
    - `ProxyConfiguration(httpCONNECTProxy:tls:)` — HTTP CONNECT with optional TLS **[NEW]**
  - `NWParameters.PrivacyContext` — context object for per-connection proxy settings
    - `proxyConfigurations: [ProxyConfiguration]`
  - `NWParameters.setPrivacyContext(_:)` — attach privacy context to parameters
  - `NWConnection` — standard connection object; gains relay support via parameters
  - `NWEndpoint.url(_:)` — create endpoint from HTTPS URL
- **Foundation / URLSession**
  - `URLSessionConfiguration.proxyConfigurations: [ProxyConfiguration]` **[NEW]**
- **WebKit**
  - `WKWebsiteDataStore.proxyConfigurations: [ProxyConfiguration]` **[NEW]**
  - `WKWebViewConfiguration.websiteDataStore` — attach data store to web view config
- **NetworkExtension**
  - `NERelay` — represents a single relay server **[NEW]**
    - `http3RelayURL: URL?`
    - `http2RelayURL: URL?`
    - `additionalHTTPHeaderFields: [String: String]` — e.g., Authorization headers
  - `NERelayManager` — manages device-wide relay configuration **[NEW]**
    - `NERelayManager.shared()` — singleton
    - `relays: [NERelay]`
    - `matchDomains: [String]` — domains to route through relay; empty = whole device
    - `isEnabled: Bool`
    - `saveToPreferences() async throws` — installs configuration into system
- **MDM Payload**
  - `com.apple.relay.managed` — new MDM payload type for managed relay configurations **[NEW]**
    - `HTTP3RelayURL` / `HTTP2RelayURL` — relay server URLs
    - `PayloadCertificateUUID` — client certificate reference
    - `MatchDomains` — domain filter array

## Code Highlights

Configuring a MASQUE relay in Network framework:
```swift
import Network
let relayEndpoint = NWEndpoint.url(URL(string: "https://relay.example.com")!)
let relayServer = ProxyConfiguration.RelayHop(http3RelayEndpoint: relayEndpoint)
let relayConfig = ProxyConfiguration(relayHops: [relayServer])

var context = NWParameters.PrivacyContext(description: "my relay")
context.proxyConfigurations = [relayConfig]
let parameters = NWParameters.tls
parameters.setPrivacyContext(context)
let connection = NWConnection(host: "www.example.com", port: 443, using: parameters)
connection.start(queue: .main)
```

Configuring a relay in URLSession:
```swift
let config = URLSessionConfiguration.default
config.proxyConfigurations = [relayConfig]
let mySession = URLSession(configuration: config)
let (data, response) = try await mySession.data(from: URL(string: "https://www.example.com/api")!)
```

Installing a device-wide relay via NERelayManager:
```swift
import NetworkExtension
let newRelay = NERelay()
newRelay.http3RelayURL = URL(string: "https://relay.example.com:443/")
newRelay.http2RelayURL = newRelay.http3RelayURL
newRelay.additionalHTTPHeaderFields = ["Authorization": "PrivateToken=123"]

let manager = NERelayManager.shared()
manager.relays = [newRelay]
manager.matchDomains = ["internal.example.com"]
manager.isEnabled = true
try await manager.saveToPreferences()
```

## Takeaways

- The new `ProxyConfiguration` class makes it trivial to route app traffic (Network, URLSession, WebKit) through MASQUE or Oblivious HTTP relays with no back-end server changes required.
- Multi-hop MASQUE relay chains prevent any single server from correlating a user's IP address with their activity — the same privacy model as iCloud Private Relay.
- `NERelayManager` enables enterprises to replace VPNs with lighter, faster relays that users can configure from within an app and that appear transparently in System Settings.
- Oblivious HTTP provides single-hop, unlinkable privacy for HTTP-specific use cases like anonymous analytics or DNS; server-side support is required.

---
_Source: WWDC23 Session 10002 page (abstract, chapter summaries, code samples, and resource links)._
