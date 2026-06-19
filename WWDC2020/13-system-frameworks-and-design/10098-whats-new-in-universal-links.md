# What's New in Universal Links
**WWDC20 · Session 10098** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10098/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, watchOS 7, tvOS 14

## Overview
Universal Links received substantial improvements in 2020 across three areas: new platform support (watchOS and SwiftUI), powerful new pattern-matching features in the apple-app-site-association file (substitution variables, case-insensitive matching, Unicode/percent-encoding support), and a new Apple-managed CDN that pre-fetches associated domain data at app install time to eliminate failures caused by server unavailability.

A major pain point — exponential growth of pattern entries when combining languages, regions, and product names — is solved by substitution variables, which name reusable lists of match strings. A real-world example in the session showed a 27 MB apple-app-site-association file reduced to a compact, readable declaration.

## Key Topics

### New Platform Support
- **watchOS** — Universal Links now work on watchOS. Use `WKExtensionDelegate.handle(_ userActivity:)` to handle incoming links; use `WKExtension.shared().openSystemURL(_:)` to open links in other apps. If the target app is not installed, the system presents an alert directing the user to continue on the paired iPhone.
- **SwiftUI** — The `onOpenURL(perform:)` modifier handles incoming universal links declaratively on all platforms.

### Pattern Matching Improvements
Three new components dictionary keys enhance apple-app-site-association patterns:

1. **`caseSensitive: false`** — Enables case-insensitive matching for path components (available since macOS 10.15.5 / iOS 13.5)
2. **`percentEncoded: false`** — Patterns are matched as Unicode code points rather than percent-encoded ASCII; allows accented and non-ASCII characters directly in patterns (iOS 14 / macOS Big Sur)
3. **`defaults` dictionary** — Applies default values (e.g., `caseSensitive`, `percentEncoded`) to all patterns in scope; eliminates repetition. Sibling of `components` applies to that array; sibling of `details` applies to all links for the domain (iOS 14 / macOS Big Sur)

### Substitution Variables
Named lists of strings embedded in pattern paths, referenced as `$(variableName)`:
- Defined in a `substitutionVariables` dictionary under `applinks`
- Values support `?` and `*` wildcards but cannot reference other variables
- Variable names are case-sensitive; values follow the surrounding `caseSensitive` setting
- **Built-in variables**: `$(alpha)`, `$(upper)`, `$(lower)`, `$(alnum)`, `$(digit)`, `$(xdigit)`, `$(region)` (all ISO region codes), `$(lang)` (all ISO language codes)
- Available since macOS 10.15.6 / iOS 13.5

### Apple CDN for Associated Domains
Beginning with iOS 14 / macOS Big Sur, the system fetches apple-app-site-association files via Apple's dedicated CDN rather than directly from the app's web server at install time. The CDN:
- Pre-fetches all associated domain files concurrently at install time
- Caches data, reducing server load from millions to a handful of requests per day
- Uses a single HTTP/2 connection for all domain requests
- Improves reliability by eliminating failures due to server unavailability

Web servers will only receive requests from Apple's CDN (public Internet). Two alternate modes bypass the CDN for special cases:

- **Developer mode** — For pre-deployment testing; allows self-signed or non-trusted SSL certificates; enabled per device (iOS: Settings > Developer > Associated Domains Development; macOS: `swcutil developer-mode -e true`); only applies to development-signed builds
- **Managed mode** — For MDM-enrolled enterprise devices; similar bypass for internal servers not reachable from the public Internet

Both modes require hosting the apple-app-site-association file in the `.well-known/` directory. Configured via mode query parameter in the Associated Domains entitlement: `applinks:internal.example.com?mode=developer`, `?mode=managed`, or `?mode=developer+managed`.

## APIs & Frameworks

### WatchKit (watchOS)
- `WKExtensionDelegate.handle(_ userActivity: NSUserActivity)` — handle incoming universal links
- `WKExtension.shared().openSystemURL(_ url: URL)` — open a universal link in another app

### UIKit (iOS/iPadOS/Mac Catalyst)
- `UIApplicationDelegate.application(_:continue:restorationHandler:)` — existing handler for universal links
- `UISceneDelegate.scene(_:continue:)` — required when using `UIScene`
- `UIApplication.shared.open(_:options:completionHandler:)` — open a universal link

### AppKit (macOS)
- `NSApplicationDelegate.application(_:continue:restorationHandler:)` — handle incoming universal links

### SwiftUI (all platforms)
- `onOpenURL(perform:)` modifier **[NEW]** — declarative universal link handler

### Associated Domains / apple-app-site-association
- `Associated Domains` entitlement — lists domains; now supports `?mode=developer` and `?mode=managed` query parameters **[NEW]**
- `apple-app-site-association` JSON file — `applinks.details[].components` array
  - `caseSensitive` key **[NEW — iOS 13.5+]** — `false` for case-insensitive matching
  - `percentEncoded` key **[NEW — iOS 14+]** — `false` for Unicode pattern matching
  - `defaults` key **[NEW — iOS 14+]** — default values for all patterns in scope
  - `substitutionVariables` key **[NEW — iOS 13.5+]** — named match string lists under `applinks`
  - Built-in substitution variables: `$(alpha)`, `$(upper)`, `$(lower)`, `$(alnum)`, `$(digit)`, `$(xdigit)`, `$(region)`, `$(lang)` **[NEW]**
- `swcutil developer-mode -e true` — CLI command to enable developer mode on macOS **[NEW]**

## Code Highlights

WatchKit universal link handling:
```swift
// WKExtensionDelegate
func handle(_ userActivity: NSUserActivity) {
    // Handle incoming universal link (same body as iOS delegate)
}
// Open a link in another app:
WKExtension.shared().openSystemURL(url)
```

SwiftUI universal link handler (all platforms):
```swift
ContentView()
    .onOpenURL { url in
        // Handle the incoming URL
    }
```

apple-app-site-association with substitution variables and Unicode:
```json
{
  "applinks": {
    "substitutionVariables": {
      "food": ["soup", "salad", "sandwich", "pizza"]
    },
    "defaults": { "caseSensitive": false, "percentEncoded": false },
    "details": [{
      "appIDs": ["TEAMID.com.example.FoodApp"],
      "components": [
        { "/": "/CA/$(lang)_CA/$(food)", "exclude": true },
        { "/": "/$(region)/$(lang)_$(region)/$(food)" }
      ]
    }]
  }
}
```

Associated Domains entitlement with alternate modes:
```
applinks:www.example.com
applinks:internal.example.com?mode=developer
applinks:intranet.example.com?mode=managed
applinks:staging.example.com?mode=developer+managed
```

## Takeaways

- Substitution variables eliminate combinatorial explosion in apple-app-site-association patterns; built-in `$(region)` and `$(lang)` variables cover ISO codes for all regions and languages.
- The Apple CDN for Associated Domains makes universal link activation reliable at install time — servers no longer need to be reachable at the moment a user installs an app.
- Developer mode allows testing against internal, self-signed servers without going through the CDN; enabled per-device and only applies to development-signed builds.
- Universal links now work natively on watchOS 7 and via the `onOpenURL` modifier in SwiftUI on all platforms.

---
_Source: WWDC20 Session 10098 page (abstract, transcript, and resource links)._
