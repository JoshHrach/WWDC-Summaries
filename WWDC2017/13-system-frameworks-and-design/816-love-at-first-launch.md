# Love at First Launch
**WWDC17 · Session 816** · [Watch](https://developer.apple.com/videos/play/wwdc2017/816/)

_Platforms:_ iOS 11, macOS High Sierra 10.13, tvOS 11, watchOS 4

## Overview
First impressions determine whether a user keeps or deletes an app within the first few moments of use. This design-focused session argues that the standard pattern — greeting new users with a mandatory registration wall or a barrage of permission requests before showing any value — is the single greatest threat to user retention at first launch. The alternative: lead with content, teach through interaction rather than instructions, and ask for permissions only at the moment they become genuinely necessary.

The session uses concrete app examples to illustrate each principle. Jetsetter leads with beautiful travel content and defers registration until the moment a user tries to book. Lara Croft GO teaches all game mechanics through the first two levels of gameplay without a single tooltip or onboarding screen. Streaks uses an interactive ring-filling exercise as its onboarding — the same mechanic that defines the app's core loop. Strava requests location only when a user starts a workout, and photo library access only when a user tries to add a photo to a saved ride.

The session also warns against over-explaining intuitive interfaces. Apple's own Phone app is shown as an example of an interface so obvious that any onboarding tooltips would be absurd. Developers should design for clarity first and reserve onboarding for genuinely novel interactions.

## Key Topics
- **Content-first approach** — display useful, engaging content immediately on first launch; defer registration until the user has experienced value and has a reason to commit
- **Frictionless first experience** — no mandatory gates before showing value; open, explorable UI that invites interaction rather than demanding information
- **Teach through interaction** — use the actual interface and game mechanics to onboard (Lara Croft GO levels 1–2, Streaks ring-filling); avoid floating tooltips and static instruction screens
- **Avoid unnecessary onboarding** — evaluate honestly whether any upfront instruction is needed; design interfaces to be self-evident; onboarding is a design failure for obvious UIs
- **Context-appropriate permission requests** — request permissions at the moment they provide immediate, obvious value, not upfront in bulk
- **Deferred registration** — show what the app can do before asking who the user is; registration requests are most effective after demonstrated value
- **Value justification before permission requests** — explain or demonstrate what the user gains from granting a permission before the system dialog appears
- **Permission hygiene** — only request what is actually needed; request it when the feature that needs it is first used

## APIs & Frameworks
This is a design principles session with no new APIs. The iOS permission and onboarding-related APIs that underpin these design recommendations are:

### Permission APIs
- **`CLLocationManager.requestWhenInUseAuthorization()`** / **`requestAlwaysAuthorization()`** — location permission; request only at first meaningful use of location (e.g., starting a workout)
- **`PHPhotoLibrary.requestAuthorization(_:)`** — photo library access; request at the moment user initiates a photo-adding action
- **`AVCaptureDevice.requestAccess(for:completionHandler:)`** — camera/microphone; request at first camera use
- **`HKHealthStore.requestAuthorization(toShare:read:completionHandler:)`** — HealthKit data; request at first health-dependent feature use
- **`CNContactStore.requestAccess(for:completionHandler:)`** — contacts; request at "find friends" or similar feature initiation
- **`NSLocationWhenInUseUsageDescription`**, **`NSPhotoLibraryUsageDescription`**, **`NSCameraUsageDescription`**, **`NSContactsUsageDescription`**, **`NSHealthShareUsageDescription`** (Info.plist keys) — human-readable justification shown in system permission alert

### UIKit / App Architecture
- **`UIViewController` initial presentation** — choose opening view controller carefully; lead with content, not login/registration
- **`UIPageViewController`** — sometimes used for tutorial flows; use only when interactive onboarding is not possible
- **Skip/Later patterns** — always offer a way to bypass registration or permission requests and return to them later

## Code Highlights
No code samples were shown; this is a design principles session. The key architectural decision is to not present a registration screen as `rootViewController` until the user chooses to register:

```swift
// Anti-pattern: always show login first
window?.rootViewController = LoginViewController()

// Better: show content immediately; only prompt when action requires auth
window?.rootViewController = ContentFeedViewController()

// In ContentFeedViewController, when user taps "Book":
func bookingTapped() {
    guard isLoggedIn else {
        present(RegistrationViewController(), animated: true)
        return
    }
    // proceed with booking
}

// Permission: request at point of use, not at launch
func addPhotoTapped() {
    PHPhotoLibrary.requestAuthorization { status in
        // now show photo picker
    }
}
```

## Takeaways
- Lead with content: the first screen a user sees should demonstrate the app's core value, not demand credentials or personal information.
- Teach through the actual interface and interaction patterns; static onboarding screens and tooltips are a fallback for unclear design, not a feature.
- Request permissions contextually — at the moment the user takes an action that requires them — and always establish value before the system dialog appears.
- Deferred registration dramatically reduces abandonment at first launch; users who have experienced value are far more willing to create an account.

---
_Source: WWDC17 Session 816 page (abstract, chapter summaries, code samples, and resource links)._
