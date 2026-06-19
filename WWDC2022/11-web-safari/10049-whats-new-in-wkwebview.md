# What's new in WKWebView
**WWDC22 · Session 10049** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10049/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13

## Overview
WKWebView gained four significant new capabilities in iOS 16 / iPadOS 16: JavaScript Fullscreen API support, new CSS dynamic viewport units (`svh`, `lvh`, `dvh`), built-in Find interaction, and an enhanced content blocking rule (`if-frame-url`). iPadOS 16 also adds support for Encrypted Media Extensions (EME) and Media Source Extensions (MSE) for premium DRM-protected content. Apps with the `com.apple.developer.web-browser` entitlement now get Remote Web Inspector support automatically.

The session also reiterates that `UIWebView` is deprecated and will be removed in a future release; migration to `WKWebView` is required.

## Key Topics

### JavaScript Fullscreen API
Web content can now enter fullscreen (e.g., via `element.webkitRequestFullscreen()`) within a WKWebView when `WKPreferences.isElementFullscreenEnabled = true`. Apps can observe `WKWebView.fullscreenState` (a KVO-observable property) to customize transitions when content enters or exits fullscreen.

### CSS Dynamic Viewport Units
New CSS length units `svh` (small viewport height), `lvh` (large viewport height), `dvh` (dynamic viewport height), and related units (`svw`, `lvw`, `dvw`, `svb`, `lvb`, `dvb`, `svi`, `lvi`, `dvi`, `svmin`, `lvmin`, `dvmin`, `svmax`, `lvmax`, `dvmax`) let web content adapt layout to viewport size changes caused by UI chrome (e.g., Safari's collapsing toolbar). Apps that change WKWebView viewport insets must call `setMinimumViewportInset(_:maximumViewportInset:)` **[NEW]** to provide the min/max inset range so WebKit can calculate these units correctly.

### Find Interaction
Setting `WKWebView.findInteractionEnabled = true` **[NEW]** adds a native system Find UI (same as UIKit's `UIFindInteraction`) to the web view. Users can invoke it via Command-F or the system edit menu. The `WKWebView.findInteraction` property exposes the underlying `UIFindInteraction` object for programmatic control (present/dismiss panel, navigate results).

### Content Blocking — if-frame-url
`WKContentRuleList` JSON rules gain a new `"if-frame-url"` trigger condition **[NEW]** — an array of regular expressions matched against the URL of the frame making the request (rather than the top-frame URL). This lets content blockers scope rules precisely to content within specific iframes.

### Encrypted Media (iPadOS 16 New)
WKWebView on iPadOS now supports Encrypted Media Extensions (EME) and Media Source Extensions (MSE), enabling DRM-protected premium video content (e.g., Apple TV+) in web apps on iPad.

### Remote Web Inspector
Apps with the `com.apple.developer.web-browser` entitlement automatically support Remote Web Inspector in production, identical to Safari on iOS. Requires Web Inspector enabled in Safari Settings on the device and the Develop menu enabled in Safari on macOS.

## APIs & Frameworks

**WebKit / WKWebView**
- `WKPreferences.isElementFullscreenEnabled` **[NEW]** — enables JS Fullscreen API
- `WKWebView.fullscreenState` **[NEW]** — KVO-observable fullscreen state (`notInFullscreen`, `enteringFullscreen`, `inFullscreen`, `exitingFullscreen`)
- `WKWebView.setMinimumViewportInset(_:maximumViewportInset:)` **[NEW]** — defines viewport inset range for dynamic CSS viewport units
- `WKWebView.findInteractionEnabled` **[NEW]** — enables system Find UI
- `WKWebView.findInteraction` **[NEW]** — exposes `UIFindInteraction` object
- `UIFindInteraction.presentFindNavigator(showingReplace:)` — show find panel
- `UIFindInteraction.dismissFindNavigator()` — hide find panel
- `WKContentRuleList` — content blocking rule list
- `WKContentRuleListStore.compileContentRuleList(forIdentifier:encodedContentRuleList:completionHandler:)` — compile JSON rules
- `WKWebViewConfiguration.userContentController` — attach rule lists
- `WKUserContentController.add(_ contentRuleList:)` — register blocking rules
- `WKUserContentController.removeContentRuleList(_:)` — remove rules

**Content Blocking JSON (New Trigger Field)**
- `"if-frame-url"` **[NEW]** — array of regexes matched against the requesting frame's URL

**CSS (New Units — Web Standard, now supported in WebKit)**
- `svh`, `svw`, `svb`, `svi`, `svmin`, `svmax` — small viewport units **[NEW]**
- `lvh`, `lvw`, `lvb`, `lvi`, `lvmin`, `lvmax` — large viewport units **[NEW]**
- `dvh`, `dvw`, `dvb`, `dvi`, `dvmin`, `dvmax` — dynamic viewport units **[NEW]**

**Web APIs (JavaScript)**
- `element.webkitRequestFullscreen()` — request fullscreen
- `document.webkitExitFullscreen()` — exit fullscreen

## Code Highlights

Enable fullscreen and observe state:
```swift
webView.configuration.preferences.isElementFullscreenEnabled = true

let observation = webView.observe(\.fullscreenState, options: [.new]) { webView, _ in
    print("fullscreenState: \(webView.fullscreenState)")
}
```

Set viewport inset range for dynamic CSS units:
```swift
let minimum = UIEdgeInsets(top: 0, left: 0, bottom: 30, right: 0)
let maximum = UIEdgeInsets(top: 0, left: 0, bottom: 200, right: 0)
webView.setMinimumViewportInset(minimum, maximumViewportInset: maximum)
```

Enable Find interaction:
```swift
webView.findInteractionEnabled = true
// Programmatically present:
webView.findInteraction?.presentFindNavigator(showingReplace: false)
```

Content blocking rule scoped to an iframe's URL:
```json
[{
    "action": {"type": "block"},
    "trigger": {
        "resource-type": ["image"],
        "url-filter": ".*",
        "if-frame-url": ["https?://([^/]*\\.)?wikipedia.org/"]
    }
}]
```

## Takeaways
- `UIWebView` is deprecated and will be removed; migrate to `WKWebView` now.
- Setting `isElementFullscreenEnabled = true` is all that's needed to support JavaScript fullscreen within a WKWebView; observe `fullscreenState` for custom transitions.
- `findInteractionEnabled = true` adds a complete system Find UI to any WKWebView in one line.
- The new `"if-frame-url"` content blocking trigger enables precise per-iframe blocking rules for content blocker implementations.

---
_Source: WWDC22 Session 10049 page (abstract, chapter summaries, code samples, and resource links)._
