# What's New in App Clips
**WWDC23 · Session 10178** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10178/)

_Platforms:_ iOS 17, iPadOS 17

## Overview
This session covers three improvements to App Clips in iOS 17: an increased size limit for digital invocations, a new auto-generated "default App Clip link" system that removes the need to configure custom web endpoints for simple use cases, and the ability to invoke any App Clip directly from within another app without going through Safari or Safari View Controller.

App Clips continue to be invocable from Messages, Maps points of interest, Safari, Spotlight, and physical triggers (NFC tags, QR codes, App Clip Codes). The new changes lower the barrier to building and distributing App Clips while enabling richer experiences at launch time.

## Key Topics

### New Size Limit (iOS 17)
- Digital invocations (links, Messages, Maps, Safari, Spotlight): size limit raised from 15 MB to **50 MB**.
- Physical invocations (NFC tags, App Clip Codes, QR codes): limit remains **15 MB** (as set in iOS 16) to ensure fast on-the-go experiences.
- iOS 15 and earlier: original **10 MB** limit still applies.
- Larger size allows bundling more assets at launch for a richer initial experience rather than downloading during runtime.

### Default App Clip Links (iOS 16.4+)
- Apple auto-generates a default App Clip link when an App Clip is published in App Store Connect — no developer-hosted web endpoint required.
- Link format: `https://appclip.apple.com/id?p=<bundle-id>`
- App-specific query parameters can be appended to the URL for context (e.g., a character selection parameter for a game demo).
- Parameters are retrieved using `NSUserActivity.webpageURL` and parsed with `NSURLComponents` — same mechanism as existing App Clip invocations.
- Eliminates the need to configure Apple App Site Association (AASA) files or maintain a web server for the default experience.

### Invoke App Clips from Your App (iOS 17 NEW)
- Any app can now invoke any App Clip directly, without routing through Safari or Safari View Controller.
- Use cases: food ordering from a messaging app, launching a service App Clip from a navigation app, etc.
- Two SwiftUI/UIKit approaches:
  1. **`Link` view** (SwiftUI) with the App Clip URL.
  2. **`UIApplication.shared.open(_:)`** with the App Clip URL.
- For richer integration: use `LPMetadataProvider` to fetch metadata for the URL, then present an `LPLinkView` as a tappable rich preview card.
- Works from any app to any App Clip that supports universal links.

## APIs & Frameworks

- `AppClip` framework — App Clips framework (existing)
- `NSUserActivity` — used to receive App Clip invocation context; `webpageURL` property carries the invocation URL
- `NSUserActivityTypeBrowsingWeb` — activity type for web-based App Clip invocations
- `NSURLComponents` — parse query items from the invocation URL
- `URLQueryItem` — individual query parameters from the App Clip URL
- `LPMetadataProvider` **[used for in-app App Clip invocation]** — fetches rich metadata for a URL to populate a link preview
- `LPMetadataProvider.startFetchingMetadata(for:completionHandler:)` — fetch URL metadata asynchronously
- `LPLinkView` — renders a tappable rich link preview; set `metadata` property to display App Clip card
- `Link` (SwiftUI) **[new use case for App Clips]** — SwiftUI view that opens a URL, now supporting direct App Clip invocation
- `UIApplication.shared.open(_:)` — UIKit method to open App Clip URLs directly
- Default App Clip link URL scheme: `https://appclip.apple.com/id?p=<bundle-id>` **[NEW]**
- App Store Connect — generates default App Clip links automatically on publish **[NEW]**
- Universal links / associated domains — underlying mechanism for App Clip invocations
- App Clip Codes, NFC tags, QR codes — physical invocation methods (15 MB limit maintained)

## Code Highlights

```swift
// Parsing URL parameters from a default App Clip link invocation
.onContinueUserActivity(NSUserActivityTypeBrowsingWeb) { userActivity in
    guard let inputURL = userActivity.webpageURL else { return }
    let components = NSURLComponents(url: inputURL, resolvingAgainstBaseURL: true)
    guard let parameters = components?.queryItems else { return }
    self.parameters = parameters
}

// Invoke an App Clip from SwiftUI using the default App Clip link
Link("Backyard Birds", destination: URL(string: "https://appclip.apple.com/id?p=com.example.backyardbirds.Clip")!)

// Invoke an App Clip from UIKit
UIApplication.shared.open(URL(string: "https://appclip.apple.com/id?p=com.example.backyardbirds.Clip")!)

// Show a rich App Clip preview using LinkPresentation
let provider = LPMetadataProvider()
provider.startFetchingMetadata(for: url) { metadata, error in
    guard let metadata = metadata else { return }
    DispatchQueue.main.async { lpView.metadata = metadata }
}
```

## Takeaways
- The 50 MB digital invocation limit opens App Clips to richer use cases (game demos, immersive on-demand ordering) that were previously impractical at 15 MB.
- Default App Clip links (`appclip.apple.com/id?p=...`) eliminate server-side setup for simple single-experience App Clips; parameters are appended as query items and parsed identically to existing universal-link invocations.
- Direct in-app App Clip invocation via `Link` or `UIApplication.open` (iOS 17) allows any app to seamlessly surface another app's App Clip, unlocking cross-app discovery and lightweight integration scenarios.

---
_Source: WWDC23 Session 10178 page (abstract, chapter summaries, code samples, and resource links)._
