# Discover Managed Device Attestation
**WWDC22 · Session 10143** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10143/)

_Platforms:_ iOS 16, iPadOS 16, tvOS 16

## Overview
Managed Device Attestation is a new security feature for enterprise and education deployments introduced in iOS 16, iPadOS 16, and tvOS 16. It enables devices to provide cryptographically signed, Apple-attested evidence of their identity and properties to MDM servers and other organizational infrastructure, supporting zero trust architecture by ensuring only legitimate, uncompromised devices can access organizational resources.

The feature has two implementation paths: an enhanced `DeviceInformation` MDM command that returns an attestation certificate chain for the device's properties, and an ACME (Automated Certificate Management Environment) profile payload that ties a Secure Enclave-bound private key to an Apple-attested certificate for use with MDM, VPN, Wi-Fi, Kerberos, and other services. Trust in the attestation requires only trusting the Secure Enclave and Apple's attestation servers, not Apple's entire software stack.

## Key Topics

### Zero Trust Architecture Context
- Traditional perimeter security (VPN/firewall) has limitations: cloud resources live outside the perimeter, threats can start inside, and attackers can spoof legitimate clients.
- Zero trust: each resource performs its own trust evaluation based on posture information (user identity, device identity, location, etc.).
- Managed Device Attestation improves the **device identity** component of posture.

### DeviceInformation Attestation
- MDM server sends a `DeviceInformation` command with the new `DevicePropertiesAttestation` query key **[NEW]**.
- Optional `DeviceAttestationNonce` key (up to 32 bytes) ensures attestation freshness.
- Device contacts Apple's attestation servers and returns a certificate chain (`DevicePropertiesAttestation` array of DER-encoded certs).
- Leaf certificate contains custom OIDs: serial number, UDID (omitted for User Enrollments), sepOS version, nonce.
- MDM server validates: cert chain roots to Apple CA (from Apple Private PKI Repository), nonce matches, OID values are evaluated.
- **Rate limit**: one new attestation per device every 7 days; cached attestations returned if nonce omitted or matches cached value.
- Best practice: monitor other `DeviceInformation` properties for changes (e.g., OS version), then request fresh attestation; add occasional random fresh requests.
- Failed attestation (network issue, Apple server issue, compromised hardware/software) should lower device trust score, not necessarily trigger immediate action.

### ACME Payload Attestation
- New `ACME` configuration profile payload **[NEW]** — instructs device to request a client certificate from an organization ACME server (RFC 8555).
- Device generates a hardware-bound key in the Secure Enclave using `ECSECPrimeRandom` 384-bit (highest security option).
- Device requests attestation from Apple using the ACME server's nonce (SHA-256 hashed before embedding).
- Device sends CSR + attestation chain + `ClientIdentifier` to ACME server.
- ACME server validates: `ClientIdentifier` is valid/unused, cert chain roots to Apple CA, public key in attestation matches CSR, nonce hash matches, remaining OIDs evaluated.
- ACME server issues client certificate from the organization CA.
- The same Secure Enclave key used for attestation, ACME certificate request, and subsequent TLS connections proves all actions are from the same device.
- Up to 10 ACME payloads with attestation per device simultaneously.
- Hardware-bound keys are NOT preserved across backup restores.
- Certificate can be referenced by other payloads: MDM, Wi-Fi, VPN, Kerberos, Safari.

### Threats Mitigated
- Compromised OS lying about device properties → Apple attests properties independently of the OS.
- Outdated attestation for changed properties → nonce enforces freshness.
- Device spoofing different device's identifiers to MDM → attestation tied to the TLS client identity.
- Private key extracted from legitimate device → Secure Enclave attestation proves key is hardware-bound and cannot be exported.
- Certificate request hijacking → attestation ties the request to the requesting device's identity.

## APIs & Frameworks

### MDM Protocol (Device Management)
- `DeviceInformation` MDM command — enhanced with attestation **[NEW]**
- `DevicePropertiesAttestation` query key **[NEW]**
- `DeviceAttestationNonce` request key (32-byte max) **[NEW]**
- `DevicePropertiesAttestation` response key — array of DER certificate chain **[NEW]**
- Custom OIDs in attestation leaf: serial number, UDID, sepOS version, nonce

### Configuration Profiles
- `ACME` profile payload type **[NEW]**
- `ClientIdentifier` — single-use ticket for certificate issuance
- `ECSECPrimeRandom` key type — required for hardware-bound keys (256 or 384-bit; 384-bit preferred)
- `HardwareBound` key property — specifies Secure Enclave binding

### Protocols & Standards
- ACME protocol — RFC 8555 (Automated Certificate Management Environment)
- `device-attest-01` ACME validation type — IETF draft extension for device attestation **[NEW]**
- X.509 certificates — used for attestation certificate chains
- Apple Private PKI Repository — source of Apple CA certificates for validation

## Code Highlights

```xml
<!-- DeviceInformation attestation request (MDM command plist) -->
<dict>
    <key>RequestType</key>
    <string>DeviceInformation</string>
    <key>Queries</key>
    <array>
        <string>DevicePropertiesAttestation</string>
    </array>
    <key>DeviceAttestationNonce</key>
    <data>bWFnaWMgd29yZHM6IHNxdWVhbWlzaCBvc3NpZnJhZ2U=</data>
</dict>

<!-- DeviceInformation attestation response -->
<key>DevicePropertiesAttestation</key>
<array>
    <data>MIIC0TCCAli...pIbnVw=</data>  <!-- Leaf certificate -->
    <data>MIICSTCCAc6...wjtGA==</data>  <!-- Intermediate certificate -->
</array>
```

## Takeaways
- Managed Device Attestation brings zero trust device identity to Apple managed devices — Apple's Secure Enclave and attestation servers cryptographically sign device properties, making them trustworthy even if the OS is compromised.
- Use `DeviceInformation` attestation to continuously verify device properties for MDM posture evaluation; use freshness nonces strategically (not at maximum rate) and trigger on property changes.
- Deploy ACME payload attestation for the strongest proof of device identity — the same Secure Enclave key in the attestation, the client certificate, and TLS connections proves all communications are from the same genuine device.
- At minimum, use an ACME payload for MDM client identity so the MDM server has cryptographic proof of which device it is managing.

---
_Source: WWDC22 Session 10143 page (abstract, chapter summaries, code samples, and resource links)._
