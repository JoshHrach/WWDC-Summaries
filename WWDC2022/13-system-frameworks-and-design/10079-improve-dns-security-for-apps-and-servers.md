# Improve DNS Security for Apps and Servers
**WWDC22 · Session 10079** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10079/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13

## Overview
DNS is the foundational protocol that translates domain names to IP addresses, but it was designed in 1983 with few security considerations. Two major vulnerabilities exist: DNS responses are unauthenticated (enabling cache poisoning attacks) and historically unencrypted (enabling DNS sniffing/surveillance). This session covers two complementary technologies that address these problems: DNSSEC for authentication and Discovery of Designated Resolvers (DDR) for automatic encrypted DNS.

iOS 16 and macOS Ventura add client-side DNSSEC validation support and automatic DDR-based encrypted DNS. Developers can require DNSSEC validation in their apps through a single property on `URLSessionConfiguration`, `URLRequest`, or `NWParameters`, and network operators can deploy DDR on their infrastructure to enable automatic DNS encryption for all clients without per-app configuration.

## Key Topics
- **DNSSEC (DNS Security Extensions)** — IETF suite that adds digital signatures to DNS responses; now supported client-side on iOS 16 / macOS Ventura
  - Data integrity: signatures attached to responses detect tampering; altered responses are discarded
  - Authenticated denial of existence: NSEC records prove which names exist or do not exist in a zone
  - Chain of trust: recursive trust from IP addresses up through DNS zone keys to a pre-installed root trust anchor
- **Adopting DNSSEC in apps** — requires IPv6 support for domains (synthesized addresses fail DNSSEC), DNSSEC signing by your DNS provider, and one property on session/request/parameters
- **DNSSEC failure cases** — altered responses, inability to reach trust anchor, network not supporting DNS-over-TCP or EDNS0, IPv6-only environments with unsigned domains
- **Discovery of Designated Resolvers (DDR)** — new IETF protocol (developed by Apple and partners) allowing DNS clients to auto-discover encrypted resolver configurations via a special `_dns.resolver.arpa` DNS query; new in iOS 16 / macOS Ventura
  - Works only when the DNS server's IP is a public IP (private IPs cannot be in TLS certificates)
  - Verifies that the unencrypted resolver's IP appears in the TLS certificate of the designated encrypted resolver
- **Client authentication for encrypted DNS** — new iOS 16 / macOS Ventura feature allows enterprise DNS servers to require client certificates; configured via `NEDNSSettings.identityReference`

## APIs & Frameworks
**Foundation / URLSession**
- `URLSessionConfiguration.requiresDNSSECValidation: Bool` **[NEW]** — enables DNSSEC validation for all requests in a session
- `URLRequest.requiresDNSSECValidation: Bool` **[NEW]** — enables DNSSEC validation for an individual request

**Network framework**
- `NWParameters.requiresDNSSECValidation: Bool` **[NEW]** — enables DNSSEC validation for an NWConnection; connection moves to `.ready` only after DNSSEC validation completes

**NetworkExtension**
- `NEDNSSettingsManager` — existing API for configuring encrypted DNS system-wide (iOS 14+)
- `NEDNSSettings.identityReference` **[NEW]** — property to specify a client certificate identity for encrypted DNS servers requiring client authentication; applies to both DNS-over-TLS and DNS-over-HTTPS

**DNS / Network Infrastructure**
- DNSSEC (RFC 4033/4034/4035) — digital signature suite for DNS
- NSEC records — authenticated denial-of-existence DNS record type
- DDR — Discovery of Designated Resolvers; `_dns.resolver.arpa` Service Binding query
- EDNS0 — DNS extension mechanism required for DNSSEC (DO bit)
- DNS-over-TLS / DNS-over-HTTPS — encrypted DNS transports used by DDR

## Code Highlights
Require DNSSEC at session level:
```swift
let configuration = URLSessionConfiguration.default
configuration.requiresDNSSECValidation = true
let session = URLSession(configuration: configuration)
```

Require DNSSEC for a single request:
```swift
var request = URLRequest(url: URL(string: "https://www.example.org")!)
request.requiresDNSSECValidation = true
let (data, response) = try await URLSession.shared.data(for: request)
```

Require DNSSEC for a Network.framework connection:
```swift
let parameters = NWParameters.tls
parameters.requiresDNSSECValidation = true
let connection = NWConnection(host: "www.example.org", port: .https, using: parameters)
```

## Takeaways
- DNSSEC client-side validation is now supported on iOS 16 / macOS Ventura; enabling it in an app requires one line of code plus server-side DNSSEC signing and IPv6 support.
- DDR enables fully automatic DNS encryption on networks that support it — no per-app configuration needed; network operators should deploy DDR on their DNS infrastructure to benefit all clients.
- DNSSEC validation failures are silent (treated as no response, not an error), so ensure your domain and network infrastructure are correctly configured before requiring DNSSEC in production.
- Client certificate authentication for enterprise encrypted DNS is now configurable via `NEDNSSettings.identityReference`.

---
_Source: WWDC22 Session 10079 page (abstract, chapter summaries, code samples, and resource links)._
