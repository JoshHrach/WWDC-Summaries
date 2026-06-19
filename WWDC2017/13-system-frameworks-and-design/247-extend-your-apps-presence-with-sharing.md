# Extend Your App's Presence With Sharing
**WWDC17 · Session 247** · [Watch](https://developer.apple.com/videos/play/wwdc2017/247/)

_Platforms:_ iOS 11

## Overview
This session covers how to design and implement a sharing experience in an iOS app that drives both engagement from existing users and discovery by new users. The session is organized around three decisions every developer must make before adding a share button: what content to share, where to place the share entry point, and when in the user experience to surface it. The session uses Crossy Road as its primary example, showing how the game identified a naturally shareable moment (the comical losing screen), created a photograph representation of that moment, and surfaced sharing immediately at that instant.

The session is a companion to "Extend Your App's Presence with Deep Linking" (session 250) and "Deep Linking on tvOS" (session 246). Together the three sessions form a complete picture of how to expose app content externally through Universal Links.

## Key Topics

- **Content: what to share** — identify experiences in the app that users will want to show others. For Crossy Road, losing is funny and photo-worthy; the developers generated a photograph representation of the moment and pre-staged it for sharing. Content should be something the recipient will find immediately valuable without context that only the sender has.
- **Placement: where the share entry point lives** — share buttons do not belong in list views where the user has not yet committed to a specific item. On a detail view (single session, single article, single photo), a share button in the navigation bar title area communicates that the entire view's content is shareable. Place the share entry point where it is unambiguous what will be shared.
- **Timing: when to surface sharing** — the optimal sharing moment is when the user is most emotionally connected to the content — the moment they think "my friend would love this." For games, this may be a victory screen or a defeat screen. For productivity apps, it may be when a task is completed. For media apps, it may be after a rating action.
- **UIActivityViewController as the universal mechanism** — the system share sheet is the correct implementation for sharing on iOS. It automatically includes system targets (Messages, Mail, Notes) and populates additional targets from apps that have implemented a share extension. Initialize it with any combination of shareable objects (strings, images, URLs).
- **Universal Links as the shareable item** — the best item to put in the share sheet is a Universal Link (an `https://` URL) rather than a raw string or a custom-scheme URL. This ensures the link works for recipients who do not have the app installed — they see the web fallback — and the link launches the app directly for recipients who do have it installed.
- **App Store Smart App Banner** — when a Universal Link is opened in Safari by a recipient without the app, a Smart App Banner (`<meta name="apple-itunes-app">`) at the top of the page promotes the app and links directly to the App Store. This converts shares into new installs.
- **Open Graph metadata for rich iMessage previews** — when a Universal Link is shared in iMessage, the system generates a link preview card. Without Open Graph metadata the card shows only the raw URL. With `og:title`, `og:image`, and `og:description` set on the target web page, the card shows an image thumbnail and title, dramatically increasing recipient engagement. This is one of the highest-leverage improvements for sharing in iMessage.

## APIs & Frameworks

**UIKit**
- `UIActivityViewController` — system share sheet; initialize with an array of activity items (any combination of `String`, `UIImage`, `URL`, or objects conforming to `UIActivityItemSource`); set `excludedActivityTypes` to suppress irrelevant activities
- `UIActivityItemSource` — protocol for supplying different representations of the same content to different sharing targets (e.g., a URL for Messages, a string for Mail subject)
- `UIBarButtonItem(barButtonSystemItem: .action, ...)` — the standard action button (share icon / "sharrow") placed in the navigation bar; tapping it presents `UIActivityViewController`

**Foundation / Web Infrastructure**
- `URL` (Universal Link) — the primary object passed to `UIActivityViewController`; must be an `https://` URL pointing to the app's own domain
- `apple-app-site-association` JSON file — maps URL paths to the app (required for Universal Link support; see session 250)
- Associated Domains entitlement — `applinks:<domain>` entry required for Universal Link recognition

**Smart App Banner (HTML `<meta>` tag)**
- `<meta name="apple-itunes-app" content="app-id=<ID>, app-argument=<universal-link-url>">` — renders a banner in Mobile Safari on the fallback web page; tapping it opens the App Store and pre-populates the install with the deep-link URL so the user lands in the right place after installation

**Open Graph (HTML `<meta>` tags)**
- `og:title` — title shown in the iMessage link preview card
- `og:image` — image shown in the iMessage link preview card
- `og:description` — description shown in the iMessage link preview card

## Code Highlights

Presenting the share sheet with a Universal Link:

```swift
@IBAction func shareButtonTapped(_ sender: UIBarButtonItem) {
    let contentURL = URL(string: "https://example.com/content/42")!
    let activityVC = UIActivityViewController(
        activityItems: [contentURL],
        applicationActivities: nil)
    // On iPad, anchor the popover to the bar button
    activityVC.popoverPresentationController?.barButtonItem = sender
    present(activityVC, animated: true)
}
```

## Takeaways

- Choose one sharing moment per major user workflow and optimize it fully (timing, placement, content representation) rather than adding generic share buttons to every screen.
- Always share a Universal Link rather than a plain URL or a custom-scheme URL; the `https://` scheme ensures the link degrades gracefully to Safari for recipients without the app.
- Add Open Graph metadata to every page that can receive a Universal Link; the richness of the iMessage preview card has a measurable effect on whether recipients tap the link.
- Place Smart App Banners on all Universal Link landing pages; a share from an existing user is the highest-quality mechanism for acquiring a new user with immediate intent.

---
_Source: WWDC17 Session 247 page (abstract, transcript, and resource links)._
