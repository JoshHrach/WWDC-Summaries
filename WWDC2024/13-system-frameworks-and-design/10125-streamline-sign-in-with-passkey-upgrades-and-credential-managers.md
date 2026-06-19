# Streamline Sign-in with Passkey Upgrades and Credential Managers
**WWDC24 · Session 10125** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10125/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia 15, visionOS 2

## Overview
This session covers three interconnected improvements to Apple's sign-in ecosystem: automatic passkey upgrades for existing password-based accounts, expanded capabilities for third-party credential manager apps, and optimizing app/website appearance in the new Passwords app.

Automatic passkey upgrades allow apps and websites to silently create a passkey immediately after a successful password sign-in, without interrupting the user with an upsell dialog. The system and active credential manager apply their own preconditions (device passcode set, credential manager was recently used for the same account, etc.) and either silently register the passkey—notifying the user with a system notification—or return an error with no UI shown.

The new Passwords app, available on macOS Sequoia, iOS 18, and visionOS 2, surfaces passkeys, passwords, and verification codes in a unified interface. Developers can control how their service appears by adopting OpenGraph metadata, high-resolution icons, the well-known change-password URL, and `otpauth:` links for one-tap verification code setup.

## Key Topics

### Automatic Passkey Upgrades
Pass `requestStyle: .conditional` to `createCredentialRegistrationRequest(...)` to request a silent, automatic upgrade. The system checks preconditions (credential manager present, device set up for passkeys, non-private browsing on the web) and passes the request to the credential manager, which checks whether it just filled credentials for the same account. Success delivers a new passkey and a system notification; failure returns an error silently.

### The Passkey Journey
Phishing resistance requires eliminating all phishable factors (passwords, SMS, email OTP, push, TOTP). The first step is adding a passkey as an alternative; the ultimate goal is removing all phishable factors. Automatic upgrades help accelerate that first step.

### Credential Manager Improvements
Credential managers can now support automatic passkey upgrades, fill time-based verification codes, and fill usernames/passwords/OTPs into any text field. New Info.plist keys in `ASCredentialProviderExtensionCapabilities` declare each capability. Up to three credential manager apps can now be active simultaneously with AutoFill.

### The New Passwords App
The Passwords app (macOS Sequoia, iOS 18, visionOS 2) merges passkeys and passwords for the same account into a single entry. OpenGraph `og:site_name` metadata controls how a website's name appears. A Security section flags weak, reused, or leaked passwords and links to the well-known change-password URL. A menu bar item on macOS provides quick access.

### One-Tap Verification Code Setup
Offering an `otpauth:` URI alongside the standard QR code lets users set up TOTP codes in one tap on-device, opening in the default verification code app (the Passwords app by default).

## APIs & Frameworks

- `ASAuthorizationPlatformPublicKeyCredentialProvider` — creates passkey registration/assertion requests
- `createCredentialRegistrationRequest(challenge:name:userID:requestStyle:)` — **[NEW]** `requestStyle` parameter
- `ASAuthorizationPlatformPublicKeyCredentialRegistrationRequest.RequestStyle.conditional` **[NEW]** — silent automatic upgrade style
- `ASAuthorizationController.performRequest(_:)` — executes the registration request
- `ASCredentialProviderExtensionCapabilities` **[NEW]** — Info.plist dictionary for credential manager capabilities
  - `ProvidesPasswords` — declares password autofill support
  - `ProvidesPasskeys` — declares passkey support
  - `SupportsConditionalPasskeyRegistration` **[NEW]** — opt-in to automatic passkey upgrade flow
  - `ProvidesOneTimeCodes` **[NEW]** — declares TOTP/OTP fill support
  - `ProvidesTextToInsert` **[NEW]** — declares ability to fill any text field
- `PublicKeyCredential.getClientCapabilities()` — Web API: check browser support for conditional create
- `navigator.credentials.create(options)` — Web API: standard passkey registration
- `mediation: "conditional"` **[NEW]** — Web API option for automatic passkey upgrade
- `Authentication Services` framework — umbrella framework for passkeys, passwords, credential providers
- `otpauth:` URI scheme — standard for TOTP setup links
- Well-known URL for changing passwords (`/.well-known/change-password`) — surfaces Change Password button in Passwords app
- OpenGraph `og:site_name` meta tag — controls service name display in Passwords app

## Code Highlights

Traditional upsell-based passkey registration (before):
```swift
let request = provider.createCredentialRegistrationRequest(
    challenge: try await fetchChallenge(),
    name: userInfo.user,
    userID: userInfo.accountID)
// ... show upsell UI, then performRequest
```

Automatic passkey upgrade using `.conditional` style (after):
```swift
let request = provider.createCredentialRegistrationRequest(
    challenge: try await fetchChallenge(),
    name: userInfo.user,
    userID: userInfo.accountID,
    requestStyle: .conditional)

do {
    let passkey = try await authorizationController.performRequest(request)
    // Save new passkey to backend; system shows silent notification
} catch { /* no UI shown; try again next time or show upsell */ }
```

Web conditional passkey creation:
```javascript
let capabilities = await PublicKeyCredential.getClientCapabilities();
if (!capabilities.conditionalCreate) { return; }
await navigator.credentials.create({
    publicKey: { rp: { … }, user: { name: userInfo.user, … }, … },
    mediation: "conditional"
});
```

Credential manager Info.plist capabilities:
```xml
<key>ASCredentialProviderExtensionCapabilities</key>
<dict>
    <key>ProvidesPasswords</key><true/>
    <key>ProvidesPasskeys</key><true/>
    <key>SupportsConditionalPasskeyRegistration</key><true/>
    <key>ProvidesOneTimeCodes</key><true/>
    <key>ProvidesTextToInsert</key><true/>
</dict>
```

## Takeaways

- Use `requestStyle: .conditional` on `ASAuthorizationPlatformPublicKeyCredentialRegistrationRequest` to silently upgrade password accounts to passkeys immediately after sign-in, with no upsell UI needed.
- Credential managers can now declare support for automatic passkey upgrades, TOTP fill, and any-text-field fill via new `ASCredentialProviderExtensionCapabilities` Info.plist keys; up to three credential managers can be active simultaneously.
- Adopt OpenGraph `og:site_name`, the well-known change-password URL, and high-resolution app icons to ensure your service looks great in the new Passwords app on iOS 18, macOS Sequoia, and visionOS 2.
- Offer an `otpauth:` URI alongside TOTP QR codes for a one-tap verification code setup experience that works with the Passwords app and any third-party credential manager.

---
_Source: WWDC24 Session 10125 page (abstract, chapter summaries, code samples, and resource links)._
