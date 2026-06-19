# What's New in Privacy
**WWDC24 · Session 10123** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10123/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia 15

## Overview
Organized around Apple's four Privacy Pillars (Data Minimization, On-Device Processing, Transparency and Control, Security), this session covers the new privacy-enhancing changes in iOS 18 and macOS Sequoia: new out-of-process pickers (FinanceKit transaction picker, Image Playground picker, AccessorySetupKit picker), upgraded platform protections (Private Wi-Fi address rotation, macOS extension transparency, app group container protection), updated permission flows (Contacts two-stage access, Bluetooth prompt redesign, Local Network access on macOS), and new platform capabilities (app locking/hiding, automatic passkey upgrades, Live Caller ID via private information retrieval).

## Key Topics

### New Pickers for Data Minimization
Out-of-process pickers render system UI on top of the app without the app being able to read the picker content — only the explicitly selected item is shared.

**FinanceKit Transaction Picker**: Best for apps that need a set of existing transaction records (not a complete history). User selects specific transactions; app receives only those. For ongoing financial data access, FinanceKit's full access API lets users choose which accounts and the earliest sharing date per account. Full access requires the FinanceKit entitlement.

**Image Playground Picker**: Presents the system image generation UI (same as the Image Playground app). App provides optional text prompt or image input; user iterates over generated images. Only the final chosen image is shared. No permission prompt required.

**AccessorySetupKit Picker**: Replaces three separate prompts (Bluetooth pairing, Wi-Fi network join, Bluetooth confirmation) with a single out-of-process picker. App gets Bluetooth and Wi-Fi access scoped to the paired accessory only — cannot discover unpaired devices. Includes accessory management: rename, set icon, forget (removes permissions entirely), share with another app.

### Upgraded Platform Protections
**Private Wi-Fi (MAC Address Rotation)**: iOS already used random per-network MAC addresses. iOS 18 adds rotation: when "Rotate Wi-Fi Address" is on, the MAC changes approximately every two weeks. Off = static random per-network. Forgotten networks always rotate within 24 hours. Public networks default to static rotating; others default to random. This setting (renamed from "Private Wi-Fi Address") now exists on macOS Sequoia too. Apps using Wi-Fi MAC for network management or rate limiting must account for address changes.

**macOS Extension Transparency**: System notifications now appear when extensions are installed. Additional extension types (Dock Tile, QuickLook generators, etc.) appear in Login Items & Extensions and can be disabled there. Cron is now off by default (can be re-enabled). Directory Services plug-ins, legacy QuickLook plug-ins, and `com.apple.loginitems.plist` are no longer supported. Update any instructions pointing users to enable system extensions to reference General → Login Items & Extensions.

**App Group Container Protection**: App groups can now have sandbox-level protection on macOS. When an app from another developer tries to access a protected group container, a prompt is shown. Even un-sandboxed apps can protect a subset of their data in a group container. Use `containerURL(forSecurityApplicationGroupIdentifier:)` and declare the entitlement in `Info.plist` with correctly formatted group identifiers to avoid prompting your own apps.

### Updated Permission Flows
**Contacts (iOS 18)**: Two-stage prompt: first asks share/not-share, then (if "Continue") offers Limited or Full access. No new API required. New `ContactAccessButton` lets apps add contacts incrementally in-context (within app UI) without a full-screen picker — a tap on a unique match shares just that contact.

**Bluetooth (iOS 18)**: Updated permission prompt now shows a map of the device's current location and a sample of associated Bluetooth devices, alongside the usage string. Makes the privacy implications of Bluetooth access (location revelation, device tracking) transparent. No new APIs required.

**Local Network (macOS Sequoia)**: Local network access prompts come to macOS. Apps with `Bonjour Services` key in `Info.plist` or the `Networking Multicast` entitlement must include `NSLocalNetworkUsageDescription` or access is blocked. Applies to Bonjour browsing/advertising, custom multicast, custom broadcast, and unicast connections.

### New Platform Capabilities
**App Locking and Hiding (iOS 18)**: Users can lock any app (require Face ID/Touch ID/passcode for all entry points including Share Sheet actions) and hide apps (removed from search, notifications, and other system surfaces). Contents of locked/hidden apps are not accessible or visible. No developer action to support the feature; developers should not assume app visibility in search or notifications.

**Automatic Passkey Upgrades**: Apps can automatically upgrade existing accounts to passkeys during sign-in. Passkeys work alongside passwords — no login flow changes needed. See "Streamline sign-in with passkey upgrades and credential managers."

**Live Caller ID with Private Information Retrieval**: Uses homomorphic encryption so the server evaluates an encrypted query and returns transformed cipher text — the server never decrypts the phone number. Enables caller ID lookup without revealing the incoming number to the server. Open-source server resources available late 2024. Uses private relay for additional privacy. See `IdentityLookup` documentation.

## APIs & Frameworks

**FinanceKit**
- Transaction picker — out-of-process, no entitlement required **[NEW]**
- Full financial data access with per-account date control — requires FinanceKit entitlement **[NEW]**

**Image Playground**
- `ImagePlaygroundSheet` — out-of-process image generation picker **[NEW]**
- No permission prompt; only final image shared with app

**AccessorySetupKit**
- Unified out-of-process accessory pairing picker **[NEW]**
- Replaces separate Bluetooth pairing, Wi-Fi join, and Bluetooth confirmation prompts
- Scoped Bluetooth/Wi-Fi access to paired accessory only

**Contacts**
- `CNContactStore` — two-stage authorization flow with Limited access option **[NEW behavior]**
- `ContactAccessButton` (new UI component) **[NEW]** — in-context single-contact sharing without full picker
- `CNContactPickerViewController` — still available; works with limited access

**Bluetooth**
- `CBCentralManager` — updated permission prompt with location map and device sample **[NEW UI]**
- `NSBluetoothAlwaysUsageDescription` — plist key (required, as before)

**Local Network (macOS)**
- Local Network access prompt now on macOS Sequoia **[NEW]**
- `NSLocalNetworkUsageDescription` — required plist key if using Bonjour/multicast/broadcast **[enforcement]**
- `NSBonjourServices` — plist key that triggers Local Network prompt

**Authentication**
- Automatic passkey upgrades during sign-in **[NEW]** — `AuthenticationServices` framework
- `ASAuthorizationController` — existing, new automatic upgrade path

**Identity Lookup**
- Live Caller ID via Private Information Retrieval (homomorphic encryption) **[NEW]**
- `ILLookupExtensionContext` — Live Caller ID lookup extension

**App Groups / Sandbox (macOS)**
- App group container protection for sandboxed and non-sandboxed apps **[NEW]**
- `FileManager.containerURL(forSecurityApplicationGroupIdentifier:)` — correct API for group container access
- `com.apple.security.application-groups` entitlement — must be declared

**Privacy Infrastructure**
- Private Wi-Fi address rotation — system feature, no API; affects MAC-based network management
- App locking/hiding — system feature, no developer API

## Code Highlights
No developer-facing code snippets appear in the session. Integration points are through FinanceKit, AccessorySetupKit (see "Meet AccessorySetupKit"), ContactAccessButton (see "Meet the Contact Access Button"), and the `AuthenticationServices` passkey upgrade API (see "Streamline sign-in with passkey upgrades and credential managers").

## Takeaways
- Adopt the new pickers (FinanceKit transaction picker, AccessorySetupKit) instead of requesting broad permissions — they eliminate most of the permission dialog complexity while giving users full control over what is shared.
- Prepare apps for MAC address rotation: any network management, rate limiting, or user identification logic that relies on Wi-Fi MAC addresses must handle addresses that change every ~2 weeks.
- Update macOS apps that include extensions: add instructions pointing to General → Login Items & Extensions; legacy extension mechanisms (Directory Services plug-ins, legacy QuickLook, loginitems.plist) are no longer supported.
- Apps should not assume they are visible in search, Spotlight, or notifications — iOS 18 lets users hide any app, making any content in those surfaces inaccessible.

---
_Source: WWDC24 Session 10123 page (abstract, chapter summaries, and resource links)._
