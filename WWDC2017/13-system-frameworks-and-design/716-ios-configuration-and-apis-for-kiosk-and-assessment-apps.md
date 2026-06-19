# iOS Configuration and APIs for Kiosk and Assessment Apps
**WWDC17 · Session 716** · [Watch](https://developer.apple.com/videos/play/wwdc2017/716/)

_Platforms:_ iOS 11

## Overview
This session covers four distinct techniques for locking an iOS device to a single application — a requirement for kiosk, hospitality, and educational assessment scenarios. The approaches range from the manual, user-accessible Guided Access (any device, any app) through MDM-managed Single App Mode for permanent kiosk deployments, to the two programmer-controlled methods: Autonomous Single App Mode (ASAM) for managed/supervised devices, and Automatic Assessment Configuration (AAC) for high-stakes educational testing on any device.

ASAM and AAC use the same underlying API (`UIAccessibilityRequestGuidedAccessSession`) but behave very differently depending on device configuration and entitlement. AAC is the recommended choice for assessment developers because in addition to locking the Home button it also automatically disables spell-check, auto-correct, dictionary lookups, the share sheet, the universal clipboard, and other features students could exploit to cheat — all of which remain active under the other three techniques.

The session includes a live coding demo where an insecure test app is upgraded to use AAC, covering the entitlement provisioning steps, the entitlements file, and the lock/unlock call pattern. Best practices for pasteboard clearing, extension (keyboard) blocking, and failure-mode handling are also covered.

## Key Topics
- **Guided Access** — works on any unmanaged iPad; user triple-taps Home button with passcode; not scalable for deployments
- **Single App Mode (MDM)** — MDM admin pushes command to supervised/managed devices; permanently locks to designated app; no developer code required; admin must remember to unlock
- **Autonomous Single App Mode (ASAM)** — developer calls `UIAccessibilityRequestGuidedAccessSession(true/false)` from within the app; requires supervised device + MDM whitelist of bundle IDs (`com.apple.developer.autonomous-single-app-mode` configuration profile key); app controls lock/unlock timing
- **Automatic Assessment Configuration (AAC)** — same API as ASAM; works on any device (managed or not); requires `com.apple.developer.edu-assessment-mode` entitlement (apply via Tech Support Incident); automatically disables: spell-check, auto-correct, dictionary lookups, share sheet, universal clipboard, Siri, and more; unlocks on explicit call, reboot, or after 8 hours
- **AAC failure modes** — fails if app lacks entitlement, if user declines the lock prompt, or if device is already locked; always check `UIAccessibilityIsGuidedAccessEnabled()` before attempting to lock
- **Pasteboard hygiene** — AAC disables universal clipboard but not the local pasteboard; must explicitly clear `UIPasteboard.general.items = []` on test entry and exit
- **Extension/keyboard blocking** — AAC does not block third-party keyboards; implement `application(_:shouldAllowExtensionPointIdentifier:)` in app delegate to disallow all extensions during a test
- **Provisioning requirements** — entitlement must be in both the provisioning profile and a project `.entitlements` file; set Code Signing Entitlements build setting in Xcode

## APIs & Frameworks

### UIKit / UIAccessibility
- **`UIAccessibilityRequestGuidedAccessSession(_:completionHandler:)`** — pass `true` to lock, `false` to unlock; completion handler receives `Bool` indicating success; same function for both ASAM and AAC **[KEY API]**
- **`UIAccessibilityIsGuidedAccessEnabled()`** — returns `Bool`; check before locking and on app launch to detect pre-existing lock state
- **`UIAccessibilityGuidedAccessStatusDidChangeNotification`** — `NotificationCenter` notification posted when lock state changes
- **`application(_:shouldAllowExtensionPointIdentifier:)` (UIApplicationDelegate)** — return `false` for `.keyboard` (and other extension point identifiers) to block third-party keyboards and extensions during assessment

### UIPasteboard
- **`UIPasteboard.general.items = []`** — clears local pasteboard on test start and exit to prevent content leakage

### Entitlement
- **`com.apple.developer.edu-assessment-mode`** (AAC entitlement) **[NEW]** — enables AAC on any device; granted by Apple via Tech Support Incident; must be present in both provisioning profile and `.entitlements` file
- **Autonomous Single App Mode config profile key** — bundle IDs whitelisted by MDM admin for ASAM usage on supervised devices

### MDM / Configuration
- **Mobile Device Management (MDM) Protocol** — used for Single App Mode push commands; referenced documentation at Apple Developer Library
- **Configuration Profile `AllowedApplications` key** — whitelist for ASAM-capable bundle IDs pushed to supervised devices
- **Supervised device requirement** — ASAM requires supervision; AAC does not

## Code Highlights

```swift
// Lock the device (ASAM or AAC depending on entitlement/device setup)
UIAccessibilityRequestGuidedAccessSession(true) { success in
    if success {
        // Device is now locked — proceed with sensitive content
        self.presentTestViewController()
    } else {
        // Lock failed — do not proceed
        self.showLockFailureAlert()
    }
}

// Unlock the device when done
UIAccessibilityRequestGuidedAccessSession(false) { _ in }

// Check lock state before attempting to lock
if UIAccessibilityIsGuidedAccessEnabled() {
    // Already locked — something is wrong, abort test
    showEnvironmentError()
}

// Clear pasteboard on test entry and exit (AAC does not do this automatically)
UIPasteboard.general.items = []

// Block third-party keyboards and extensions during assessment
func application(_ application: UIApplication,
                 shouldAllowExtensionPointIdentifier id: UIApplication.ExtensionPointIdentifier) -> Bool {
    return false  // Disallow all extensions
}
```

## Takeaways
- Use Guided Access for parental controls or one-off kiosks; use MDM Single App Mode for permanent fleet-wide kiosk deployments; use ASAM for contextual locking within managed-device enterprise apps; use AAC for any standardized testing scenario.
- AAC is the only technique that also disables cheating-relevant system features (spell-check, dictionaries, share sheet, etc.); it is the mandated approach for educational assessment developers.
- Always check `UIAccessibilityIsGuidedAccessEnabled()` before calling the lock function and verify that your app (not some other process) performed the locking before relying on AAC's feature restrictions.
- Third-party keyboards are not blocked by AAC; explicitly implement the extension-point delegate method to prevent keyboard-based cheating.

---
_Source: WWDC17 Session 716 page (abstract, chapter summaries, code samples, and resource links)._
