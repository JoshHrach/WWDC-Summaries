# Introducing Desktop-class Browsing on iPad
**WWDC19 · Session 203** · [Watch](https://developer.apple.com/videos/play/wwdc2019/203/)

_Platforms:_ iPadOS 13, iOS 13; WebKit / Safari

## Overview
iPadOS 13 introduces desktop-class browsing as a fundamental platform change to Safari and WebKit. iPad now presents itself to websites as a Mac (desktop user agent), viewports match the physical screen size of each iPad model, and mouse hover events from desktop sites work out of the box via improved compatibility heuristics. These changes are not superficial — they represent deep WebKit engine changes to make the world's desktop websites work naturally on iPad with touch input.

The session covers both app developers (how to adopt new WKWebView APIs for content mode control) and web developers (new web standards and best practices for making websites shine on iPad). The core message for web developers is to build one responsive website rather than separate mobile and desktop versions, and to use feature detection instead of user agent sniffing.

The Shiny Browser and Shiny Sketch demos walk through concrete code changes: fixing a custom user agent, implementing per-site desktop/mobile switching, adopting Pointer Events, using the `touch-action` CSS property, and using media queries to conditionally show hover-dependent controls.

## Key Topics

**Desktop User Agent on iPad**
Safari on iPadOS now presents the Mac user agent to all websites by default. This is accompanied by viewport changes: viewports now match each iPad's actual screen size (larger iPads get more layout real estate), while smaller iPads and narrow Split View panes fall back to mobile scaling.

**App Developer: WKWebView Content Mode API**
- `WKWebView` built against iOS 13 SDK automatically opts into desktop mode on iPad.
- Use `WKWebViewConfiguration.applicationNameForUserAgent` instead of `WKWebView.customUserAgent` to add app-specific user agent tokens; WebKit fills in the rest correctly.
- New `WKWebpagePreferences` class controls preferred content mode (`.recommended`, `.mobile`, `.desktop`).
- New `WKNavigationDelegate` method `decidePolicyFor(navigationAction:preferences:decisionHandler:)` lets apps set per-navigation (per-site) content mode preferences.
- `WKNavigation.effectiveContentMode` — new property to read the actual content mode used after a navigation commits.
- `Safari View Controller` and `ASWebAuthenticationSession` handle desktop browsing automatically — no code changes needed for these APIs.

**Pointer Events**
WebKit now supports the W3C Pointer Events standard on iOS 13 and macOS Catalina. Pointer Events provide a unified abstraction over mouse, touch, and Apple Pencil input. Adoption is straightforward: the `PointerEvent` object inherits from `MouseEvent`, so existing handler logic works unchanged. Use feature detection with `window.PointerEvent` to register pointer event listeners and fall back to mouse events for older clients.

**Touch and Hover Compatibility**
- WebKit sends compatibility mouse events (mousedown, mouseup, click, hover) on tap for desktop sites.
- Improved hover detection in iOS 13: WebKit detects many more "meaningful changes" from hover events, delaying the click until meaningful hover content appears, then proceeding.
- `touch-action` CSS property is the correct way to prevent default browser scroll/zoom behaviors in touch contexts; `preventDefault()` alone is insufficient on iOS.

**Hardware-Accelerated Scrolling Everywhere**
Subframes and `overflow: scroll` regions now have hardware-accelerated scrolling on iPad by default. `-webkit-overflow-scrolling: touch` is now a no-op on iPad; JavaScript scroll-emulation libraries are no longer needed.

**Viewport and Text Sizing**
- WebKit now ignores the `width=device-width` viewport meta tag promise if the page actually lays out wider than the device width, scaling it to fit instead.
- Add `shrink-to-fit=no` to the viewport meta tag for websites intentionally designed to scroll horizontally.
- WebKit automatically boosts text size on pages that were shrunk to fit.

**Visual Viewport API**
New W3C Visual Viewport API (`window.visualViewport`) fires `resize` events when the on-screen keyboard appears/disappears and when Safari's smart search field collapses during scroll. This allows web apps to reposition fixed UI elements (e.g., buttons) that would otherwise be obscured by the keyboard.

**Media Source Extensions (MSE)**
MSE is now available for desktop sites on iPadOS for the first time, enabling existing desktop video streaming engines to work on iPad without modification. HLS remains the recommended approach.

## APIs & Frameworks

**WebKit / WKWebView** (iOS 13 / iPadOS 13)
- `WKWebViewConfiguration.applicationNameForUserAgent: String` (preferred over `customUserAgent`) **[NEW behavior]**
- `WKWebpagePreferences` **[NEW]**
  - `WKWebpagePreferences.preferredContentMode: WKWebpagePreferences.ContentMode` **[NEW]**
  - `WKWebpagePreferences.ContentMode` enum: `.recommended`, `.mobile`, `.desktop` **[NEW]**
- `WKNavigationDelegate.webView(_:decidePolicyFor:preferences:decisionHandler:)` **[NEW]**
- `WKNavigation.effectiveContentMode: WKWebpagePreferences.ContentMode` **[NEW]**
- `WKWebView.customUserAgent` (existing; now discouraged in favor of `applicationNameForUserAgent`)

**Safari View Controller / SFSafariViewController** (no changes required)
**ASWebAuthenticationSession** (no changes required; presents form sheet on iPad **[NEW behavior]**)

**Web Platform (iPadOS 13 / Safari)**
- Pointer Events API (`PointerEvent`, `pointerdown`, `pointermove`, `pointerup`) **[NEW]**
- `touch-action` CSS property (existing; now critical for pointer event adoption)
- `window.visualViewport` — Visual Viewport API **[NEW]**
- `window.visualViewport.addEventListener('resize', ...)` **[NEW]**
- `@media (hover: hover)` CSS media query — detect hover capability **[existing standard, now relevant]**
- `shrink-to-fit=no` viewport meta value (existing since iOS 9; now also applies to wide desktop sites)
- Media Source Extensions (MSE) for desktop sites **[NEW on iPadOS]**
- Hardware-accelerated scrolling for subframes and overflow regions **[NEW]**

## Code Highlights

Setting app name in user agent (preferred approach):
```swift
let config = WKWebViewConfiguration()
config.applicationNameForUserAgent = "Version/1.0 ShinyBrowser/1.0"
let webView = WKWebView(frame: .zero, configuration: config)
```

Per-site content mode via navigation delegate:
```swift
func webView(_ webView: WKWebView,
             decidePolicyFor navigationAction: WKNavigationAction,
             preferences: WKWebpagePreferences,
             decisionHandler: @escaping (WKNavigationActionPolicy, WKWebpagePreferences) -> Void) {
    if let host = navigationAction.request.url?.host,
       let mode = contentModeForHost[host] {
        preferences.preferredContentMode = mode
    }
    decisionHandler(.allow, preferences)
}
```

Adopting Pointer Events with mouse fallback:
```javascript
if (window.PointerEvent) {
    canvas.addEventListener('pointermove', updateInteraction)
} else {
    canvas.addEventListener('mousemove', updateInteraction)
}
```

Conditionally hiding hover-dependent controls:
```css
@media (hover: hover) {
    .static-control { display: none; }
}
```

Visual viewport resize listener:
```javascript
window.visualViewport.addEventListener('resize', () => {
    donateButton.style.bottom = `${window.innerHeight - window.visualViewport.height}px`
})
```

## Takeaways
- iPad now presents a desktop user agent by default; apps built against iOS 13 SDK get desktop `WKWebView` content automatically — test existing WKWebView-based apps for layout breakage.
- Use `WKWebpagePreferences` and the new navigation delegate method to implement per-site mobile/desktop switching; never use `customUserAgent` for the entire user agent string.
- Web developers should adopt Pointer Events instead of relying on mouse events, use `touch-action` CSS for scroll-safe drawing canvases, and use `@media (hover)` to conditionally show hover-dependent UI.
- Build one responsive website using feature detection; the expanding Apple device landscape (iPhone, iPad, Mac Catalyst, Watch) makes user agent sniffing increasingly fragile and unmaintainable.

---
_Source: WWDC19 Session 203 page (abstract, chapter summaries, code samples, and resource links)._
