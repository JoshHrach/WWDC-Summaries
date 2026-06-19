# AutoFill Everywhere
**WWDC20 · Session 10115** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10115/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11

## Overview
This session covers how to adopt AutoFill across all types of text fields in iOS and macOS apps. By annotating text fields with the correct `UITextContentType` (UIKit/SwiftUI) or the new `NSTextContentType` (AppKit), developers enable the system's QuickType keyboard bar to surface contextually appropriate suggestions — recent addresses, contact emails and phone numbers, saved passwords, one-time security codes, and strong new passwords — without the app ever having to request permission to read Contacts or handle credentials directly.

A key privacy point: the new Contact AutoFill in iOS 14 lets the keyboard suggest names, email addresses, and phone numbers from the user's Contacts while the app receives nothing until the user explicitly taps a suggestion. This eliminates the need to request Contacts permission or build custom contact-suggestion UI in many common cases. macOS Big Sur extends AutoFill for the first time to AppKit apps, adding password and security code suggestions, plus support for third-party password manager apps as data sources.

## Key Topics
- **`UITextContentType`** — Setting this property on any `UITextField` or `UITextView` is the single action needed to enable all AutoFill variants; the system infers which suggestions to show based on the semantic type.
- **Address AutoFill** — Content types: `.fullStreetAddress`, `.streetAddressLine1`, `.streetAddressLine2`, `.addressCity`, `.addressCityAndState`, `.addressState`, `.countryName`, `.postalCode`, `.sublocality`; the system proactively suggests relevant locations (recent searches, calendar events, home/work addresses) in the QuickType bar.
- **Contact AutoFill (iOS 14)** **[NEW]** — Setting `.emailAddress` or `.telephoneNumber` now triggers contact suggestions from the QuickType bar; no Contacts permission requested; nothing shared with the app until user taps the suggestion.
- **Password AutoFill** — `.username` + `.password` pair triggers iCloud Keychain / third-party password manager suggestion; requires associated domains (AASA file) to scope suggestions to the app's domain.
- **Security Code AutoFill** — `.oneTimeCode` triggers SMS or email one-time code suggestion; also works for Catalyst apps on macOS Big Sur.
- **Automatic Strong Passwords** — `.newPassword` on a new password field triggers the system to suggest and save a strong password to iCloud Keychain; requires associated domains.
- **`NSTextContentType` (macOS Big Sur)** **[NEW]** — AppKit equivalent of `UITextContentType`; supported values: `.username`, `.password`, `.oneTimeCode`; set on `NSTextField` or `NSSecureTextField`; third-party password managers can supply data.
- **Privacy-first design** — Prefer Contact AutoFill (`.emailAddress`, `.telephoneNumber`) and `CNContactPickerViewController` over requesting full Contacts permission; the app receives only what the user explicitly selects.

## APIs & Frameworks

### UIKit (iOS / iPadOS / Mac Catalyst)
- **`UITextField.textContentType`** / **`UITextView.textContentType`** — `UITextContentType`; existing property; key values for 2020:
  - Address: `.fullStreetAddress`, `.streetAddressLine1`, `.streetAddressLine2`, `.addressCity`, `.addressCityAndState`, `.addressState`, `.countryName`, `.postalCode`, `.sublocality`
  - Contact **[NEW behavior in iOS 14]**: `.emailAddress`, `.telephoneNumber` — now surfaces contact suggestions in QuickType
  - Credentials: `.username`, `.password`, `.newPassword`
  - Security: `.oneTimeCode`
- **`CNContactPickerViewController`** — Existing; presents system contact picker without requiring Contacts permission; only the selected contact data is returned to the app

### AppKit (macOS Big Sur)
- **`NSTextField.contentType`** **[NEW]** — `NSTextContentType?`; new property on `NSTextField`
- **`NSSecureTextField.contentType`** **[NEW]** — `NSTextContentType?`; new property on `NSSecureTextField`
- **`NSTextContentType`** **[NEW]** — New type mirroring `UITextContentType`; currently supported: `.username`, `.password`, `.oneTimeCode`

### SwiftUI
- **`.textContentType(_:)`** — View modifier on `TextField` / `SecureField`; takes `UITextContentType` (iOS) or `NSTextContentType` (macOS)

## Code Highlights

Address AutoFill in a navigation app:
```swift
let streetAddressTextField = UITextField()
streetAddressTextField.textContentType = .fullStreetAddress
// system suggests recent addresses, calendar event locations, home/work
```

Contact AutoFill (iOS 14) — no permission required:
```swift
let emailTextField = UITextField()
emailTextField.textContentType = .emailAddress   // suggests contact emails

let phoneTextField = UITextField()
phoneTextField.textContentType = .telephoneNumber // suggests contact phone numbers
```

Password AutoFill pair:
```swift
let userTextField = UITextField()
userTextField.textContentType = .username

let passwordTextField = UITextField()
passwordTextField.textContentType = .password
```

Automatic Strong Password on account creation:
```swift
let newPasswordTextField = UITextField()
newPasswordTextField.textContentType = .newPassword
// system generates and saves a strong password to iCloud Keychain
```

AppKit AutoFill (macOS Big Sur) — new `NSTextContentType`:
```swift
let usernameTextField = NSTextField()
usernameTextField.contentType = .username

let passwordField = NSSecureTextField()
passwordField.contentType = .password

let securityCodeTextField = NSTextField()
securityCodeTextField.contentType = .oneTimeCode
```

## Takeaways
- Tag every text field with the most precise `UITextContentType` that matches its purpose — this single property enables address, contact, password, and security-code AutoFill with no additional code.
- The new iOS 14 Contact AutoFill for `.emailAddress` and `.telephoneNumber` fields surfaces contact suggestions without ever requesting Contacts permission; adopt it before adding a Contacts permission request.
- `NSTextContentType` on `NSTextField`/`NSSecureTextField` brings password and security-code AutoFill to native AppKit apps for the first time in macOS Big Sur, with full third-party password manager support.
- For app login flows, associate the app with your domain (AASA) to scope password suggestions; for new account creation, use `.newPassword` to trigger Automatic Strong Passwords.

---
_Source: WWDC20 Session 10115 page (abstract, transcript, and code samples)._
