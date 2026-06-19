# Deploy Passkeys at Work
**WWDC23 · Session 10263** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10263/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14

## Overview
This session covers enterprise deployment of passkeys in managed Apple device environments. It addresses three enterprise-specific requirements that go beyond the consumer passkey experience covered in "Meet Passkeys" (WWDC22): managing which Apple IDs hold passkeys (Managed Apple IDs now support iCloud Keychain), controlling which devices passkeys sync to (new Access Management controls in Apple Business Manager / Apple School Manager), and proving to relying parties that passkeys were created on managed devices (new WebAuthn enterprise attestation via Declarative Device Management).

The session also makes a security case for passkeys in enterprise by quantifying phishing click rates and showing that passkeys eliminate phishing, server credential theft, and 2FA bypass attacks simultaneously—without sacrificing user experience.

## Key Topics

### Why Passkeys Matter for Enterprise Security
Three major attack categories eliminated by passkeys:

- **Phishing**: 2.9% of employees click phishing links (steady year over year, per 2022 Verizon DBIR). At 1,000 employees, ~29 accounts are phishable. Passkeys are cryptographically bound to the specific website/app; they cannot be used on a phishing site.
- **Credential theft from servers**: 40% of 2022 breaches involved stolen credentials (DBIR). Password hashes stolen in a server breach can be cracked and reused across services. With passkeys, only a public key is stored server-side—nothing worth stealing.
- **2FA bypass**: SMS codes and TOTP are phishable exactly like passwords. Push notification 2FA is susceptible to "push fatigue" attacks (approve a stream of unsolicited prompts). Passkeys replace the broken primary factor rather than patching it.

### Enterprise Requirements and Solutions

| Requirement | Solution |
|---|---|
| Manage which Apple ID stores passkeys | Managed Apple IDs now support iCloud Keychain (iOS 17, iPadOS 17, macOS Sonoma) |
| Restrict passkey sync to managed devices | Access Management controls in Apple Business Manager / Apple School Manager |
| Ensure passkeys for work aren't shared | Passkeys in Managed Apple ID iCloud Keychain cannot be shared |
| Prove passkey created on a managed device | New DDM passkey attestation configuration |

### Managed Apple IDs and iCloud Keychain (New in iOS/iPadOS 17, macOS Sonoma)
Managed Apple IDs—owned and managed by the organization in Apple Business Manager or Apple School Manager—now support iCloud Keychain. Passkeys saved to the Managed Apple ID iCloud Keychain:
- Sync across all devices signed in with that Managed Apple ID.
- Cannot be AirDropped or otherwise shared with other users.
- Are under organizational control because IT manages the Apple ID account.

### Access Management Controls (New in ABM/ASM)
Two independent controls in Apple Business Manager / Apple School Manager:

**Managed Apple ID sign-in restriction**: Choose which devices employees can sign into with their Managed Apple ID:
- Any device (default)
- Managed devices only (for BYOD environments)
- Managed supervised devices only (highest security; org-provisioned devices)

**iCloud content sync restriction**: Choose which devices can sync iCloud content (including passkeys in iCloud Keychain):
- Any device
- Managed devices only
- Managed supervised devices only

MDM server vendors must implement support; works out of the box with Apple Business Essentials.

### Passkey Attestation via Declarative Device Management (New)
The new `com.apple.configuration.security.passkey.attestation` DDM configuration enables WebAuthn enterprise attestation for specific relying parties:

**Flow**:
1. MDM server sends the attestation configuration and an identity asset (certificate from corporate CA) to the device.
2. The certificate is provisioned into the device keychain.
3. When the user visits a covered relying party and creates a passkey, the device generates the passkey and attests it using the provisioned identity certificate.
4. The relying party verifies the attestation by checking that the device certificate chains back to the organization's CA.
5. The passkey is stored in the Managed Apple ID's iCloud Keychain.

**Relying party verification**: Check for Apple device AAGUID (`dd4ec289-e01d-41c9-bb89-70fa845d4bf2`), then verify the `x5c` certificate chain in the packed attestation statement against the organization's CA certificate. Attestation is validated only at creation time; subsequent authentications use the passkey directly.

Note: this is **not** hardware attestation. For hardware attestation, see Managed Device Attestation (session 10040).

## APIs & Frameworks

**Declarative Device Management (New)**
- `com.apple.configuration.security.passkey.attestation` **[NEW]** — DDM configuration for WebAuthn enterprise passkey attestation
  - `AttestationIdentityAssetReference` — reference to identity asset (corporate CA certificate)
  - `RelyingParties` — array of domain strings where attestation applies

**WebAuthn / FIDO2 (Relying Party Side)**
- Packed attestation statement format — `fmt: "packed"`, `alg: -7` (ES256), `sig: bytes`, `x5c: [attestnCert, caCert]`
- Apple device AAGUID: `dd4ec289-e01d-41c9-bb89-70fa845d4bf2`
- Standard WebAuthn registration (`navigator.credentials.create()`) on the web side; enterprise attestation is opt-in via relying party configuration

**Apple Business Manager / Apple School Manager (New)**
- Managed Apple ID sign-in restriction: any device / managed devices only / managed supervised devices only
- iCloud content sync restriction: any device / managed devices only / managed supervised devices only

## Code Highlights

DDM passkey attestation configuration (JSON payload):
```json
{
    "Type": "com.apple.configuration.security.passkey.attestation",
    "Identifier": "B1DC0125-D380-433C-913A-89D98D68BA9C",
    "ServerToken": "8EAB1785-6FC4-4B4D-BD63-1D1D2A085106",
    "Payload": {
        "AttestationIdentityAssetReference": "88999A94-B8D6-481A-8323-BF2F029F4EF9",
        "RelyingParties": [
            "www.example.com"
        ]
    }
}
```

WebAuthn packed attestation statement (relying party verification):
```json
{
    "fmt": "packed",
    "attStmt": {
        "alg": -7,
        "sig": "<bytes>",
        "x5c": ["<attestnCert: bytes>", "<caCert: bytes>"]
    },
    "authData": {
        "attestedCredentialData": {
            "aaguid": "dd4ec289-e01d-41c9-bb89-70fa845d4bf2"
        }
    }
}
```

## Resources
- [Passkeys overview](https://developer.apple.com/passkeys/)
- Related: "Meet passkeys" (WWDC22 10092)
- Related: "Do more with Managed Apple IDs" (WWDC23 10254)
- Related: "Explore advances in declarative device management" (WWDC23 10041)
- Related: "What's new in managing Apple devices" (WWDC23 10040)

## Takeaways
- Managed Apple IDs now support iCloud Keychain, making passkeys viable in enterprise environments where IT must control credential storage.
- The two Access Management controls in ABM/ASM (Managed Apple ID sign-in restriction and iCloud sync restriction) together ensure passkeys sync only to devices the organization manages.
- The DDM passkey attestation configuration is the key mechanism for proving to relying parties that passkey creation occurred on an org-managed device; relying parties should validate the `x5c` certificate chain against the org's CA.
- Passkeys are not just another 2FA layer — they replace a fundamentally broken primary factor (passwords) and are architecturally resistant to phishing, credential theft, and 2FA bypass without any sacrifice in user experience.

---
_Source: WWDC23 Session 10263 page (abstract, transcript, chapter summaries, code samples, and resource links)._
