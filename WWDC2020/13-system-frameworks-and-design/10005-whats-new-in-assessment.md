# What's New in Assessment
**WWDC20 · Session 10005** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10005/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11 (Mac Catalyst)

## Overview
The Automatic Assessment Configuration (AAC) framework is introduced as a unified, cross-platform API for locking down devices during standardized testing. Previously, assessment mode on iPad relied on a UIKit API; AAC replaces that with a dedicated framework (`AutomaticAssessmentConfiguration`) that ships on iOS, iPadOS, and—for the first time—macOS Big Sur. Catalyst apps get Mac assessment mode support for free: calling the same `AEAssessmentSession` API on a Catalyst app triggers the appropriate OS-level lock-down on whichever platform the app is running.

On Mac, assessment mode restricts the environment to a single visible app (Dock, Mission Control, and Notification Center are hidden), prevents screen recording and capture, hides windows of other open apps, blocks network access for all other processes, pauses media playback, and clears the pasteboard at session start and end. On iOS/iPadOS, AAC replaces the older `UIAccessibilityRequestGuidedAccessSession` API and introduces an important new capability: `AEAssessmentConfiguration` exposes properties to selectively re-enable system services—dictation, predictive keyboard, spell check, continuous path keyboard—so assessment developers can tailor restriction levels to test content, student needs, and regional requirements.

## Key Topics
- **`AEAssessmentSession`** — the central object; manage its lifecycle via delegation **[NEW framework]**
- **`AEAssessmentConfiguration`** — configures what is restricted; default = most restrictive; expandable for future OS capabilities **[NEW]**
- **Session lifecycle** — inactive → begin requested → beginning (async, show transition UI) → active → end requested → ending (async) → inactive
- **Delegate callbacks** — `assessmentSessionDidBegin`, `failedToBeginWithError`, `assessmentSessionDidEnd`, `wasInterruptedWithError`
- **Error handling** — distinguish startup failures (session is now invalid; discard) from runtime interruptions (hide content immediately, then call `end`)
- **macOS restrictions** — Dock/Mission Control/Notification Center hidden; screen recording blocked; other app windows hidden; non-app network access blocked; pasteboard cleared; media paused
- **iOS/iPadOS configurability** — `AEAssessmentConfiguration` properties to enable: dictation, predictive keyboard, spell check, continuous path keyboard
- **Mac Catalyst support** — same API triggers correct platform-specific assessment mode **[NEW]**
- **Protocol-oriented design** — abstract behind a custom protocol to enable unit testing and Xcode debugging (assessment mode blocks Xcode interaction; a mock implementation avoids device lockout during development)
- **Debugging tip** — if assessment mode locks out Xcode during debugging, reboot the Mac; all services guaranteed to restore on restart

## APIs & Frameworks

**AutomaticAssessmentConfiguration (new framework)**
- `AEAssessmentConfiguration` **[NEW]** — session configuration object; default = most restrictive
  - `allowsAccessibilitySpeech: Bool` — enable dictation/speech accessibility
  - `allowsPredictiveKeyboard: Bool` — enable predictive text
  - `allowsSpellCheck: Bool` — enable spell check
  - `allowsContinuousPathKeyboard: Bool` — enable swipe-to-type keyboard
  - (Additional properties added over time; designed to be an expandable value type)
- `AEAssessmentSession` **[NEW]** — manages one assessment mode session
  - `init(configuration: AEAssessmentConfiguration)` — create a session
  - `delegate: AEAssessmentSessionDelegate?` — set before calling `begin()`
  - `func begin()` — asynchronously begin assessment mode
  - `func end()` — asynchronously end assessment mode and restore services
- `AEAssessmentSessionDelegate` **[NEW]** — protocol for lifecycle callbacks
  - `assessmentSessionDidBegin(_ session:)` — session fully active
  - `assessmentSession(_ session:, failedToBeginWithError error:)` — startup failed; session invalid
  - `assessmentSessionDidEnd(_ session:)` — session ended and OS restored
  - `assessmentSession(_ session:, wasInterruptedWithError error:)` — runtime integrity failure; hide content and call `end()`

**UIKit (deprecated path)**
- `UIAccessibilityRequestGuidedAccessSession(_:)` — older iPad-only assessment mode API; **now deprecated** in favor of `AEAssessmentSession`

## Code Highlights

Full AAC client implementation (session lifecycle + delegate):
```swift
import AutomaticAssessmentConfiguration

class AssessmentManager: NSObject {
    private var assessmentSession: AEAssessmentSession?

    func beginAssessmentMode() {
        let config = AEAssessmentConfiguration()
        // config.allowsSpellCheck = true  // enable as needed

        let session = AEAssessmentSession(configuration: config)
        session.delegate = self
        assessmentSession = session

        // Present transition UI (spinner, etc.)
        session.begin()
    }

    func endAssessmentMode() {
        guard let session = assessmentSession else { return }
        // Present transition UI
        session.end()
    }
}

extension AssessmentManager: AEAssessmentSessionDelegate {
    func assessmentSessionDidBegin(_ session: AEAssessmentSession) {
        // Remove transition UI
        // Present testing content
    }

    func assessmentSession(_ session: AEAssessmentSession,
                           failedToBeginWithError error: Error) {
        // Remove transition UI
        // Show error to user
        assessmentSession = nil  // session is now invalid
    }

    func assessmentSessionDidEnd(_ session: AEAssessmentSession) {
        // Remove transition UI
        // Show post-test content (results, confirmation)
        assessmentSession = nil
    }

    func assessmentSession(_ session: AEAssessmentSession,
                           wasInterruptedWithError error: Error) {
        // Immediately hide all sensitive content
        // Show error to user
        session.end()
    }
}
```

Protocol-oriented mock for unit testing (avoids locking down the test machine):
```swift
protocol AssessmentManaging {
    func beginAssessmentMode()
    func endAssessmentMode()
}

// Production implementation uses AEAssessmentSession
class RealAssessmentManager: AssessmentManaging { /* ... */ }

// Test implementation — no device lockdown
class MockAssessmentManager: AssessmentManaging {
    var didBegin = false
    func beginAssessmentMode() { didBegin = true }
    func endAssessmentMode() { didBegin = false }
}
```

## Takeaways
- `AEAssessmentConfiguration` is designed as an expandable value: new OS-level capability toggles will be added over time, so always construct fresh configurations rather than caching them.
- Assessment mode is asynchronous in both directions; always display transition UI between calling `begin()`/`end()` and receiving the corresponding delegate callback so users understand the device state is changing.
- Use the `wasInterruptedWithError` delegate to detect mid-exam integrity failures; hide sensitive content immediately and call `session.end()` to restore services before presenting an error.
- Abstract `AEAssessmentSession` behind a protocol for unit testing; the real session will lock down the Mac, preventing Xcode interaction during breakpoints—if this happens, a reboot fully restores all services.

---
_Source: WWDC20 Session 10005 page (transcript and code samples)._
