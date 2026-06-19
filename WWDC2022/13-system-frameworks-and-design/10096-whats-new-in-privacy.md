# What's New in Privacy
**WWDC22 · Session 10096** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10096/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, watchOS 9

## Overview
Apple's annual privacy update covers platform changes that affect app behavior, new privacy-enhancing technologies to adopt, and a new user-facing safety feature. The session is organized around Apple's four privacy pillars: data minimization, on-device processing, transparency and control, and security protections.

Key platform changes include `UIDevice.name` now returning the device model instead of the user-assigned name (requiring a new entitlement for the full name), Gatekeeper now checking the integrity of all notarized apps (not just newly quarantined ones), a new simplified login-item API via `SMAppService`, and pasteboard access now requiring explicit user intent. New developer-facing technologies include UIKit paste controls, Media Device Discovery Extensions (for prompt-free streaming device access), PHPicker on Mac/Watch, Private Access Tokens, and Passkeys.

Safety Check is a new iOS 16 system feature that helps people in domestic violence situations quickly revoke all data sharing and access they may have previously granted to others.

## Key Topics

### Platform Changes

**Device Name Access (`UIDevice.name`)**
`UIDevice.name` now returns the device model (e.g., "iPhone") rather than the user-assigned name. Apps needing the real device name for multi-device UI features can request the new "Device Name Entitlement." Sharing device names with third parties other than cloud-hosting providers is not permitted.

**Location Attribution in Control Center**
iOS 16 shows which app is using location when swiping down to Control Center. Apps must only use location when expected.

**Gatekeeper on macOS Ventura**
Gatekeeper now checks the integrity of all notarized apps on every launch, not just newly-quarantined downloads. Apps must be validly signed. Apps that lose their valid signature will be blocked. Updates from the same developer team still work automatically. A new `NSUpdateSecurityPolicy` Info.plist key controls which external processes are allowed to update an app. Unauthorized modifications notify the user via a system prompt.

**Login Items — `SMAppService` API**
macOS Ventura replaces the fragmented landscape of launch agents, daemons, SMLoginItems, and open-at-login with a single unified `SMAppService` API. Resources live inside the app bundle. Users are notified when apps add login items. Apps with elevated-privilege daemons require admin approval. All login item types are now managed in the Login Items pane in System Settings.

**Pasteboard Access**
iOS 16 requires explicit user intent for all cross-app pasteboard access. Apps using `UIPasteboard` APIs to read data from other apps will show a modal prompt. Alternatives: edit options menu, keyboard shortcuts, or the new `UIPasteControl` button.

### New Privacy-Enhancing Technologies to Adopt

**UIKit Paste Controls (`UIPasteControl`)**
A new button class that provides prompt-free pasteboard access when visibly displayed and tapped. Customizable corners, text, icon, and background colors. System verifies the button was genuinely visible and interacted with.

**Media Device Discovery Extensions (`DeviceDiscoveryExtension`)**
A new app extension type for streaming protocol providers. The extension discovers local network and Bluetooth devices in a sandboxed process that cannot send raw scan results back to the app. Only the user-selected device is exposed to the app. Eliminates the need for Local Network and Bluetooth permission prompts in streaming apps. Uses `AVRoutePickerView` for the unified picker UI.

**PHPicker on Mac and Watch**
`PHPickerViewController` is now available on macOS Ventura and watchOS 9, enabling photo access without prompting for full photo library permission.

**Private Access Tokens**
Replace CAPTCHAs using the Privacy Pass IETF open standard. Blinded tokens let servers verify legitimate devices without tracking device identity. Apple does not know what sites a device fetches tokens for; servers don't know device identity.

**Passkeys**
Public-key credential standard (WebAuthn/FIDO2) replacing passwords. Inherently phish-resistant (bound to the originating website). Uses FaceID/TouchID for biometric verification. See "Meet Passkeys" for full implementation details.

**Safety Check (iOS 16)**
New system privacy tool for users in domestic/intimate partner violence situations. Two modes: Emergency Reset (immediately resets all sharing with people and apps) and Manage Sharing & Access (granular per-person, per-app review). Includes Quick Exit for fast exit if observed. Not a developer API, but important context for how apps handle data sharing.

## APIs & Frameworks

**UIKit**
- `UIDevice.name` — **[CHANGED]** now returns model name; requires entitlement for user-assigned name
- `UIPasteControl` **[NEW]** — button providing prompt-free pasteboard access
- `UIPasteControl.Configuration` **[NEW]** — styling configuration for paste button

**AppKit / ServiceManagement**
- `SMAppService` **[NEW]** — unified API for registering login items, agents, and daemons
- `NSUpdateSecurityPolicy` Info.plist key **[NEW]** — allow specific teams/processes to update app

**DeviceDiscoveryExtension** **[NEW]**
- `DeviceDiscoveryExtension` framework — app extension for prompt-free streaming device discovery
- `AVRoutePickerView` — extended to handle `DeviceDiscoveryExtension` selections

**Photos**
- `PHPickerViewController` — now available on macOS Ventura and watchOS 9 **[NEW platforms]**

**Security / Authentication**
- Passkeys (WebAuthn/FIDO2) — phish-resistant public-key credentials; see "Meet Passkeys"
- Private Access Tokens (Privacy Pass) — CAPTCHA replacement; see "Replace CAPTCHAs with Private Access Tokens"
- App Tracking Transparency — see "Explore App Tracking Transparency"

**Gatekeeper**
- `NSUpdateSecurityPolicy` — allow specific teams to update app without user prompt **[NEW]**
- Gatekeeper now validates all notarized app signatures on every launch **[macOS Ventura behavior change]**

## Code Highlights

```swift
// Register app/agent/daemon as a login item (macOS Ventura)
import ServiceManagement
try SMAppService.mainApp.register()

// UIPasteControl — prompt-free paste button
let config = UIPasteControl.Configuration()
config.displayMode = .iconAndLabel
config.cornerStyle = .capsule
let pasteButton = UIPasteControl(configuration: config)
pasteButton.target = self
```

```xml
<!-- NSUpdateSecurityPolicy: allow Pal About to update this app -->
<key>NSUpdateSecurityPolicy</key>
<dict>
  <key>AllowProcesses</key>
  <dict>
    <key>PAL123TEAMID</key>
    <array>
      <string>com.example.pal.about</string>
    </array>
  </dict>
</dict>
```

## Takeaways

- `UIDevice.name` returns the device model in iOS 16; request the Device Name Entitlement only if your app genuinely shows the user-assigned name in a multi-device UI context.
- Update macOS apps to use `SMAppService` for login items and audit `NSUpdateSecurityPolicy` for any external update mechanisms before shipping on macOS Ventura.
- Add `UIPasteControl` buttons to avoid the new iOS 16 pasteboard access modal prompt for cross-app clipboard reads.
- Adopt `DeviceDiscoveryExtension` for streaming apps to eliminate Local Network and Bluetooth permission prompts — providing a better user experience and stronger privacy guarantee.

---
_Source: WWDC22 Session 10096 page (abstract, chapter summaries, code samples, and resource links)._
