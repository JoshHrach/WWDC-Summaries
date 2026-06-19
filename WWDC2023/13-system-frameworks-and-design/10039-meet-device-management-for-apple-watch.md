# Meet Device Management for Apple Watch
**WWDC23 · Session 10039** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10039/)

_Platforms:_ watchOS 10, iOS 17

## Overview
Organizations can now deploy and configure Apple Watch alongside their other Apple devices. This session introduces the new MDM enrollment flow for watchOS 10, explaining how Apple Watch enrolls into device management in tandem with its paired, supervised iPhone running iOS 17.

The enrollment flow leverages Declarative Device Management and a new Watch Enrollment configuration declaration. The pairing token mechanism ensures the MDM server can securely verify which iPhone and Apple Watch are paired, creating a tight trust relationship between the two devices.

Once enrolled, watchOS 10 supports the full suite of MDM protocol features including configuration payloads, restrictions, commands, and queries, as well as the entire Declarative Device Management stack including the proactive status channel.

## Key Topics

### Enrollment Flow
- Requires a managed, supervised iPhone running iOS 17
- A new `Watch Enrollment` configuration declaration is applied to the iPhone, signaling that any Watch pairing should trigger MDM enrollment
- Apple Watch must be new or freshly reset; existing paired Watches must be wiped to enroll
- MDM enrollment happens during the Watch pairing flow; the user must accept Remote Management on the Watch
- A secure pairing token flow confirms the iPhone–Watch relationship to the MDM server

### Pairing Token Security Flow
1. Watch sends a first enrollment request; server returns HTTP 403 with a `security-token`
2. Watch sends the security-token to the iPhone
3. iPhone sends a new `GetToken` CheckIn request (type `watchPairingToken`) including both device UDIDs and the security-token
4. Server generates and returns a signed pairing token
5. Watch re-sends enrollment request with the pairing token embedded in Machine Info; server verifies and returns the MDM profile

### Declarative Device Management on watchOS
- All declaration types supported in watchOS 10
- Activation predicates supported for multi-step management workflows
- Status channel supported for proactive updates from the Watch

### Configuration Payloads
- Wi-Fi, Cellular networking payloads
- Per-app VPN payload **[NEW]** for watchOS
- SCEP / ACME certificate payloads
- Passcode / password policy payload (restrictions applied to iPhone sync to paired Watch; reverse sync does not occur)
- Restrictions payload

### MDM Commands
- Remote passcode clear, device lock, device erase
- Remove MDM profile (triggers unpairing and full erase of Watch; does not affect paired iPhone's enrollment)
- If iPhone is unenrolled, the paired managed Watch is also unenrolled and reset

### App Deployment
- **Paired apps**: share data with an iPhone companion app but can function independently
- **Dependent apps**: require the companion iPhone app to be functional
- **Standalone watchOS apps**: no iOS companion; managed with App Install command sent only to Watch
- For paired/dependent apps: install to iPhone first, then send App Install to Watch; updates and removals require commands sent to both devices

## APIs & Frameworks

- **Declarative Device Management** – declaration-based MDM framework **[NEW on watchOS]**
- `Watch Enrollment` configuration declaration **[NEW]** – signals that Watch pairing should trigger MDM enrollment
  - `enrollmentURL` – endpoint delivering the MDM enrollment profile
  - `AnchorCertificateAssetReferences` – optional array of anchor certificates
- `GetToken` CheckIn message type **[NEW]** – requests secure tokens from the MDM server
  - `TokenServiceType` – identifies the token service (e.g., Watch pairing token)
  - `TokenParameters` – dictionary of service-specific parameters (Watch UDID, iPhone UDID, security-token)
- **MDM Protocol** – `DeviceManagement` framework; existing iOS payloads extended to watchOS
- Status channel (Declarative Device Management) – proactive status updates to MDM server
- `Authenticate` CheckIn messageType – signals enrollment completion on Watch
- Supported MDM payloads on watchOS: Wi-Fi, Cellular, Per-App VPN **[NEW]**, SCEP, ACME, Passcode Policy, Restrictions
- MDM commands on watchOS: `ClearPasscode`, `DeviceLock`, `EraseDevice`, `RemoveProfile`, `InstallApplication`, `RemoveApplication`

## Code Highlights

Watch Enrollment configuration declaration (JSON structure sent to iPhone):
```json
{
  "Type": "com.apple.configuration.watch.enrollment",
  "Payload": {
    "enrollmentURL": "https://mdm.example.com/watch/enroll",
    "AnchorCertificateAssetReferences": ["<asset-reference-identifier>"]
  }
}
```

HTTP 403 response from MDM server when pairing token is missing:
```json
{
  "ErrorCode": "com.apple.watch.pairing.token.missing",
  "Details": {
    "security-token": "<random-UUID-string>"
  }
}
```

## Takeaways
- Apple Watch management is paired-device management: iPhone and Watch must be treated as a unit, and MDM UIs should expose this relationship clearly.
- Use the Declarative Device Management status channel instead of polling MDM queries to preserve Watch battery life.
- Restrictions and passcode policies flow from iPhone to Watch; Watch-only policies do not sync back to iPhone.
- Removing the MDM profile from either device has cascading effects—removing from the Watch unpairs and erases it; removing from the iPhone also unenrolls and erases the Watch.

---
_Source: WWDC23 Session 10039 page (abstract, chapter summaries, code samples, and resource links)._
