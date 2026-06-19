# Secure Login with iCloud Keychain Verification Codes
**WWDC21 · Session 10105** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10105/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12

## Overview
This session introduces the built-in TOTP (Time-Based One-Time Password) verification code generator added to iCloud Keychain in iOS 15 and macOS Monterey. The feature brings together on-device code generation, AutoFill, iCloud sync/backup, and a streamlined two-tap setup flow to make multi-factor authentication far less painful for users while improving security compared to SMS-delivered codes.

The session opens by comparing SMS delivery (vulnerable to carrier snooping and SIM-swapping, unreliable, costly) with TOTP on-device generation (defined in RFC 6238: secret key + time → short numeric code, no network required), then explains how iCloud Keychain closes the remaining gap — historically, TOTP setup required a separate authenticator app and QR code scanning across devices.

It closes by placing these improvements on the broader authentication spectrum and highlighting domain-bound SMS codes (iOS 14+) as a near-term hardening step for services still relying on SMS.

## Key Topics

### iCloud Keychain TOTP Generator
- New in iOS 15/macOS Monterey: time-based verification code generators built into the iCloud Keychain Password Manager.
- AutoFill works with generated codes, exactly like SMS codes — one tap to fill.
- Verification codes sync across all devices and are end-to-end encrypted in iCloud Keychain (accessible only to the account owner).
- Account recovery is improved: codes are backed up and survive device loss.

### Two-Tap Setup with `apple-otpauth:` URLs
- Standard `otpauth:` URLs encode the secret key, digit count, period, algorithm, and issuer (set to your domain name).
- Prefix the URL with `apple-` to create an `apple-otpauth:` URL that opens directly into iCloud Keychain setup.
- On web pages: add the URL as an `<a href>` link or button.
- In iOS/iPadOS apps: open the URL with `UIScene.open(_:)` or link via an `NSAttributedString` with `.link` attribute.
- Check availability with `UIApplication.shared.canOpenURL(_:)` before showing the setup button.
- On unsupported OS versions, hide the setup button and fall back to QR code.

### QR Code Setup via Safari
- Safari's on-device image analysis detects `otpauth:` URLs encoded in raster QR code images (JPG, PNG, GIF).
- When detected, Safari offers a context menu action to set up the code generator in iCloud Keychain.
- Use raster images (not SVG) for QR codes intended for this flow.

### AutoFill Annotation
- Annotate verification code text fields so AutoFill knows where to fill codes.
- SwiftUI: `.textContentType(.oneTimeCode)` modifier.
- UIKit: `UITextField.textContentType = .oneTimeCode`.
- AppKit: `NSTextField.contentType = .oneTimeCode`.
- Web: `<input autocomplete="one-time-code">`.

### Domain-Bound SMS Codes (iOS 14+)
- For services still sending codes over SMS, add a domain binding to the message text to resist phishing.
- Format: append `@domain.com #123456` (or similar) to the SMS message body.
- AutoFill will only offer the code when the browser/app's domain matches the bound domain.
- Works via Associated Domains / Universal Links; no additional configuration needed if those are already set up.

## APIs & Frameworks

- `iCloud Keychain` — TOTP verification code generator **[NEW in iOS 15/macOS Monterey]**
- `otpauth:` URL scheme — standard TOTP provisioning URL (RFC 6238)
- `apple-otpauth:` URL scheme **[NEW]** — deep-links directly into iCloud Keychain setup
- `UIApplication.shared.canOpenURL(_:)` — availability check for `apple-otpauth:` on iOS
- `UIWindowScene.open(_:options:completionHandler:)` — open `apple-otpauth:` URL from an app
- `NSAttributedString` `.link` attribute — link to `apple-otpauth:` URL in a text view
- `UITextField.textContentType` — set to `.oneTimeCode` for AutoFill support
- `NSTextField.contentType` — set to `.oneTimeCode` on macOS
- `.textContentType(.oneTimeCode)` — SwiftUI modifier
- HTML `autocomplete="one-time-code"` attribute — web AutoFill annotation
- `AuthenticationServices` framework — Sign in with Apple (referenced as best-in-class baseline)
- `ASAuthorizationAppleIDCredential` — Sign in with Apple credential
- Domain-bound SMS codes — `@domain.com #CODE` format in SMS body (iOS 14+)
- `WebAuthentication` (WebAuthn) — referenced as the passwordless future (preview in iOS 15/macOS Monterey)

## Code Highlights

Opening the iCloud Keychain TOTP setup URL in an app:
```swift
// Check support before showing the setup button
if UIApplication.shared.canOpenURL(URL(string: "apple-otpauth://")!) {
    // Show setup button
}
// On button tap:
windowScene.open(URL(string: "apple-otpauth://totp/Example:user@example.com?secret=BASE32SECRET&issuer=example.com")!, options: nil)
```

Annotating a verification code field in SwiftUI:
```swift
TextField("Verification Code", text: $code)
    .textContentType(.oneTimeCode)
```

Web annotation:
```html
<input type="text" autocomplete="one-time-code" inputmode="numeric">
```

## Takeaways
- Adding `apple-otpauth:` links to TOTP setup pages gives iOS 15 users a two-tap setup experience and dramatically increases TOTP adoption over SMS.
- Annotate verification code fields with `.oneTimeCode` content type so AutoFill can fill both SMS codes and generated TOTP codes with one tap.
- Domain-bound SMS codes are a low-effort security improvement that resists phishing for services that must continue to send SMS codes.
- iCloud Keychain's end-to-end encryption and cross-device sync make on-device TOTP strictly superior to SMS from both a security and reliability standpoint.

---
_Source: WWDC21 Session 10105 page (abstract, chapter summaries, and resource links)._
