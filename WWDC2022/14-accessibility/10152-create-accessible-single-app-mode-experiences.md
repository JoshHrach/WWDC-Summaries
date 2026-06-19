# Create Accessible Single App Mode Experiences
**WWDC22 · Session 10152** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10152/)

_Platforms:_ iOS 16, iPadOS 16

## Overview
This session covers the three Single App Mode options available on iOS and iPadOS — Guided Access, Autonomous Single App Mode, and Assessment Mode — and explains how to make each mode accessible to users who rely on assistive technologies. The presenter frames Single App Modes as tools for creating focused environments in kiosk, medical, and educational contexts, while emphasizing that restrictions must never block accessibility features users depend on.

The session walks through how to add custom app-level restrictions to Guided Access using `UIGuidedAccessRestrictionDelegate`, how to programmatically enter and exit Autonomous Single App Mode with `UIAccessibility.requestGuidedAccessSession(enabled:completionHandler:)`, and how to use the newly unified Automatic Assessment Configuration framework for test-proctoring scenarios. A key new API — `UIAccessibility.configureForGuidedAccess(_:enabled:)` — lets apps toggle specific accessibility features (VoiceOver, Zoom, etc.) programmatically during any Single App Mode session.

## Key Topics

### Guided Access — Custom App Restrictions
Guided Access is the user-facing accessibility feature that puts any app into Single App Mode temporarily. Apps can define custom restrictions that appear in the Guided Access options menu by conforming `AppDelegate` to `UIGuidedAccessRestrictionDelegate`. Custom restrictions are identified by string identifiers, have user-facing titles and optional descriptions, and can be queried at runtime.

Cognitive accessibility design principles to apply during Guided Access: be forgiving of errors (warn before irreversible actions), reduce dependence on timing, and always confirm before payments.

### Autonomous Single App Mode (ASAM)
ASAM requires a supervised device and the app's bundle ID to be allowlisted in the device's MDM configuration profile. The app calls `UIAccessibility.requestGuidedAccessSession(enabled:completionHandler:)` to programmatically enter or exit the restricted session. The completion handler indicates success or failure; apps should surface errors to the user. Current state is checked via `UIAccessibility.isGuidedAccessEnabled`; state changes are observed via `guidedAccessStatusDidChangeNotification`.

### Single App Mode (SAM)
Basic Single App Mode is configured through Apple Configurator or MDM on supervised devices. It persists across reboots and requires no runtime API. Best suited for permanently-dedicated kiosk devices.

### Assessment Mode
The Automatic Assessment Configuration framework (unified for iOS and macOS) supports testing scenarios, disabling autocorrect and spellcheck and locking to the app. Requires an assessment entitlement from Apple; no device supervision required.

### Accessibility During Single App Mode
Device management software can pre-configure accessibility features and the Accessibility Shortcut for use while locked. At runtime, `UIAccessibility.configureForGuidedAccess(_:enabled:)` lets apps programmatically toggle supported accessibility features — useful in kiosk enclosures where hardware buttons are blocked.

## APIs & Frameworks

### UIKit — UIAccessibility
- `UIAccessibility.requestGuidedAccessSession(enabled:completionHandler:)` — enters or exits Autonomous Single App Mode programmatically; app must be allowlisted in MDM profile
- `UIAccessibility.isGuidedAccessEnabled: Bool` — returns current Single App Mode state
- `UIAccessibility.guidedAccessStatusDidChangeNotification` — notification fired on mode state change
- `UIAccessibility.configureForGuidedAccess(_:enabled:)` **[NEW]** — programmatically toggles an accessibility feature during Single App Mode
- `UIGuidedAccessAccessibilityFeature` — enum of features controllable via `configureForGuidedAccess`: `.voiceOver`, `.zoom`, `.invertColors`, `.assistiveTouch`, `.grayscale`

### UIKit — UIGuidedAccessRestrictionDelegate
- `UIGuidedAccessRestrictionDelegate` protocol — add to `AppDelegate` to define custom Guided Access restrictions
- `guidedAccessRestrictionIdentifiers: [String]` — list of custom restriction IDs
- `textForGuidedAccessRestriction(withIdentifier:)` — user-facing title for each restriction
- `detailTextForGuidedAccessRestriction(withIdentifier:)` — optional longer description
- `guidedAccessRestriction(withIdentifier:didChange:)` — callback when a restriction is toggled
- `UIAccessibility.guidedAccessRestrictionState(forIdentifier:) -> UIAccessibilityGuidedAccessRestrictionState` — query current state of a custom restriction

### Automatic Assessment Configuration
- `AEAssessmentSession` — manages an assessment session (iOS/macOS unified framework)
- Requires assessment entitlement from Apple

## Code Highlights

Defining custom Guided Access restrictions:
```swift
class AppDelegate: UIResponder, UIApplicationDelegate, UIGuidedAccessRestrictionDelegate {
    var guidedAccessRestrictionIdentifiers: [String]? {
        return ["com.example.app.restriction.accountSettings"]
    }
    func textForGuidedAccessRestriction(withIdentifier id: String) -> String? {
        return "Account Settings"
    }
    func guidedAccessRestriction(withIdentifier id: String, didChange state: UIAccessibilityGuidedAccessRestrictionState) {
        NotificationCenter.default.post(name: .restrictionChanged, object: state)
    }
}

// Check restriction state:
let state = UIAccessibility.guidedAccessRestrictionState(forIdentifier: "com.example.app.restriction.accountSettings")
```

Entering and exiting Autonomous Single App Mode:
```swift
func enterRestrictedSession() {
    UIAccessibility.requestGuidedAccessSession(enabled: true) { success in
        if success { startNewPatientForm() }
        else { showRetryAlert() }
    }
}
func exitRestrictedSession() {
    UIAccessibility.requestGuidedAccessSession(enabled: false) { success in
        if success { savePatientData() }
    }
}
```

Toggling VoiceOver programmatically during Single App Mode:
```swift
UIAccessibility.configureForGuidedAccess(.voiceOver, enabled: true)
```

## Takeaways
- Choose the right Single App Mode for the use case: SAM for permanently dedicated kiosks, ASAM for apps that enter/exit restriction frequently, Assessment Mode for test-proctoring.
- Custom Guided Access restrictions via `UIGuidedAccessRestrictionDelegate` let apps tailor the locked experience for users with cognitive disabilities — hide confusing options and reduce error-prone actions.
- Always ensure accessibility features are available during Single App Mode, either via MDM pre-configuration or the new `configureForGuidedAccess(_:enabled:)` API at runtime.
- ASAM requires both device supervision and MDM allowlisting of the app's bundle ID — confirm this before shipping.

---
_Source: WWDC22 Session 10152 page (abstract, chapter summaries, code samples, and resource links)._
