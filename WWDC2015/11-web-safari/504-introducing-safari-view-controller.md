# Introducing Safari View Controller
**WWDC15 · Session 504** · [Watch](https://developer.apple.com/videos/play/wwdc2015/504/)

_Platforms:_ iOS 9

## Overview
This session introduces `SFSafariViewController`, a new iOS 9 class in the SafariServices framework that lets apps display web content with the full feature set of Safari — including shared cookies, Password AutoFill via iCloud Keychain, Safari Reader, Content Blocking, and proper SSL communication — while keeping the user inside the app. The session argues that most in-app "mini browsers" built on WKWebView are missing years of features that users already expect, and that `SFSafariViewController` can replace them with just a handful of lines of code.

The session also covers the updated `WKWebView` in iOS 9 (secure file URL loading, custom user agents, `WKWebsiteDataStore`), the three categories of web content use cases in apps, and how `SFSafariViewController` is ideal for web-based OAuth/authentication flows — eliminating the app's need to handle user credentials.

A live demo replaces an 80-line custom WKWebView browser with ~7 lines using `SFSafariViewController`, restoring Reader, Password AutoFill, and scroll dynamics.

## Key Topics

### Three Web Content Use Cases
1. **Custom web content inside the app** — HTML/JS/CSS you own or control; use `WKWebView`.
2. **Showing a website when a user taps a link** — use `SFSafariViewController` (or `UIApplication.openURL` to hand off to Safari).
3. **Web-based authentication (OAuth)** — use `SFSafariViewController` for the login web page; dismiss on the custom URL scheme redirect.

### WKWebView Updates in iOS 9
- `loadFileURL(_:allowingReadAccessTo:)` — secure file URL loading **[NEW]**
- `loadData(_:mimeType:characterEncodingName:baseURL:)` — load raw HTML strings **[NEW]**
- `customUserAgent` property **[NEW]**
- `WKWebsiteDataStore` — read/write data store on `WKWebViewConfiguration` **[NEW]**
  - Remove data by type; remove data added within a time window
  - Use a non-persistent store to implement private browsing

### SFSafariViewController Features
- **Shared cookies and website data** with Safari — users may already be logged in.
- **Password AutoFill** via iCloud Keychain — fills credentials synced across devices.
- **Contact Card AutoFill** — fills shipping/billing address information.
- **Credit Card AutoFill**.
- **Safari Reader** — distraction-free article view; customizable themes and fonts in iOS 9 **[NEW]**.
- **Content Blocking** — all content blockers the user has enabled in Settings apply automatically.
- **Share sheet** with system activities plus **custom activities** provided by the host app.
- **Custom tint color** — `tintColor` on the view controller for brand identity.
- **SSL validity communication** and **phishing warnings** — identical to Safari behavior.
- **Read-only URL bar** — eliminates distractions; no tab management.
- Progress bar, informative error pages.
- Runs in a **separate process** from the host app — the host app never has access to user credentials or website data.

### Safari View Controller for OAuth / Web-based Authentication
- Present `SFSafariViewController` where you previously used a custom web view for the login page.
- User may already be logged in (shared cookies); if not, Password AutoFill fills credentials.
- Accept the redirect URL via `UIApplicationDelegate.application(_:open:options:)` (custom URL scheme callback).
- Dismiss the `SFSafariViewController` in the callback.
- Result: higher authentication conversion rates; no credential handling by the app.

### Delegating to Safari (Alternative)
- `UIApplication.shared.open(_:options:completionHandler:)` / `openURL` hands off to Safari entirely.
- iOS 9's system back-swipe affordance (return to previous app) makes this lighter weight than before.

## APIs & Frameworks

- `SafariServices` framework **[NEW]**
- `SFSafariViewController` **[NEW]** — `UIViewController` subclass
  - `init(url: URL)` **[NEW]**
  - `init(url: URL, entersReaderIfAvailable: Bool)` **[NEW]**
  - `delegate: SFSafariViewControllerDelegate?`
  - `preferredBarTintColor` / `preferredControlTintColor` (tint color customization) **[NEW]**
- `SFSafariViewControllerDelegate` **[NEW]**
  - `safariViewController(_:activityItemsFor:title:)` — provide custom share sheet activities **[NEW]**
  - `safariViewControllerDidFinish(_:)` — called on Done tap **[NEW]**
- `WKWebView` (WebKit)
  - `loadFileURL(_:allowingReadAccessTo:)` **[NEW in iOS 9]**
  - `loadData(_:mimeType:characterEncodingName:baseURL:)` **[NEW in iOS 9]**
  - `customUserAgent: String?` **[NEW in iOS 9]**
  - `WKWebViewConfiguration`
  - `WKWebsiteDataStore` **[NEW in iOS 9]** — `default()`, `nonPersistent()`
    - `removeData(ofTypes:modifiedSince:completionHandler:)` **[NEW]**
    - `removeData(ofTypes:for:completionHandler:)` **[NEW]**
    - `WKWebsiteDataType` constants: `.cookies`, `.diskCache`, `.memoryCache`, `.localStorage`, etc.
- `UIWebView` — legacy; `WKWebView` is preferred replacement
- `UIApplication.shared.openURL(_:)` / `open(_:options:completionHandler:)` — delegate browsing to Safari
- `UIApplicationDelegate.application(_:open:options:)` — receive custom URL scheme callbacks for OAuth
- Content Blocking extensions (`WKContentRuleList`) — applied automatically in `SFSafariViewController`
- iCloud Keychain — backs Password AutoFill in `SFSafariViewController`

## Code Highlights

Minimal Safari View Controller adoption:
```swift
import SafariServices

class MyViewController: UIViewController, SFSafariViewControllerDelegate {
    func showWebsite(url: URL) {
        let svc = SFSafariViewController(url: url)
        svc.delegate = self
        present(svc, animated: true)
    }

    func safariViewControllerDidFinish(_ controller: SFSafariViewController) {
        dismiss(animated: true)
    }
}
```

OAuth flow — dismiss on redirect:
```swift
func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey: Any]) -> Bool {
    // Handle custom scheme redirect from OAuth web page
    if url.scheme == "myapp" {
        // Process token in url.query
        safariVC?.dismiss(animated: true)
        return true
    }
    return false
}
```

WKWebsiteDataStore — clear all data from the last hour:
```swift
let store = WKWebsiteDataStore.default()
let since = Date(timeIntervalSinceNow: -3600)
store.removeData(ofTypes: WKWebsiteDataStore.allWebsiteDataTypes(),
                 modifiedSince: since) { }
```

## Takeaways
- `SFSafariViewController` replaces custom in-app browsers with ~7 lines of code while adding Password AutoFill, Reader, shared cookies, Content Blocking, and proper security indicators that a DIY WKWebView browser can never match.
- Running in a separate process, `SFSafariViewController` gives apps zero access to user credentials — a security benefit for both users and developers.
- For OAuth and web-based authentication, `SFSafariViewController` significantly increases login conversion rates by offering AutoFill and possible already-logged-in state.
- `WKWebView` remains the right tool for web content you own and control (custom UI, JavaScript interop); `SFSafariViewController` is the right tool for third-party website browsing.

---
_Source: WWDC15 Session 504 page (abstract, chapter summaries, code samples, and resource links)._
