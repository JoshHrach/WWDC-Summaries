# Discover WKWebView Enhancements
**WWDC20 · Session 10188** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10188/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11

## Overview
iOS 14 and macOS Big Sur bring a significant expansion of the `WKWebView` API, addressing long-standing developer requests for capabilities that were available in the now-deprecated `WebView`/`UIWebView` but missing from `WKWebView`. The session uses a narrative of building a combined news feed app that embeds two separate websites to demonstrate five categories of new APIs: JavaScript isolation and world management, improved JavaScript interaction patterns, rendering fine-tuning, content export, and privacy protections.

Key headline features include `WKContentWorld` for isolating app JavaScript from page JavaScript, `callAsyncJavaScript` for natural async JavaScript calls with native argument passing, `WKWebView.find(_:)` for native find-in-page, `createPDF(configuration:)` and `createWebArchiveData()` for content export, `pageZoom` for CSS-free zoom control, `customMediaType` for CSS media query targeting, and Intelligent Tracking Prevention (ITP) enabled by default. A new privacy feature called App-bound domains allows apps to declare which domains receive deep interaction capabilities.

## Key Topics

### Isolating App from Web Content
- `WKPreferences.javaScriptEnabled` is **deprecated** in iOS 14.
- New `WKWebPagePreferences.allowsContentJavaScript` — disables only JavaScript from the web page itself (inline scripts, remote JS files, JS URLs), while app-injected JavaScript continues to work. Configured per-navigation in the policy delegate.
- `WKContentWorld` — isolated JavaScript sandbox; prevents app code from accidentally conflicting with page code and prevents malicious pages from stealing app state.
  - `.page` world — the web page's own JavaScript context.
  - `.defaultClient` world and named worlds — separate sandboxes for app JavaScript.
  - Apply to `evaluateJavaScript(_:in:in:completionHandler:)` and `WKScriptMessageHandler` injection.

### Improved JavaScript Interaction
- `callAsyncJavaScript(_:arguments:in:in:completionHandler:)` **[NEW]** — execute a JavaScript string as a function body; named arguments are auto-serialized (Swift `String` → JS string, `Int` → JS number, `Dictionary` → JS object); if the JS returns a `Promise`, the completion handler waits for the promise to resolve.
- `WKScriptMessageHandlerWithReply` **[NEW]** — new protocol variant of `WKScriptMessageHandler`; receives a completion handler per message; calling the completion handler resolves the `Promise` returned by `window.webkit.messageHandlers.<name>.postMessage()` in JavaScript.

### Rendering Fine-Tuning
- `WKWebView.pageZoom: CGFloat` **[NEW]** — same as command-+/- zoom in Safari; sets CSS zoom on the entire page without JavaScript; no race condition with page load; does not modify the DOM.
- `WKWebView.mediaType: String?` **[NEW]** — sets a custom CSS media type (e.g., `"no-header-and-footer-device"`) that CSS media queries can target; applies globally, not per-navigation; replaces the old `WebView.mediaStyle` on Mac.

### Content Export
- `WKWebView.find(_:configuration:completionHandler:)` **[NEW]** — native find-in-page; configurable direction, case sensitivity, wrapping; scrolls to and selects the found result.
- `WKWebView.createPDF(configuration:completionHandler:)` **[NEW]** — exports visible and off-screen content as a searchable PDF; configurable via `WKPDFConfiguration`.
- `WKWebView.createWebArchiveData(completionHandler:)` **[NEW]** — snapshots the current DOM and all sub-resources as a WebArchive; useful for debugging, testing, and sharing.
- `WKWebView.load(_:mimeType:characterEncodingName:baseURL:)` — loads `Data` with a MIME type; still the correct way to load a `WebArchive` (`application/x-webarchive`).
- WKWebView printing on macOS **[NEW in Big Sur]** — mirrors the iOS printing API.

### Privacy
- **Intelligent Tracking Prevention (ITP)** enabled by default in all `WKWebView` apps in iOS 14 / macOS Big Sur **[NEW]**. Users can disable it for compatibility if the app needs to provide that option.
- **WKAppBoundDomains** **[NEW]** — Info.plist key (`WKAppBoundDomains`) listing an array of domains that are the core of the app's implementation. Deep interaction (JavaScript injection, `WKScriptMessageHandler`, etc.) is restricted to these domains only. Prevents accidental or malicious data exposure on third-party domains. Set to an empty array to restrict all domains.

## APIs & Frameworks

### WebKit
- `WKWebView.pageZoom: CGFloat` **[NEW]**
- `WKWebView.mediaType: String?` **[NEW]**
- `WKWebView.find(_:configuration:completionHandler:)` **[NEW]**
- `WKFindConfiguration` — `.backward`, `.caseSensitive`, `.wraps`
- `WKFindResult` — `.matchFound: Bool`
- `WKWebView.createPDF(configuration:completionHandler:)` **[NEW]**
- `WKPDFConfiguration` — `.rect: CGRect?`
- `WKWebView.createWebArchiveData(completionHandler:)` **[NEW]**
- `WKWebView.callAsyncJavaScript(_:arguments:in:in:completionHandler:)` **[NEW]**
- `WKWebView.evaluateJavaScript(_:in:in:completionHandler:)` — extended with `contentWorld` parameter **[NEW]**
- `WKContentWorld` **[NEW]** — `.page`, `.defaultClient`, `world(name:)`
- `WKUserContentController.add(_:contentWorld:name:)` — inject `WKScriptMessageHandler` into a specific world **[NEW]**
- `WKScriptMessageHandlerWithReply` **[NEW]** — protocol; `userContentController(_:didReceive:replyHandler:)`
- `WKWebPagePreferences.allowsContentJavaScript: Bool` **[NEW]**
- `WKPreferences.javaScriptEnabled` — **deprecated in iOS 14**
- `WKNavigationDelegate.webView(_:decidePolicyFor:preferences:decisionHandler:)` — configure `WKWebPagePreferences` per navigation
- `WKAppBoundDomains` — Info.plist key (array of domain strings) **[NEW]**

## Code Highlights

Disabling page JavaScript per-navigation while keeping app JavaScript active:
```swift
func webView(_ webView: WKWebView, decidePolicyFor action: WKNavigationAction,
             preferences: WKWebpagePreferences,
             decisionHandler: @escaping (WKNavigationActionPolicy, WKWebpagePreferences) -> Void) {
    if action.request.url == mainFeedURL {
        preferences.allowsContentJavaScript = false
    }
    decisionHandler(.allow, preferences)
}
```

Calling async JavaScript with typed arguments:
```swift
webView.callAsyncJavaScript("return await fetchPosts(from: from, count: count)",
    arguments: ["from": 0, "count": 20],
    in: .defaultClient,
    in: WKContentWorld.defaultClient) { result in
    // called when the Promise resolves
}
```

Creating a PDF of all page content:
```swift
webView.createPDF(configuration: WKPDFConfiguration()) { result in
    if case .success(let data) = result {
        // share or save data
    }
}
```

## Takeaways
- Replace `WKPreferences.javaScriptEnabled` with `WKWebPagePreferences.allowsContentJavaScript` to disable page JS per-navigation while keeping app JS working.
- Use `WKContentWorld` to isolate all app-injected JavaScript from page JavaScript — prevents conflicts and improves security.
- `callAsyncJavaScript` replaces manual string construction; argument types are auto-serialized and Promises are transparently awaited.
- Declare `WKAppBoundDomains` in Info.plist to enable App-bound domains — limits deep WebKit interaction to your own domains only.
- ITP is now on by default in all WKWebView apps — a significant privacy improvement requiring no code changes.

---
_Source: WWDC20 Session 10188 page (abstract, transcript, and resource links)._
