# Configure and Link Your App Clips
**WWDC20 · Session 10146** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10146/)

_Platforms:_ iOS 14, iPadOS 14

## Overview
App Clips are lightweight experiences invoked through physical and digital triggers (NFC tags, QR codes, App Clip Codes, Maps, Siri Nearby, Safari Smart App Banners, iMessage link bubbles). This session covers the three-part technical configuration for linking into an App Clip: setting up the `appclips` entry in the apple-app-site-association (AASA) file on the web server, adding the `appclips:` associated domain to the Xcode target, and handling the incoming `NSUserActivity` of type `NSUserActivityTypeBrowsingWeb` in SwiftUI or UIKit scene code.

The session also covers how to configure default and advanced App Clip experiences in App Store Connect (image, title, subtitle, action verb, optional location association), best practices for registering experience URLs using prefix matching, setting up the Smart App Banner HTML meta tag for Safari/Messages, and App Clip beta testing via TestFlight invocation points. The demo uses the "Fruta" smoothie ordering app clip to show the complete end-to-end flow including NFC physical invocation.

## Key Topics

**Invocation Methods**
App Clips can be launched from: NFC tags (URL encoded in tag), QR codes, App Clip Codes (Apple visual codes, released later in 2020), Maps place cards, Siri Nearby suggestions, Safari Smart App Banners, and iMessage link bubbles. All invocations deliver a URL to the App Clip via `NSUserActivity.webpageURL`.

**Associated Domains Setup (Two Parts)**

_Server side:_ Add an `"appclips"` entry to the AASA file at `.well-known/apple-app-site-association` on the web server:
```json
{
  "appclips": {
    "apps": [ "ABCDE12345.example.fruta.Clip" ]
  }
}
```
The value is the App Clip's full app identifier (team ID prefix + bundle ID).

_App side:_ Add the `Associated Domains` capability in Xcode and add the domain string `appclips:your-website-domain.com`.

**Handling NSUserActivity in the App Clip**
The App Clip's entry point (SwiftUI `WindowGroup` or UIKit `UISceneDelegate`) must register for `NSUserActivityTypeBrowsingWeb`. Parse `userActivity.webpageURL` to extract query parameters and deep-link to the correct content.

**Testing URL Handling with Xcode**
Set the `_XCAppClipURL` environment variable in the scheme's Arguments tab to a test URL. On run, the App Clip launches with that URL, simulating a real invocation.

**App Clip Experiences on App Store Connect**

_Default experience:_ Visible from Safari Smart App Banners and Messages link bubbles. Configure image, subtitle, and action verb (from a predefined list).

_Advanced experiences:_ Required for NFC, QR, App Clip Codes, Maps, and Siri Nearby. Each is tied to a specific URL (used as a prefix match). A single experience URL covers all URLs sharing that prefix. Register more specific URLs only when different metadata (image/subtitle) is needed for a specific sub-experience. The App Clip must also handle being launched with the exact registered URL (not just URL prefixes with query parameters).

**URL Prefix Match Strategy**
- One experience URL covers all invocations that match as a prefix.
- Example: registering `https://bikesrental.example/rent` covers all bike URLs with query parameters like `?id=123`.
- Register a more specific URL (e.g., flagship store URL) only when distinct metadata is needed.

**Smart App Banner HTML Configuration**
Add the meta tag to the web page's `<head>`. Set `app-clip-bundle-id` to the App Clip's bundle identifier and keep `app-id` for iOS 13 fallback:
```html
<meta name="apple-itunes-app"
      content="app-clip-bundle-id=com.example.fruta.Clip, app-id=123456789">
```
Safari validates the domain association before showing the banner.

**App Clip Card Requirements**
- Title: max 18 characters
- Subtitle: max 43 characters
- Image: 3000×2000 px (3:2 ratio), JPG or PNG
- Action verb: selected from Apple's predefined list (Order, Reserve, Get Ticket, etc.)

**TestFlight for App Clips**
Add "App Clip Invocations" in TestFlight to let beta testers test specific experience URLs without NFC tags or QR codes.

## APIs & Frameworks

### AppClip Framework **[NEW]**
- `AppClip` framework — not a code-heavy API; primarily entitlements + `NSUserActivity` handling

### Foundation / UIKit / SwiftUI
- `NSUserActivityTypeBrowsingWeb` — activity type delivered on App Clip invocation
- `NSUserActivity.webpageURL` — the URL that triggered the App Clip launch
- `NSURLComponents(url:resolvingAgainstBaseURL:)` — parse the incoming URL
- `URLComponents.queryItems` — access `?key=value` parameters from the URL

### SwiftUI App Life Cycle
- `.onContinueUserActivity(NSUserActivityTypeBrowsingWeb) { userActivity in ... }` **[NEW]** — registers handler on `WindowGroup` for browsing activity; replaces UIKit `UISceneDelegate.scene(_:continue:)`

### UIKit Scene Life Cycle
- `UISceneDelegate.scene(_:continue:)` — handles incoming user activities
- `UIScene.ConnectionOptions.userActivities` — activities from cold launch

### Web Server (AASA File)
- `appclips` key in `apple-app-site-association` — new App Clips service type; parallel to `applinks`
- `apps` array — contains the full App Clip app identifier (`TEAMID.bundleid`)
- Served from `.well-known/apple-app-site-association` with content-type `application/json`

### Xcode Capabilities
- Associated Domains capability: domain string format `appclips:yourdomain.com`
- Environment variable `_XCAppClipURL` — test URL injected at launch for debugging

### App Store Connect / TestFlight
- App Clip configuration section: visible after delivering a build containing the App Clip target
- Default experience: image + subtitle + action verb; used for Safari/Messages
- Advanced experience: URL + image + subtitle + action verb + optional location; required for physical invocations
- TestFlight App Clip Invocations: configure test URLs for beta testers to exercise specific experiences

### HTML (Smart App Banner)
- `<meta name="apple-itunes-app" content="app-clip-bundle-id=..., app-id=...">` — enables Safari banner and Messages link bubble with App Clip card

## Code Highlights

AASA file `appclips` entry:
```json
{
  "appclips": {
    "apps": ["ABCDE12345.com.example.fruta.Clip"]
  },
  "applinks": { ... },
  "webcredentials": { ... }
}
```

Handling invocation in SwiftUI:
```swift
@main
struct FrutaClip: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
                .onContinueUserActivity(NSUserActivityTypeBrowsingWeb) { userActivity in
                    guard let incomingURL = userActivity.webpageURL,
                          let components = NSURLComponents(url: incomingURL,
                                                          resolvingAgainstBaseURL: true)
                    else { return }
                    if let smoothieID = components.queryItems?
                        .first(where: { $0.name == "smoothie" })?.value {
                        dataStore.selectedSmoothieID = smoothieID
                    }
                }
        }
    }
}
```

Handling invocation in UIKit SceneDelegate:
```swift
func scene(_ scene: UIScene, continue userActivity: NSUserActivity) {
    guard userActivity.activityType == NSUserActivityTypeBrowsingWeb,
          let incomingURL = userActivity.webpageURL,
          let components = NSURLComponents(url: incomingURL, resolvingAgainstBaseURL: true)
    else { return }
    // Parse components.queryItems and navigate
}
```

Smart App Banner meta tag:
```html
<meta name="apple-itunes-app"
      content="app-clip-bundle-id=com.example.fruta.Clip, app-id=123456789">
```

## Takeaways
- The AASA file must include an `"appclips"` entry with the App Clip's full app identifier; this is a separate service type from `"applinks"` and both can coexist in the same file.
- All App Clip invocations (NFC, QR, Maps, Siri, Safari, Messages) deliver the same `NSUserActivity` of type `NSUserActivityTypeBrowsingWeb` — handle it in one place; ensure both the App Clip and the full app handle the same URLs for universal link continuity after upgrade.
- Use URL prefix matching strategically: register one short, general experience URL to cover many invocation URLs, and add more specific registrations only where distinct App Clip card metadata (different image/subtitle) is needed.
- Set `_XCAppClipURL` in the Xcode scheme's environment variables to test URL parsing during development without needing a physical NFC tag or QR code.

---
_Source: WWDC20 Session 10146 page (transcript, code samples, and resource links)._
