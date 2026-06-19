# Enable Encrypted DNS
**WWDC20 · Session 10047** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10047/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, tvOS 14, watchOS 7

## Overview
iOS 14 and macOS Big Sur add native support for encrypted DNS at the system level, addressing two longstanding privacy problems: DNS queries and answers are normally sent in plaintext over UDP (visible and tamperable by any device on the network), and the DNS resolver is usually assigned by the local network operator, who may track or block queries.

Apple platforms support two encrypted DNS protocols: DNS over TLS (DoT) and DNS over HTTPS (DoH). Developers can enable encrypted DNS in one of two ways — configuring a system-wide resolver that applies to all apps, or opting in for specific connections within their own app — using standard platform APIs.

## Key Topics

### System-Wide DNS Configuration
An app can configure the system's DNS settings using `NEDNSSettingsManager` (requires the "DNS Settings" NetworkExtension capability in Xcode, no extension point needed). An MDM profile with a `DNSSettings` payload achieves the same result for managed devices.

The configuration includes:
- **Server**: `NEDNSOverHTTPSSettings` (DoH, requires a server URL) or `NEDNSOverTLSSettings` (DoT). Optional server IP addresses can be provided to avoid a bootstrap DNS lookup.
- **Network Rules** (ordered list of `NEOnDemandRule` objects): control when the configuration applies — by network type (Wi-Fi, cellular) or by SSID. Captive network detection and VPN-tunnel DNS are handled automatically without requiring rules.

Rule types:
- `NEOnDemandRuleEvaluateConnection` — match a specific SSID or interface type, then define per-domain exceptions using `NEEvaluateConnectionRule` with `.neverConnect` for private names.
- `NEOnDemandRuleDisconnect` — disable the encrypted DNS config on a matched network.
- `NEOnDemandRuleConnect` — enable the config; use as the final catchall rule.

**Important**: some networks actively block encrypted DNS. When this occurs, iOS shows a Wi-Fi privacy warning icon and app connections fail rather than silently fall back to unencrypted DNS.

### Per-App Encrypted DNS with Network Framework
Apps that cannot or do not want to modify system settings can opt in using `NWParameters.PrivacyContext` (new in iOS 14):

- Create one `PrivacyContext` per group of connections sharing the same DNS settings.
- Call `requireEncryptedNameResolution(_:fallbackResolver:)` — specifying a `.https(url:serverAddresses:)` or `.tls(serverName:serverAddresses:)` fallback resolver. The system's encrypted DNS configuration takes precedence; the fallback applies only when no system config is present.
- Set the `PrivacyContext` on `NWParameters` before creating connections.
- To apply to the entire app (including `URLSession` tasks and POSIX `getaddrinfo` calls), configure `NWParameters.PrivacyContext.default`.

### Validating Encrypted DNS Use
After a connection is established, call `NWConnection.requestEstablishmentReport(queue:completionHandler:)`. The resulting `NWEstablishmentReport` contains a `resolutions` array; each entry has a `dnsProtocol` property:
- `.https` / `.tls` — encrypted DNS was used.
- `.udp` / `.tcp` — plaintext DNS was used.
- No protocol set — answer came from a cache (not a live lookup).

## APIs & Frameworks

### NetworkExtension (system-wide configuration)
- `NEDNSSettingsManager.shared()` — singleton for DNS settings management **[NEW]**
- `NEDNSSettingsManager.loadFromPreferences(completionHandler:)` — load existing config
- `NEDNSSettingsManager.saveToPreferences(completionHandler:)` — save and install config
- `NEDNSOverHTTPSSettings(servers:)` — DoH configuration; `.serverURL: URL` required **[NEW]**
- `NEDNSOverTLSSettings(servers:)` — DoT configuration **[NEW]**
- `NEOnDemandRuleEvaluateConnection` — match network + per-domain rules
- `NEEvaluateConnectionRule(matchDomains:andAction:)` — `.neverConnect` / `.connectIfNeeded`
- `NEOnDemandRuleDisconnect` — disable DNS on matched network
- `NEOnDemandRuleConnect` — enable DNS on matched network (use as final catchall)
- `NEDNSSettingsManager.onDemandRules: [NEOnDemandRule]` — ordered rule list

### Network Framework (per-app/per-connection)
- `NWParameters.PrivacyContext` **[NEW]** — DNS settings scope for a group of connections
- `PrivacyContext.requireEncryptedNameResolution(_:fallbackResolver:)` — opt in to encrypted DNS with fallback
- `NWParameters.PrivacyContext.default` — apply settings to all app connections (including URLSession)
- `NWParameters.setPrivacyContext(_:)` — attach context to connection parameters
- `NWConnection.requestEstablishmentReport(queue:completionHandler:)` — retrieve `NWEstablishmentReport`
- `NWEstablishmentReport.resolutions: [NWEstablishmentReport.ResolutionReport]`
- `ResolutionReport.dnsProtocol: NWEstablishmentReport.ResolutionReport.Protocol` — `.https`, `.tls`, `.udp`, `.tcp`

## Code Highlights

Configuring a system-wide DoH resolver with Network Rules:
```swift
NEDNSSettingsManager.shared().loadFromPreferences { loadError in
    let dohSettings = NEDNSOverHTTPSSettings(servers: ["2001:db8::2"])
    dohSettings.serverURL = URL(string: "https://dnsserver.example.net/dns-query")
    NEDNSSettingsManager.shared().dnsSettings = dohSettings

    let workWiFi = NEOnDemandRuleEvaluateConnection()
    workWiFi.interfaceTypeMatch = .wiFi
    workWiFi.ssidMatch = ["MyWorkWiFi"]
    workWiFi.connectionRules = [
        NEEvaluateConnectionRule(matchDomains: ["enterprise.example.net"],
                                 andAction: .neverConnect)
    ]
    let disableOnCell = NEOnDemandRuleDisconnect()
    disableOnCell.interfaceTypeMatch = .cellular
    let enableByDefault = NEOnDemandRuleConnect()

    NEDNSSettingsManager.shared().onDemandRules = [workWiFi, disableOnCell, enableByDefault]
    NEDNSSettingsManager.shared().saveToPreferences { _ in }
}
```

Using per-connection encrypted DNS with Network framework:
```swift
let privacyContext = NWParameters.PrivacyContext(description: "EncryptedDNS")
if let url = URL(string: "https://dnsserver.example.net/dns-query") {
    let address = NWEndpoint.hostPort(host: "2001:db8::2", port: 443)
    privacyContext.requireEncryptedNameResolution(
        true, fallbackResolver: .https(url, serverAddresses: [address]))
}
let tlsParams = NWParameters.tls
tlsParams.setPrivacyContext(privacyContext)
let conn = NWConnection(host: "www.example.com", port: 443, using: tlsParams)
conn.start(queue: .main)
```

Applying encrypted DNS to all app connections (URLSession, getaddrinfo):
```swift
if let url = URL(string: "https://dnsserver.example.net/dns-query") {
    let address = NWEndpoint.hostPort(host: "2001:db8::2", port: 443)
    NWParameters.PrivacyContext.default.requireEncryptedNameResolution(
        true, fallbackResolver: .https(url, serverAddresses: [address]))
}
```

## Takeaways
- iOS 14 natively supports DoH and DoT for both system-wide and per-app encrypted DNS — no third-party VPN or proxy required.
- Use `NEDNSSettingsManager` to ship a DNS settings app; network rules are essential for enterprise compatibility (private DNS names that your server cannot resolve).
- For per-app encrypted DNS without system-wide configuration, set a `PrivacyContext` on `NWParameters`; configuring `NWParameters.PrivacyContext.default` extends coverage to URLSession and POSIX APIs.
- Networks that actively block encrypted DNS will cause connection failures and display a Wi-Fi privacy warning — do not provide a plaintext fallback.

---
_Source: WWDC20 Session 10047 page (abstract, transcript, and code samples)._
