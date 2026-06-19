# Expand Your SiriKit Media Intents to More Platforms
**WWDC20 · Session 10061** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10061/)

_Platforms:_ tvOS 14, HomePod (new); iOS, watchOS (existing SiriKit Media)

## Overview
Building on the SiriKit Media Intents introduced at WWDC19 (iOS 13, watchOS 6), this session covers three new areas: platform expansion to tvOS 14 and HomePod, a new Alternatives UI that lets users pick from multiple search results without leaving Siri, and two performance improvements — in-app intent handling and app pre-warming.

The session uses the "ControlAudio" sample app from WWDC19 to demonstrate all new features, with the key API change for alternatives being a single method name swap in `resolveMediaItems`.

## Key Topics

### New Platform: tvOS 14 **[NEW]**
SiriKit Media Intents now work in Apple TV apps. The implementation is identical to iOS: add an Intents extension target, implement `resolveMediaItems(for:with:)` and `handle(intent:completion:)`. The key difference from iOS is the launch preference:
- On iOS, background playback launch is usually preferred → return `.handleInApp`.
- On tvOS, the user is usually focused on one app, so a **foreground** launch is preferred → return `.continueInApp` from `handle`.

### New Platform: HomePod
SiriKit Media Intents now work on HomePod using the same intent definitions plus a new cloud playback API specific to HomePod's characteristics. HomePod access is controlled by a developer program — see the contact/request link on the Apple Developer site.

### Siri Alternatives UI **[NEW]**
Previously, `resolveMediaItems` returned a single resolved media item. If the chosen item wasn't what the user wanted, they had to open the app to change it.

iOS 14 adds a "Maybe You Wanted" menu in the compact Siri UI. Tapping it reveals the additional items returned by the app. Tapping any alternative re-invokes `handle` with that item set as `intent.mediaItem`.

**Implementation**: change from the singular `INPlayMediaMediaItemResolutionResult.success(with: mediaItems[0])` to the plural `INPlayMediaMediaItemResolutionResult.successes(with: mediaItems)`. All items after index 0 appear as alternatives. The handle phase is unchanged — no new code required there.

### Performance: In-App Intent Handling for SiriKit Media
Previously SiriKit Media required an Intents extension. iOS 14 allows in-app handling (see also session 10073). For media:
- **Advantage**: avoids launching both the extension process and the app; only one process launch needed.
- **Advantage**: resolve phase runs in the app, so credential fetching and audio engine warm-up can start earlier in the pipeline.
- **Consideration**: full app launch is slower than launching a lightweight extension — app launch time must be optimized.

### Performance: App Pre-Warming **[NEW]**
For apps keeping the existing Intents extension approach, SiriKit can now launch the app **earlier in the pipeline** (before handle completes) to warm up the playback engine while the extension still runs resolve and confirm. This reduces the perceived latency between Siri's response and audio playback starting.

Pre-warming work should be placed in `application(_:didFinishLaunchingWithOptions:)` — credential fetching, audio player initialization, etc. Requires coordination with Apple for correct implementation.

**Trade-off summary**: In-app intent handling collapses the process count; app pre-warming keeps the extension separation but starts the app earlier. Developers should benchmark both options for their specific app launch profile.

## APIs & Frameworks

### SiriKit / Intents (iOS 14 / tvOS 14)
- `INPlayMediaIntent` — unchanged; `mediaSearch`, `mediaItems`
- `INPlayMediaIntentHandling` — `resolveMediaItems(for:with:)`, `handle(intent:completion:)`
- `INPlayMediaMediaItemResolutionResult.success(with:)` — singular (existing)
- `INPlayMediaMediaItemResolutionResult.successes(with:)` **[NEW]** — plural; items after index 0 appear as alternatives in Siri UI
- `INPlayMediaIntentResponse(code:userActivity:)` — `.handleInApp` (background launch, preferred on iOS), `.continueInApp` (foreground launch, preferred on tvOS)

### UIKit (app pre-warming / in-app handling)
- `UIApplicationDelegate.application(_:didFinishLaunchingWithOptions:)` — place pre-warming logic here (credential fetch, audio engine init)

## Code Highlights

Returning multiple alternatives (one-line change from singular to plural):
```swift
func resolveMediaItems(for intent: INPlayMediaIntent,
                       with completion: @escaping ([INPlayMediaMediaItemResolutionResult]) -> Void) {
    let mediaSearch = intent.mediaSearch
    resolveMediaItems(for: mediaSearch) { optionalMediaItems in
        guard let mediaItems = optionalMediaItems else { return }
        // Previously: INPlayMediaMediaItemResolutionResult.success(with: mediaItems[0])
        completion(INPlayMediaMediaItemResolutionResult.successes(with: mediaItems))
    }
}
```

tvOS handle returning foreground launch:
```swift
func handle(intent: INPlayMediaIntent,
            completion: (INPlayMediaIntentResponse) -> Void) {
    completion(INPlayMediaIntentResponse(code: .continueInApp, userActivity: nil))
}
```

iOS handle returning background launch:
```swift
func handle(intent: INPlayMediaIntent,
            completion: (INPlayMediaIntentResponse) -> Void) {
    completion(INPlayMediaIntentResponse(code: .handleInApp, userActivity: nil))
}
```

App pre-warming (extension-based implementations):
```swift
func application(_ application: UIApplication,
                 didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    // Fetch credentials, initialize audio player engine here —
    // system launches app early while extension still runs resolve/confirm
    return true
}
```

## Takeaways
- Adding tvOS 14 SiriKit Media support requires only an Intents extension and using `.continueInApp` in handle (instead of iOS's `.handleInApp`) — the rest is identical to iOS.
- Switching from `success(with:)` to `successes(with:)` in `resolveMediaItems` is the only change needed to show the new Alternatives UI — items beyond index 0 automatically appear in the "Maybe You Wanted" menu.
- In-app intent handling removes the extension process, enables earlier credential/engine warm-up, but requires a fast app launch; app pre-warming offers similar benefits while preserving extension separation.
- HomePod support requires joining Apple's developer program; use the contact form at developer.apple.com/siri.

---
_Source: WWDC20 Session 10061 page (abstract, transcript, and code samples)._
