# Deep Linking on tvOS
**WWDC17 · Session 246** · [Watch](https://developer.apple.com/videos/play/wwdc2017/246/)

_Platforms:_ tvOS 11

## Overview
This session covers best practices for implementing deep links in tvOS apps — URLs that route the system directly to a specific piece of content within an app, bypassing manual navigation. Deep links arrive from multiple system entry points: the Top Shelf extension, Universal Links shared from iOS, Siri suggestions, and other applications. The session explains how to implement them correctly in both UIKit and TVMLKit apps so users always land exactly where they expect and can navigate back predictably.

The core principle is immediacy: the linked content must be visible the moment the system transitions to the app, with no loading alerts, confirmation dialogs, or animated transitions. The session distinguishes between two URL types exposed by Top Shelf items — `displayURL` (shows a detail page) and `playURL` (begins video playback immediately) — and demonstrates the correct navigation stack to build for each, so the Menu button always behaves intuitively.

A sample app (`tvOS Deep Linking Demo`) is demonstrated live, showing how a `UINavigationController`-based app handles both URL types by popping to root, pushing the detail controller, and optionally pushing a player controller on top, all without animation.

## Key Topics

- **What is a deep link** — a URL the app knows how to open, pointing to content several levels below the top-level screen; avoids manual navigation.
- **Sources of deep links on tvOS** — Top Shelf extension, Universal Links, other apps, Siri, and system search results.
- **Universal Links vs. custom URL schemes** — Universal Links are recommended because they guarantee routing to the correct app and data format; custom URL schemes can be intercepted by other apps.
- **displayURL vs. playURL** — Top Shelf items expose both; `displayURL` (Select press) → detail page; `playURL` (Play press) → immediate video playback, black fade-in, no visible transitions.
- **Navigation stack design for deep links** — the correct hierarchy when a deep link is opened: detail page (or player) on top, app's main screen below; first Menu press → main screen; second Menu press → tvOS Home Screen.
- **No animation on launch** — reconfigure the UI before the system switch is complete so content appears instantly; use `UIView.setAnimationsEnabled(false)` or equivalent.
- **Handling in `application(_:open:options:)`** — the recommended implementation: disable animations, pop to root, push detail controller, optionally push player controller, all via the same segues used during normal in-app navigation.
- **UIKit vs. TVMLKit** — same concepts apply; session focuses on UIKit but notes TVMLKit apps follow the same patterns.

## APIs & Frameworks

**UIKit / tvOS**
- `UIApplicationDelegate.application(_:open:options:)` — entry point for all deep link URLs
- `UINavigationController.popToRootViewController(animated:)` — reset navigation stack without animation on deep link open
- `UIViewController.performSegue(withIdentifier:sender:)` — reuse existing segues to push detail and player controllers
- `UIView.setAnimationsEnabled(_:)` — disable animations during deep link UI reconfiguration

**Top Shelf Extension (TVServices)**
- `TVContentItem.displayURL` — URL opened on Select press; should show content detail page
- `TVContentItem.playURL` — URL opened on Play press; should immediately start video playback
- `TVTopShelfContentProvider` — provides items to the tvOS Top Shelf

**Universal Links**
- Associated Domains entitlement (`webcredentials`, `applinks`) — required for Universal Links
- `apple-app-site-association` file — hosted on the web server to map URLs to the app
- `NSUserActivity` with `webpageURL` — carries the URL into the app when opened via Universal Link

**Sample Code**
- `tvOS Deep Linking Demo` — demonstrates both displayURL and playURL handling with UINavigationController

## Code Highlights

Core deep-link handler in `UIApplicationDelegate`:
```swift
func application(_ app: UIApplication,
                 open url: URL,
                 options: [UIApplicationOpenURLOptionsKey: Any] = [:]) -> Bool {
    // 1. Disable animations so content appears immediately
    UIView.setAnimationsEnabled(false)

    // 2. Pop to root (clear any previous navigation state)
    navigationController.popToRootViewController(animated: false)

    // 3. Push the detail controller (always, even for playURL)
    navigationController.performSegue(withIdentifier: "showDetail", sender: url)

    // 4. If this is a playURL, also push the player on top
    if url == playURL {
        navigationController.performSegue(withIdentifier: "startPlayback", sender: url)
    }

    UIView.setAnimationsEnabled(true)
    return true
}
```

## Takeaways

- Use Universal Links rather than custom URL schemes for tvOS deep links — they guarantee correct routing and data integrity.
- Always pop to root and push the new content stack without animation so content is already in place when the system reveals the app.
- Build a two-level stack for every deep link (detail page + main screen) so the Menu button behaves predictably: one press back to main, two presses back to the Home Screen.
- Distinguish displayURL (detail page) from playURL (immediate playback) and implement both; the playURL stack still has the detail page underneath so Menu-from-playback works correctly.

---
_Source: WWDC17 Session 246 page (abstract, transcript, and resource links)._
