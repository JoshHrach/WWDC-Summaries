# Privacy and Your App
**WWDC15 · Session 703** · [Watch](https://developer.apple.com/videos/play/wwdc2015/703/)

_Platforms:_ iOS 9, OS X El Capitan 10.11, watchOS 2

## Overview
This session from Apple's Product Security and Privacy team covers new privacy developments across iOS 9, OS X El Capitan, and watchOS 2, along with best practices for data collection, retention, identifiers, and user consent. Presenters Katie Skinner and Jason Novak frame privacy as a fundamental human right that developers share the responsibility to protect.

The session covers platform changes that restrict app behavior (MAC address randomization expansion, `canOpenURL` scheme declarations, `sysctl` sandbox tightening, OS X cookie isolation), new security requirements (App Transport Security), privacy-preserving identifier patterns, watchOS 2 privacy architecture, and existing technologies (data protection classes, Touch ID, Apple Pay) that developers should leverage rather than implement themselves.

A significant portion focuses on iOS 9's Spotlight/Search privacy design, public indexing with NSUserActivity, and Core Spotlight data management practices.

## Key Topics

### Privacy Principles
- Only collect data you actively use; define a retention policy and delete data when its purpose is served.
- All stored data makes you a more valuable target — practice data minimization (aggregation, de-resolution, on-device processing).
- Protect data in transit (App Transport Security), at rest (data protection classes, server-side encryption), and limit what is sent off-device at all.
- Be transparent: give users purpose strings, let them inspect their data, and provide reset/delete controls.

### iOS 9 Platform Changes
- **MAC address randomization**: Extended to more Wi-Fi scan types; apps cannot rely on MAC address before authentication — test hardware integrations on iOS 9.
- **`canOpenURL` / `LSApplicationQueriesSchemes`** **[NEW]**: Apps must declare URL schemes they intend to query in `Info.plist` under `LSApplicationQueriesSchemes`. Undeclared schemes always return `false`. Apps linked before iOS 9 receive a 50-scheme grace limit (not reset on reboot). Recommended alternatives: extensions and Universal Links.
- **`sysctl` sandbox tightening**: `kern.proc`, `kern.procargs`, `kern.procargs2` no longer return data for processes other than the calling process. Apps cannot enumerate other running processes.
- **Content Blocking extensions** (Safari and SFSafariViewController): New extension point; block lists apply to both Safari and all `SFSafariViewController` instances. Developers should test with popular extensions.
- **NFC rewards card encryption** (Passbook/Wallet): Pass `.json` now includes an `nfc` dictionary with `message`, `encryptionPublicKey`, and `requiresAuthentication`; iOS encrypts the message automatically during contactless presentation.

### OS X El Capitan Changes
- **Cookie isolation**: Cookies are now process-local (previously shared across all apps and processes). No change for Mac App Store apps. Dashboard widgets and Web Clips require testing.

### watchOS 2 Privacy Architecture
- watchOS shares privacy settings with the paired iPhone — a single trust relationship applies to both devices.
- User privacy decisions (grant/deny) made on iPhone apply to all aspects of the Watch app (app, Glance, Complication).
- **Just-in-time prompts**: On watchOS, the prompt appears on the iPhone (larger screen + purpose string), not on the Watch itself. Watch app may receive an "unset" state if the user is away from iPhone; app can prompt again later.
- **Keychain on Watch** (new in watchOS 2): Apps can store secrets locally on the Watch.
- IDFV and IDFA live on the iPhone in watchOS 1; in watchOS 2, sync them from iPhone to Watch and keep them current.

### Identifiers Best Practices
- Ask: do you need an identifier at all? Server-side counters may suffice.
- Scope identifiers to purpose: session, user, or device installation — use the narrowest scope needed.
- Rotate identifiers to prevent long-term tracking correlation (e.g., rotate every 15 minutes for search sessions).
- Provided identifiers: `identifierForVendor` (resets when all apps from a team are uninstalled), `advertisingIdentifier` (user-resettable; always check `isAdvertisingTrackingEnabled` and never cache the value).
- Never build persistent identifiers using private APIs — App Store violation with consequences.
- Report aggregates, not raw data, to third-party partners; apply minimum user-count thresholds to avoid de-anonymization.

### App Transport Security (ATS) **[NEW]**
- Default: all connections through high-level networking APIs require TLS 1.2 with forward secrecy.
- Exceptions declared in `Info.plist` under `NSAppTransportSecurity` / `NSExceptionDomains`.
- Exception keys: `NSAllowsArbitraryLoads`, `NSExceptionAllowsInsecureHTTPLoads`, `NSExceptionMinimumTLSVersion`, `NSExceptionRequiresForwardSecrecy`, `NSRequiresCertificateTransparency`.

### Spotlight / App Search Privacy (iOS 9)
- **NSUserActivity** (extended for search in iOS 9): `eligibleForSearch = true` to index; `eligibleForPublicIndexing = true` for public content only. Set `expirationDate`. Default is NOT indexed.
- Public indexing: hash of the view is sent to Apple after many devices engage with it; threshold exceeded before full content is submitted.
- **Core Spotlight** (new in iOS 9): Index user content (mail, contacts, documents). Apply data protection class to index items. Keep index in sync: update on document change, delete on document delete. Use `CSSearchableIndex.default().deleteAllSearchableItems()` for bulk cleanup.

### Existing Technologies to Leverage
- **Touch ID** (`LocalAuthentication` / `LAContext`) — protect app access or in-app data.
- **Apple Pay** — eliminate storing credit card / personal payment data entirely.
- **iOS Data Protection classes**:
  - `NSFileProtectionNone` — always accessible (avoid).
  - `NSFileProtectionCompleteUntilFirstUserAuthentication` — default for all third-party app data since iOS 7; inaccessible on boot, accessible after first unlock.
  - `NSFileProtectionCompleteUnlessOpen` — writable when locked, readable only when unlocked.
  - `NSFileProtectionComplete` — only accessible when device is unlocked (for most sensitive data: health, financial).
- **CloudKit** — built-in server-side encryption at rest.
- **Privacy policies** — required for HealthKit-linked apps; submit URL in App Store Connect.

## APIs & Frameworks

- `LSApplicationQueriesSchemes` (Info.plist key) **[NEW]** — declare URL schemes for `canOpenURL`
- `NSAppTransportSecurity` / `NSExceptionDomains` (Info.plist) **[NEW]** — ATS exception configuration
- `NSUserActivity` — `eligibleForSearch`, `eligibleForPublicIndexing`, `expirationDate` **[NEW in iOS 9]**
- `CoreSpotlight` framework **[NEW]**
  - `CSSearchableItem`, `CSSearchableItemAttributeSet`
  - `CSSearchableIndex.default().indexSearchableItems(_:completionHandler:)`
  - `CSSearchableIndex.default().deleteSearchableItems(withIdentifiers:completionHandler:)`
  - `CSSearchableIndex.default().deleteAllSearchableItems(completionHandler:)` **[NEW]**
- `ASIdentifierManager` — `advertisingIdentifier`, `isAdvertisingTrackingEnabled`
- `UIDevice.identifierForVendor` — IDFV
- `NSFileProtectionType` constants: `.complete`, `.completeUnlessOpen`, `.completeUntilFirstUserAuthentication`, `.none`
- `FileManager` file attribute `NSFileProtectionKey`
- `LAContext` (LocalAuthentication) — Touch ID / biometric authentication
- `PassKit` / Wallet `nfc` dictionary in pass.json **[NEW]** — NFC rewards card payload encryption
- `WKInterfaceController` / watchOS privacy: shared settings from paired iPhone
- `SecItemAdd` / Keychain on Watch **[NEW in watchOS 2]**
- `canOpenURL(_:)` — unchanged API; behavior gated by `LSApplicationQueriesSchemes`
- Universal Links (`apple-app-site-association`) — recommended replacement for URL scheme querying
- `CloudKit` — `CKContainer`, `CKRecord` with server-side encryption at rest

## Code Highlights

Declare URL schemes for `canOpenURL` in Info.plist:
```xml
<key>LSApplicationQueriesSchemes</key>
<array>
    <string>twitter</string>
    <string>instagram</string>
</array>
```

ATS exception for a specific insecure domain:
```xml
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSExceptionDomains</key>
    <dict>
        <key>example.com</key>
        <dict>
            <key>NSExceptionAllowsInsecureHTTPLoads</key>
            <true/>
        </dict>
    </dict>
</dict>
```

Mark an NSUserActivity as searchable:
```swift
let activity = NSUserActivity(activityType: "com.myapp.recipe")
activity.title = "Classic Poutine"
activity.eligibleForSearch = true
activity.expirationDate = Date(timeIntervalSinceNow: 30 * 24 * 3600)
self.userActivity = activity
activity.becomeCurrent()
```

Set data protection on a file:
```swift
try data.write(to: fileURL, options: .completeFileProtection)
```

## Takeaways
- The `canOpenURL` / `LSApplicationQueriesSchemes` change is a breaking API behavioral change for apps linked against iOS 9 — audit all `canOpenURL` call sites and add the `Info.plist` key.
- App Transport Security is on by default in iOS 9; any HTTP or weak-TLS connections will fail without explicit exceptions in `Info.plist`.
- On watchOS 2, privacy decisions are always made on iPhone and apply uniformly across the app, Glance, and Complication — design your permission flows assuming the user might not have the Phone nearby.
- Rotating identifiers and reporting aggregates (not raw data) to partners are the key engineering levers for protecting user privacy while still meeting analytics and advertising needs.

---
_Source: WWDC15 Session 703 page (abstract, chapter summaries, code samples, and resource links)._
