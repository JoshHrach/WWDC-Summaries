# What's New in Universal Links
**WWDC19 · Session 717** · [Watch](https://developer.apple.com/videos/play/wwdc2019/717/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15, tvOS 13

## Overview
Universal links (HTTP/HTTPS URLs that open native app content when the app is installed, or fall back gracefully to the web) receive two major updates in 2019: a new `components`-based URL matching syntax that adds query and fragment matching, and the arrival of universal links on macOS Catalina for both AppKit and UIKit (Catalyst) apps.

The session also covers important changes to the `apple-app-site-association` (AASA) file format (deprecated signing and old paths), download prioritization changes, best practices for pattern matching internationalized URLs, and guidance on handling universal links gracefully in all edge cases.

## Key Topics

### Universal Links Now on macOS **[NEW]**
- Universal links work on macOS Catalina 10.15 for AppKit apps, UIKit (Catalyst) apps, and in Safari.
- First-time flow on macOS: link opens in Safari; if the app is installed, Safari shows a banner; user can choose to open in the app, and future navigations go directly there.
- App must be installed locally (remote-volume apps cannot use universal links on macOS).
- App Store apps: AASA downloads begin as soon as the app is installed. Developer ID apps: AASA downloads begin after the first launch.
- Only one copy of a given app can handle universal links on a Mac (typically the one in `/Applications`).

### apple-app-site-association File Changes
- Canonical location: `https://yourdomain/.well-known/apple-app-site-association` — other paths deprecated.
- Signed AASA files deprecated; signing was never required and will be removed in a future release.
- `apps` key at top level: no longer needed for iOS 13 / tvOS 13 / macOS 10.15 targets; still required for older OS support (must be an empty array for universal links).
- `details` key: was a dictionary, now must be an array of dictionaries.
- `appID` → can now use `appIDs` (plural) **[NEW]** when targeting current releases to share one config among multiple app identifiers.

### New `components` Key Replaces `paths` **[NEW]**
- `paths` key is deprecated (still supported; iOS 13 ignores it when `components` is present).
- `components` is an array of dictionaries, each specifying URL components to match:
  - `/` — path component (same wildcard syntax as terminal: `*` = zero or more characters, `?` = exactly one character)
  - `?` — query component or query-item dictionary
  - `#` — fragment component
- When the value of `?` is a dictionary, each key is a query item name and the value is the pattern for that item's value. All specified query items must match; unspecified components are ignored.
- Query items with no value or absent from the URL are treated as having a value equal to the empty string.
- `exclude: true` — mark a components entry as excluded (replaces `NOT` keyword from paths); system stops checking for a match and does not open as a universal link.
- For a components dictionary to match, all specified components must match.

### Pattern Matching Internationalization
- URL matching is done in ASCII (URLs are always percent-encoded ASCII); Unicode characters must be percent-encoded.
- A Unicode character may expand to multiple ASCII characters; use `?` carefully — it matches exactly one character.
- Country-code patterns like two-letter codes can be simplified to `??` to reduce AASA file size.
- Country code TLDs (ccTLDs) and internationalized TLDs are download-prioritized when they match the user's locale settings.
- Top-level domains `.com`, `.net`, `.org` are always high priority.

### App Entitlement and Association
- Add the `Associated Domains` capability in Xcode target settings; generates the `com.apple.developer.associated-domains` entitlement.
- Entitlement values: array of strings in the form `applinks:<domain>` for universal links; `applinks:*.example.com` for all subdomains (wildcard).
- Exact domain entries have higher priority than wildcard entries during pattern matching.
- Internationalized domain names must be Punycode-encoded (RFC 3492).

### Handling Incoming Universal Links in Code
- App delegate method: `application(_:continue:restorationHandler:)` (UIKit) or `application(_:continue:restorationHandler:)` (AppKit, with `NS` prefix).
- If using UIScene: `scene(_:continue:)` on `UISceneDelegate`.
- Check `userActivity.activityType == NSUserActivityTypeBrowsingWeb`.
- Retrieve `userActivity.webpageURL` — never nil for a universal link.
- Parse URL using `URLComponents` (not regex or string parsing) to avoid security issues.
- Open the universal link in-app with `UIApplication.shared.open(_:)` / `NSWorkspace.shared.open(_:)` / Launch Services.
- To require opening in an app (not browser): use the appropriate `open` API; if it fails, no universal link was available.
- Fail gracefully: if a URL cannot be handled (content outdated/invalid), open in `SFSafariViewController`, Safari, or show an error prompt.
- Use `Smart App Banner` meta tag on web pages to integrate App Store / content links without custom URL schemes.

### Deprecations and Security Guidance
- Custom URL schemes are inherently insecure and highly discouraged for new development; migrate to universal links.
- Signed AASA files deprecated.
- Paths other than `/.well-known/apple-app-site-association` deprecated.
- Old dictionary-format `details` value (instead of array) deprecated.

## APIs & Frameworks

### Foundation
- `NSUserActivity` — carries universal link information
- `NSUserActivity.activityType` — check for `NSUserActivityTypeBrowsingWeb`
- `NSUserActivity.webpageURL: URL?` — the universal link URL
- `URLComponents` — parse the URL safely

### UIKit
- `UIApplicationDelegate.application(_:continue:restorationHandler:)` — handle incoming universal link
- `UISceneDelegate.scene(_:continue:)` — scene-based alternative
- `UIApplication.shared.open(_:options:completionHandler:)` — open a universal link (will route to app if available)

### AppKit / macOS
- `NSApplicationDelegate.application(_:continue:restorationHandler:)` (NSUserActivity variant) — handle incoming link on macOS
- `NSWorkspace.shared.open(_:)` — open a universal link from a macOS app

### SafariServices
- `SFSafariViewController` — graceful fallback for universal links the app cannot handle

### Xcode / Entitlements
- `com.apple.developer.associated-domains` entitlement — array of `applinks:<domain>` strings
- Associated Domains capability in Xcode project settings

### apple-app-site-association JSON Keys
- Top level: `applinks` dictionary
- `applinks.details` — array of per-app configuration dictionaries **[UPDATED format]**
- `details[n].appID` — single app identifier string
- `details[n].appIDs` **[NEW]** — array of app identifier strings (iOS 13+ / macOS 10.15+)
- `details[n].components` **[NEW]** — array of component-matching dictionaries
- `components[n]["/"]` — path pattern
- `components[n]["?"]` — query pattern (string or dictionary of query item patterns)
- `components[n]["#"]` — fragment pattern
- `components[n]["exclude"]` — boolean, exclude matching URLs from universal link handling

## Code Highlights

App delegate handling incoming universal link:
```swift
func application(_ application: UIApplication,
                 continue userActivity: NSUserActivity,
                 restorationHandler: @escaping ([UIUserActivityRestoring]?) -> Void) -> Bool {
    guard userActivity.activityType == NSUserActivityTypeBrowsingWeb,
          let url = userActivity.webpageURL,
          let components = URLComponents(url: url, resolvingAgainstBaseURL: true) else {
        return false
    }
    // Route to content based on path and query items
    let queryItems = components.queryItemsValueDictionary  // your helper
    if components.path.hasPrefix("/order/") {
        openOrderView(for: components, queryItems: queryItems)
        return true
    }
    // Can't handle — fall back to Safari
    let safariVC = SFSafariViewController(url: url)
    present(safariVC, animated: true)
    return true
}
```

AASA file with new `components` syntax:
```json
{
  "applinks": {
    "details": [{
      "appIDs": ["ABCDE12345.com.example.MyApp"],
      "components": [
        { "/": "/*/order", "comment": "All order pages" },
        { "/": "/taco", "?": { "cheese": "?*" }, "comment": "Tacos with cheese" },
        { "/": "/coupon/1*", "exclude": true, "comment": "Exclude coupons starting with 1" },
        { "/": "/coupon/????", "comment": "All other 4-digit coupons" }
      ]
    }]
  }
}
```

## Takeaways
- Universal links now work on macOS Catalina — add the Associated Domains entitlement to your AppKit or Catalyst app and update your AASA file with the new format.
- The new `components` key replaces `paths` and adds query item and fragment matching, making URL pattern matching more precise and expressive.
- Signed AASA files and non-`.well-known` paths are deprecated; update your server configuration before they are removed.
- Always parse incoming URLs with `URLComponents` (not string manipulation) and always fail gracefully with an `SFSafariViewController` fallback or Safari open.

---
_Source: WWDC19 Session 717 page (abstract, chapter summaries, code samples, and resource links)._
