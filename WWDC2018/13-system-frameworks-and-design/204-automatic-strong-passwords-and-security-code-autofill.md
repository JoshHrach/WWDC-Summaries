# Automatic Strong Passwords and Security Code AutoFill
**WWDC18 · Session 204** · [Watch](https://developer.apple.com/videos/play/wwdc2018/204/)

_Platforms:_ iOS 12, macOS Mojave 10.14, tvOS 12

## Overview
iOS 12 and macOS Mojave introduce a trio of AutoFill improvements that significantly reduce friction in account creation and sign-in: Automatic Strong Passwords, Security Code AutoFill, and the new `ASWebAuthenticationSession` API for federated authentication. Together, these features make it easy for users to always use strong, unique passwords without thinking about them.

Automatic Strong Passwords generates a 20-character password (uppercase, lowercase, digits, hyphen; >71 bits of entropy) and inserts it automatically when a new-password field gets focus. The iCloud Keychain saves the credential automatically. Security Code AutoFill surfaces SMS one-time codes in the QuickType bar so users fill them with a single tap. `ASWebAuthenticationSession`, which replaces `SFAuthenticationSession`, handles OAuth/federated login flows by sharing Safari's cookie storage and supporting AutoFill.

## Key Topics

### Password AutoFill Prerequisites
- App-domain association via Associated Domains entitlement + `apple-app-site-association` file (as for Universal Links/Handoff) — required for both filling and saving.
- Tag fields with `UITextContentType`: `.username` for username/email, `.password` for existing-account password fields.
- `WKWebView` supports Password AutoFill since iOS 11.3.
- Third-party password manager Credential Provider Extensions can fill credentials on iOS 12.
- iOS 12 now prompts to **save** credentials automatically after sign-in (requires association and correct field tagging).

### Saving Password Best Practices
- Remove username/password fields from the view hierarchy on sign-in (e.g., dismiss the view controller) so AutoFill detects the sign-in event.
- Only clear field values after they are removed from the hierarchy.
- Use `webcredentials` associated domain service to override which domain the credential is saved to.

### Automatic Strong Passwords (iOS 12, New)
- Tag new-password fields with `UITextContentType.newPassword` **[NEW]** — AutoFill inserts a strong generated password automatically.
- Tag confirm-password fields with `UITextContentType.newPassword` as well.
- Use unique `UITextField` instances for username and password fields (not reused cell-based instances) for reliable value reading.
- Works with change-password forms if both username (can be read-only) and new-password fields are on the same screen.
- Custom password rules via `UITextInputPasswordRules` and the `passwordRules` property on `UITextField` **[NEW]**.
- Use the Password Rules Validation Tool at `developer.apple.com/password-rules` to author and test rules.

### Security Code AutoFill (iOS 12, New)
- Tag one-time code fields with `UITextContentType.oneTimeCode` **[NEW]**.
- Do not use bespoke keyboard UIs or custom `inputView` — this blocks AutoFill from injecting the code.
- Craft SMS messages with trigger words like "code" or "passcode" near the numeric code; iOS uses data-detector heuristics.
- Verify SMS triggers by texting yourself: if the code is underlined and tapping offers "Copy Code", the heuristics fired correctly.
- Available in all supported iOS/macOS locales.
- Safari on Mac: text-message forwarding relays incoming codes from iPhone → Mac; fills with one click.
- Web equivalent: `autocomplete="one-time-code"` input attribute **[NEW]**.

### ASWebAuthenticationSession (iOS 12, New)
- Replaces `SFAuthenticationSession` for OAuth/federated login.
- Shares Safari's cookie storage with the session (with explicit user consent alert).
- Supports Password AutoFill and Security Code AutoFill within the presented auth view controller.
- Block-based API; caller must hold a strong reference to the session object.
- `start()` is non-blocking; `cancel()` cancels an in-flight session.

### iCloud Keychain Manager Improvements (iOS 12 / macOS Mojave)
- Siri can look up passwords ("Show me my password for X").
- AirDrop password sharing to contacts.
- Redesigned password list UI on iOS and macOS.
- Password Reuse Auditing: warns when the same password is used on multiple sites; links to change-password page.
- tvOS 12: AutoFill from a nearby iOS device (see "What's New in tvOS 12").

## APIs & Frameworks

**UIKit**
- `UITextContentType.username` — tag username/email fields
- `UITextContentType.password` — tag existing-password fields
- `UITextContentType.newPassword` **[NEW]** — tag new/confirm-password fields; triggers Automatic Strong Passwords
- `UITextContentType.oneTimeCode` **[NEW]** — tag SMS one-time code fields; triggers Security Code AutoFill
- `UITextField.textContentType` — property accepting `UITextContentType`
- `UITextField.passwordRules` — accepts `UITextInputPasswordRules` **[NEW]**
- `UITextInputPasswordRules` **[NEW]** — initialized with a password rules descriptor string

**AuthenticationServices (new framework, iOS 12)**
- `ASWebAuthenticationSession` **[NEW]** — replaces `SFAuthenticationSession`
  - `init(url:callbackURLScheme:completionHandler:)`
  - `start() -> Bool`
  - `cancel()`

**Security / Associated Domains**
- `com.apple.developer.associated-domains` entitlement with `webcredentials:<domain>` entries
- `apple-app-site-association` JSON file served from the domain
- `SecAddSharedWebCredential` — still needed for web-view-based login screens

**Web / HTML**
- `autocomplete="username"` — username fields
- `autocomplete="current-password"` — existing-password fields
- `autocomplete="new-password"` — new/confirm-password fields
- `autocomplete="one-time-code"` **[NEW]** — one-time SMS code fields

**Developer Tool**
- Password Rules Validation Tool — `https://developer.apple.com/password-rules` **[NEW]**

## Code Highlights

Tagging a new-password field with custom password rules:
```swift
let rulesDescriptor = "allowed: upper, lower, digit; required: [$]; minlength: 20;"
passwordTextField.textContentType = .newPassword
passwordTextField.passwordRules = UITextInputPasswordRules(descriptor: rulesDescriptor)
```

Federated authentication with ASWebAuthenticationSession:
```swift
import AuthenticationServices

let oauthURL = URL(string: "https://example.com/oauth/authorize?client_id=…")!
let session = ASWebAuthenticationSession(url: oauthURL, callbackURLScheme: "myapp") { callbackURL, error in
    guard let callbackURL = callbackURL, error == nil else { return }
    // handle OAuth callback
}
self.authSession = session  // retain strong reference
session.start()
```

## Takeaways
- Tag every auth-related text field with the correct `UITextContentType` — `.newPassword` enables Automatic Strong Passwords, `.oneTimeCode` enables Security Code AutoFill, and both require almost no additional code.
- Associated Domains are the gateway to both filling and saving passwords; without them, Automatic Strong Passwords cannot save the credential.
- Use `ASWebAuthenticationSession` (replacing `SFAuthenticationSession`) for all OAuth/federated flows — it gets cookie sharing, Password AutoFill, and Security Code AutoFill for free.
- Avoid custom keyboard UIs for security code fields; they prevent AutoFill from injecting the code and harm accessibility.

---
_Source: WWDC18 Session 204 page (abstract, full transcript, and resource links)._
