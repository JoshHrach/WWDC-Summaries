# Deploying AirPrint in Enterprise
**WWDC16 · Session 725** · [Watch](https://developer.apple.com/videos/play/wwdc2016/725/)

_Platforms:_ iOS 10, macOS Sierra 10.12

## Overview
This session is aimed at IT administrators and enterprise deployment specialists responsible for making AirPrint work across complex corporate networks. It reviews the full suite of AirPrint enterprise features: PDF creation, end-to-end encryption, PIN release, authentication and accounting, and printer discovery mechanisms including Wide-Area Bonjour DNS configuration, MDM profiles, and the brand-new AirPrint Bluetooth Beacon introduced in iOS 10.

The session is practical and configuration-focused, walking through exact DNS zone file records, Bluetooth beacon byte formats, and MDM payload structure. It complements the developer-focused "Advances in AirPrint" session and the AirPrint licensing information available at developer.apple.com/airprint.

## Key Topics

### PDF Creation from the Print Panel (iOS 10)
- New in iOS 10: all iOS devices can create a PDF anywhere the Print panel is available by pinching out on the print preview thumbnail.
- The Share sheet appears, allowing any PDF-accepting app to receive the document; supports AirDrop and Managed Open In (enterprise document separation).
- Devices with 3D Touch can use Peek and Pop on the print preview to reach the same share view.

### Security
- AirPrint supports TLS 1.2 end-to-end encryption over HTTP (IPP over HTTPS / IPPS); required for all new AirPrint printers and servers.
- **PIN Release**: users enter a PIN at the printer to release jobs; iOS shows the PIN in an alert after tapping Print; macOS allows the user to specify their own PIN. Supports both required and optional PIN workflows.

### Access Control and Accounting
- Username/password authentication credentials stored in the keychain; iOS 10 adds the ability to forget stored credentials for multi-user printer workflows **[NEW]**.
- **Password-only authentication** **[NEW in iOS 10]** — printer protected with a simple shared password.
- Account ID / billing code support: required or optional account information can be attached to each print job on both iOS and macOS.

### Printer Discovery Mechanisms

#### Local Bonjour (no setup required)
- Printers on the same subnet appear automatically — the standard AirPrint experience.

#### Wide-Area Bonjour (DNS-SD)
- Add DNS records to a server in the device's DNS server list:
  - A/AAAA record for the printer's static IP
  - PTR record for `_ipps._tcp` service type
  - PTR record for `_universal._sub._ipps._tcp` subtype (required for AirPrint discovery)
  - SRV record pointing to the fully-qualified domain name
  - TXT record with printer capabilities (copy from `dns-sd -Z _ipps._tcp. local.` output on the same subnet, then update `local` references to FQDN)
- Key gotcha: replace all `local.` Bonjour name references in the SRV and TXT records with the printer's FQDN; otherwise the macOS admin webpage button will not work.

#### MDM AirPrint Payload
- Add an AirPrint payload to any MDM profile with `IPAddress` (or hostname) and `ResourcePath` fields.
- `ResourcePath` is `ipp/print` for most printers; for print servers, this is the queue path.
- Works with Apple Configurator and any MDM system supporting the AirPrint payload.

#### AirPrint Bluetooth Beacon (NEW in iOS 10)
- A BLE beacon placed near or built into a printer that advertises the printer's IP address and resource path.
- iOS discovers the printer if it is in Bluetooth range AND the advertised IP address is reachable (even a public internet IP).
- Beacon format (manufacturer-specific BLE advertisement data):
  - Fixed header identifying the payload as AirPrint
  - Connection info byte: IPv4 vs. IPv6, direct printer vs. server queue, TLS required or not
  - Printer ID / resource path field
  - Port (standard IPP: 631 / `0x0277`; TLS: 443)
  - IP address (static; dynamic IPs will go stale)
  - Measured signal power at 1 meter (same methodology as iBeacon measured power)
- Beacon format is similar to iBeacon but one byte longer.
- Can be implemented using off-the-shelf third-party BLE beacon hardware or built into printer firmware; future AirPrint printers will include it natively.
- Works well with print servers: each beacon advertises the server's IP and the queue ID for the associated printer.

## APIs & Frameworks

- **AirPrint** — Apple's driverless printing technology
- **IPP over HTTPS (IPPS)** — TLS 1.2 encrypted print protocol
- **Bonjour / DNS-SD** — local and wide-area printer discovery
  - `_ipps._tcp` service type
  - `_universal._sub._ipps._tcp` subtype (required for AirPrint)
  - `dns-sd -Z _ipps._tcp. local.` command-line tool for extracting DNS zone records
- **AirPrint Bluetooth Beacon** **[NEW in iOS 10]** — BLE-based printer discovery
  - Manufacturer-specific BLE advertisement format (specification published at developer.apple.com/wwdc16/725)
  - IPv4 and IPv6 support
  - TLS-required flag
  - iBeacon-compatible setup flow
- **MDM AirPrint Payload** — `IPAddress`, `ResourcePath` fields in configuration profile
- **Managed Open In** — enterprise document separation; compatible with PDF creation workflow
- `UIPrintInteractionController` — iOS print panel (developer-side; see "Advances in AirPrint")
- PIN Release — IPP job attribute (`job-password`) support in AirPrint-certified printers
- Keychain — stores per-printer username/password credentials
- **Password-only authentication** **[NEW in iOS 10]** — printer-level shared password without username

## Code Highlights

No developer code in this session (IT/deployment focused). Key configuration snippets:

DNS zone file entries for Wide-Area Bonjour (illustrative):
```
; PTR records
_ipps._tcp.example.com.        PTR  MyPrinter._ipps._tcp.example.com.
_universal._sub._ipps._tcp.example.com. PTR MyPrinter._ipps._tcp.example.com.

; SRV record
MyPrinter._ipps._tcp.example.com. SRV 0 0 631 printer.example.com.

; TXT record (copy from dns-sd output, replace local. with FQDN)
MyPrinter._ipps._tcp.example.com. TXT "txtvers=1" "qtotal=1" "rp=ipp/print" ...
```

AirPrint Bluetooth Beacon advertisement bytes (illustrative structure):
```
[Header: 10 bytes fixed] [ConnInfo: 1 byte] [ResourcePath/QueueID] [Port: 2 bytes] [IP: 4 or 16 bytes] [MeasuredPower: 1 byte]
```

## Takeaways
- iOS 10 adds PDF creation directly from the print panel on all devices — a key enterprise paper-free workflow; no developer changes required, users just pinch out on the print preview.
- The AirPrint Bluetooth Beacon is the most flexible new discovery mechanism: it works across subnets and VLANs as long as the IP is reachable, making network topology irrelevant for printer visibility.
- Wide-Area Bonjour DNS setup requires adding a `_universal._sub._ipps._tcp` PTR record in addition to the standard `_ipps._tcp` PTR record; use `dns-sd -Z` to extract the exact TXT record from a locally-visible printer.
- All AirPrint-certified printers and servers must now support TLS 1.2 encryption; PIN release, per-user credentials, and accounting codes are supported via standard IPP attributes.

---
_Source: WWDC16 Session 725 page (abstract, transcript, and resource links)._
