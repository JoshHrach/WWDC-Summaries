# Extend Your App's Presence with Deep Linking
**WWDC17 · Session 250** · [Watch](https://developer.apple.com/videos/play/wwdc2017/250/)

_Platforms:_ iOS 11

## Overview
This session explains what deep links are, why they matter for discoverability and engagement, and how Universal Links are the correct implementation vehicle on iOS. The session frames deep linking as a way to skip the normal user journey — browse → navigate → find content — by launching an app directly to a specific piece of content or triggering a specific function automatically. It then catalogs every major surface in iOS where Universal Links can appear, from social media shares to Spotlight search to Siri App Suggestions to Lock Screen widgets to the TV app.

The session is a companion to "Deep Linking on tvOS" (session 246) and is cross-referenced by that session's navigation stack design guidance. Where session 246 focuses on navigation UX after launch, session 250 focuses on the infrastructure and entry points that produce the launch.

## Key Topics

- **What deep links solve** — apps contain content (videos, photos, places) and functions (watch, navigate, share). Normally users must know the content exists and then navigate to it. Deep links bypass both steps: the app launches directly to the target content or executes a function automatically.
- **Universal Links format** — an `https://` URL using the app's own domain and a path to the resource. Format: `https://<domain>/<path>`. Because the scheme is `https`, the link works as a normal web URL if the app is not installed — Safari handles it as a fallback.
- **Setup steps** — four required steps:
  1. Add an Associated Domains entitlement listing every web domain the app handles (`applinks:<domain>`).
  2. Create an `apple-app-site-association` (AASA) JSON file listing the URL patterns the app can handle.
  3. Upload the AASA file to the web server at `https://<domain>/.well-known/apple-app-site-association` (or the root).
  4. Implement `application(_:continue:restorationHandler:)` in the App Delegate; the passed `NSUserActivity` carries the Universal Link URL.
- **Launch animation guidance** — when launching from a Universal Link, use animated transitions to orient the user within the app hierarchy. This reminds users where they are and makes the back-navigation path discoverable. (For the TV app specifically, no animation is necessary because the system pre-renders the video player interface.)
- **Discoverability entry points** — Universal Links appear in:
  - Social media posts (user-shared links)
  - Messages and email (user-shared links with rich Open Graph preview)
  - Website links from other sites
  - Spotlight / Safari search index results
  - Siri App Suggestions (requires registered `NSUserActivity` with `isEligibleForSearch` and `isEligibleForPrediction`)
  - Handoff (requires registered `NSUserActivity` with `isEligibleForHandoff`)
  - Lock Screen widgets and Notification actions
  - Home screen Quick Actions (`UIApplicationShortcutItem`)
  - SiriKit intent responses (map intent results to Universal Links)
  - Other apps that link to your content
  - TV app integration (requires `displayURL` and `playURL` per session 246)
- **Fallback to Safari** — if the app is not installed, the Universal Link opens in Safari. The page can show an App Store Smart App Banner (`<meta name="apple-itunes-app">`) to guide the recipient to install the app.
- **Open Graph metadata** — add Open Graph `<meta>` tags (`og:title`, `og:image`, `og:description`) to the linked web page. When the Universal Link is shared in Messages, iMessage renders the link as a rich card with an image and title rather than a plain URL. This dramatically increases tap-through rate.

## APIs & Frameworks

**Core URL Handling**
- `UIApplicationDelegate.application(_:continue:restorationHandler:)` — receives the `NSUserActivity` containing the Universal Link URL when the app is launched or brought to foreground via a Universal Link
- `NSUserActivity` — carries the `webpageURL` property with the incoming Universal Link
- `NSUserActivity.isEligibleForSearch` — opt activity into Spotlight indexing
- `NSUserActivity.isEligibleForPrediction` — opt activity into Siri App Suggestions
- `NSUserActivity.isEligibleForHandoff` — opt activity into Handoff

**Entitlement and Server Files**
- Associated Domains entitlement — `com.apple.developer.associated-domains` with `applinks:<domain>` entries
- `apple-app-site-association` JSON file — declares path patterns the app handles; hosted on the web server

**Sharing**
- `UIActivityViewController` — launches the system share sheet; accepts any object (string, image, URL); Universal Link should be the primary shareable item

**App Icon Quick Actions**
- `UIApplicationShortcutItem` — Home screen 3D Touch quick action; can launch directly to a deep-linked screen
- `UIApplicationShortcutIcon` — glyph for the quick action item

**TV App**
- `displayURL` — Universal Link that opens the content detail view (with animation)
- `playURL` — Universal Link that immediately begins playback (no animation per system pre-rendering)

**Smart App Banners (web)**
- `<meta name="apple-itunes-app" content="app-id=<ID>, app-argument=<url>">` — renders an App Store banner in Safari for non-app users visiting a Universal Link URL

## Code Highlights

No code samples were presented. The session is a conceptual overview with implementation steps described verbally.

Key delegate method invoked for all Universal Link launches:

```swift
func application(_ application: UIApplication,
                 continue userActivity: NSUserActivity,
                 restorationHandler: @escaping ([UIUserActivityRestoring]?) -> Void) -> Bool {
    guard userActivity.activityType == NSUserActivityTypeBrowsingWeb,
          let url = userActivity.webpageURL else { return false }
    // Parse url.path and navigate to the appropriate content
    handleDeepLink(url)
    return true
}
```

## Takeaways

- Create Universal Links for every piece of content and every function in your app that has sharing or discoverability value — even if you don't initially surface share buttons everywhere, having the infrastructure in place costs nothing and enables future entry points.
- The AASA file must be served over HTTPS with no redirects and must be reachable without authentication; Apple's servers fetch it during app installation, not at runtime.
- Always include a web fallback page at the Universal Link URL with an App Store Smart App Banner and Open Graph metadata; this converts app-less recipients into new users.
- Add `NSUserActivity` with `isEligibleForPrediction = true` for the most important in-app destinations to surface them in Siri App Suggestions and Spotlight; this is one of the highest-leverage discoverability investments available on iOS.

---
_Source: WWDC17 Session 250 page (abstract, transcript, and resource links)._
