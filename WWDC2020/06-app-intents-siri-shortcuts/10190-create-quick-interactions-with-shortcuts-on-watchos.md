# Create Quick Interactions with Shortcuts on watchOS
**WWDC20 · Session 10190** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10190/)

_Platforms:_ watchOS 7, iOS 14

## Overview
This session introduces the all-new Shortcuts app on watchOS 7 and explains how developers can build great Shortcuts experiences for Apple Watch. The Shortcuts app on watchOS syncs shortcuts from iOS via iCloud, allowing users to run any shortcut from their wrist with a single tap — and even launch specific shortcuts directly from watch face complications.

The session details the two distinct execution paths for watch shortcuts — local execution via a watchOS Intents extension, and remote execution on the paired iPhone — and explains the constraints of each. Developers learn how to design intents that work reliably on watchOS, when remote execution applies, and how to craft high-quality voice dialogs for Siri on watchOS (where Intents UI extensions are not available).

## Key Topics

**Shortcuts App on watchOS 7 (New)**
A new standalone Shortcuts app on Apple Watch lets users run any shortcut with a single tap. Users can organize an Apple Watch collection on iOS that syncs to the watch. Parameters can be answered inline. Complications can launch the Shortcuts app or run a specific shortcut directly from the watch face.

**Execution Paths for Shortcuts on watchOS**
- **NSUserActivity-based shortcuts**: require opening the app on the watch. If the watch app is not installed, an error occurs. Remote execution to the paired phone is not supported.
- **Intent-based shortcuts (local)**: if an Intents extension is installed on the watch and supports the intent, it handles it directly — fastest path.
- **Intent-based shortcuts (remote)**: if no local watchOS Intents extension handles the intent, it is sent to the paired iPhone's Intents extension. This adds latency but works even without a watch app.
- **Remote execution caveat**: if the iPhone Intents extension returns `.continueInApp`, the shortcut cannot be completed remotely and an error is shown.

**Best Practices for watchOS Shortcuts**
- Build a watchOS app and a corresponding watchOS Intents extension targeting the same intents as the iOS version.
- Prefer intent-based shortcuts over `NSUserActivity`-based ones for new adopters — Intents are richer, support parameters, and can run in background.
- Compile the Intents extension for both iOS and watchOS (requires a separate watchOS Intents extension target in Xcode).
- Support as many intents as possible natively in the watchOS Intents extension to avoid remote execution latency.
- Enable "supports background execution" in the Xcode intent editor for all parameter combinations used in interaction donations to allow remote execution.
- Do not return `.continueInApp` from `confirm()` or `handle()` if remote execution support is needed.

**Voice Dialogs on watchOS**
Because Intents UI extensions are not supported on watchOS, all Siri interactions must convey necessary information through carefully written dialog strings. Custom confirm responses (rather than default "ready" responses) allow apps to surface order totals, confirmation details, and other relevant information before the user commits to an action.

**Siri Watch Face Integration**
Apps can offer shortcuts on the Siri watch face using the Relevant Shortcuts API so that shortcuts surface at the right time and context.

## APIs & Frameworks

### SiriKit / Intents Framework
- `INIntent` — base class for all SiriKit intents
- `INIntentHandling` — protocol for handling intents in an Intents extension
- `INIntentResponse` — response returned from intent handlers
- `INIntentResponseCode.continueInApp` — response code that forces app launch; must be avoided for remote execution
- `INIntentResponseCode.ready` — default confirmation response code
- Custom intent response with dialog — provides richer confirmation UI in Siri
- `INRelevantShortcut` — used to donate shortcuts to the Siri watch face
- `INRelevantShortcutStore.default.setRelevantShortcuts(_:completionHandler:)` — registers relevant shortcuts for the Siri watch face
- `INShortcut` — wraps either an `INIntent` or `NSUserActivity` for shortcut donation
- `INInteraction` — donated to the system to build shortcut suggestions

### Intents Extension (watchOS)
- watchOS Intents extension target — separate Xcode target required; bundled with the watchOS app
- "Supports background execution" checkbox (Xcode intent editor) — must be enabled per parameter combination for remote execution eligibility

### NSUserActivity
- `NSUserActivity` — basis for user activity-based shortcuts; not supported for remote execution on watchOS
- `NSUserActivity.isEligibleForPrediction` — marks activity as shortcut-eligible

### Shortcuts App (watchOS 7, New)
- Shortcuts app on watchOS — standalone app for browsing and running shortcuts **[NEW]**
- Apple Watch collection — iOS-configured list of shortcuts synced via iCloud to the watch **[NEW]**
- Shortcut complications — complications that launch a specific shortcut directly from the watch face **[NEW]**
- Inline parameter answering — Shortcuts app prompts for parameters on-device **[NEW]**

## Code Highlights

To support remote execution on the paired phone, the intent handler must not return `.continueInApp`:
```swift
func handle(intent: OrderSoupIntent, completion: @escaping (OrderSoupIntentResponse) -> Void) {
    // Perform the task in background — do NOT call completion with .continueInApp
    placeOrder(for: intent.soup) { success in
        completion(success ? OrderSoupIntentResponse(code: .success, userActivity: nil)
                           : OrderSoupIntentResponse(code: .failure, userActivity: nil))
    }
}
```

Custom confirmation dialog providing order total before user commits:
```swift
func confirm(intent: OrderSoupIntent, completion: @escaping (OrderSoupIntentResponse) -> Void) {
    let response = OrderSoupIntentResponse(code: .ready, userActivity: nil)
    response.totalPrice = calculateTotal(for: intent.soup)
    // System will display totalPrice in the confirmation dialog
    completion(response)
}
```

## Takeaways
- Build a native watchOS Intents extension to handle shortcuts locally on the watch — this is the fastest and most reliable path and avoids dependency on the paired iPhone.
- For apps without a watch app, intent-based shortcuts with background execution support will work via remote execution on the paired iPhone, but `NSUserActivity` shortcuts will fail on watchOS without a watch app installed.
- Never return `.continueInApp` from intent handlers if you want the shortcut to be usable on watchOS without opening the app.
- Craft informative custom dialogs for Siri on watchOS; since Intents UI extensions are unsupported on the watch, all confirmation and result context must be conveyed through response dialog strings.

---
_Source: WWDC20 Session 10190 page (abstract, chapter summaries, code samples, and resource links)._
