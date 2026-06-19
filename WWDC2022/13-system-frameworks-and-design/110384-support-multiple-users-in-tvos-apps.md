# Support Multiple Users in tvOS Apps
**WWDC22 · Session 110384** · [Watch](https://developer.apple.com/videos/play/wwdc2022/110384/)

_Platforms:_ tvOS 16

## Overview
tvOS 16 makes it significantly easier to support multiple users in apps running on a shared Apple TV. With a single capability checkbox — "Runs as Current User" — apps can partition their data per user, giving each person on the household device the same personalized experience they would have on their own iPhone.

A new user-independent Keychain API complements this entitlement, allowing a single signed-in account's credentials to remain accessible to all household users while every other piece of user data (UserDefaults, files, etc.) is siloed per user. Together these two primitives let streaming apps skip the profile picker on every user switch, jumping straight to personalized content without requiring each person to re-authenticate.

tvOS 16 also deprecates the manual TVUserManager profile-mapping APIs, since the OS now handles user-to-data mapping automatically through the entitlement.

## Key Topics

### Runs as Current User Entitlement
Adding the "User Management" capability in Xcode (with the "Runs as Current User" checkbox) causes tvOS to relaunch the app process as the currently selected user on every user switch. All system resources — UserDefaults, file system sandbox, per-user Keychain — are automatically scoped to that user.

### User-Independent Keychain (New in tvOS 16)
A new Keychain Services constant `kSecUseUserIndependentKeychain` lets apps write credentials that are visible to every user on the device. This enables the "sign in once, everyone benefits" model: sign-in credentials live in the shared keychain, while profile preferences live in per-user UserDefaults.

### Suggested Family Members in Control Center
iCloud Family members who haven't yet been added to the Apple TV are now surfaced as suggested users directly in Control Center (long-press the TV button). Adding a new family member only requires bringing their iPhone nearby and confirming on device — no account setup needed on Apple TV.

### Passkeys and OAuth on tvOS 16
tvOS 16 gains support for passkeys and OAuth authentication flows, enabling modern, passwordless sign-in experiences on Apple TV.

### Deprecated: TVUserManager Manual Profile Mapping
The TVUserManager methods for manually mapping Apple TV system users to app profiles are deprecated in tvOS 16. The OS handles this automatically once the entitlement is adopted.

## APIs & Frameworks

**TVServices / Keychain Services**
- `kSecUseUserIndependentKeychain` — **[NEW]** CFString key for `SecItemAdd` / `SecItemCopyMatching` attribute dictionaries; when set to `kCFBooleanTrue`, the item is stored in the user-independent Keychain accessible by all users
- `SecItemAdd(_:_:)` — existing Keychain Services function; now accepts `kSecUseUserIndependentKeychain`
- `kSecAttrService`, `kSecClass`, `kSecAttrAccount`, `kSecValueData` — standard Keychain attribute keys

**TVServices**
- `TVUserManager` — existing class for querying current user; profile-mapping methods deprecated in tvOS 16

**Foundation**
- `UserDefaults` — automatically scoped per-user when running with the Runs as Current User entitlement; same API as on iOS

**Entitlements / Capabilities**
- `com.apple.runningboard.assertions.xpc` / User Management capability — "Runs as Current User" entitlement; added in Xcode Signing & Capabilities tab, no code changes required

## Code Highlights

Saving credentials to the user-independent Keychain so all users on the Apple TV can access them:

```swift
func save(username: String, password: String) {
    guard let passwordData = password.data(using: .utf8) else { return }

    let attributes: [CFString: AnyObject] = [
        kSecAttrService: "MyApp" as AnyObject,
        kSecClass: kSecClassGenericPassword,
        kSecAttrAccount: username as AnyObject,
        kSecValueData: passwordData as AnyObject,
        kSecUseUserIndependentKeychain: kCFBooleanTrue
    ]

    let status = SecItemAdd(attributes as CFDictionary, nil)
    if status == errSecSuccess {
        self.credentials = (username, password)
    }
}
```

Storing the profile selection in UserDefaults (same code path on iOS and tvOS when running as current user):

```swift
// No platform check needed — UserDefaults is automatically per-user on tvOS
// when the Runs as Current User entitlement is active
UserDefaults.standard.set(selectedProfileID, forKey: "preferredProfile")
```

## Takeaways
- Adding the "Runs as Current User" entitlement requires only a single Xcode capability checkbox and zero code changes for most apps.
- Use `kSecUseUserIndependentKeychain: kCFBooleanTrue` in Keychain attribute dictionaries to share sign-in credentials across all users while keeping all other data separate.
- The same UserDefaults / file-system code used on iOS works as-is on tvOS once the entitlement is active — the OS scopes storage to the current user automatically.
- `TVUserManager` profile-mapping APIs are deprecated; remove manual mapping code when adopting the new entitlement.

---
_Source: WWDC22 Session 110384 page (abstract, chapter summaries, code samples, and resource links)._
