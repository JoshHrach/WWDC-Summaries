# Explore WKWebView Additions
**WWDC21 · Session 10032** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10032/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12

## Overview
This session covers new WKWebView APIs in iOS 15 and macOS Monterey that reduce the need to inject JavaScript for common web content interactions. The additions fall into two categories: non-JavaScript manipulation APIs (theme color, text interaction, media playback controls) and browser-level features previously only available in Safari (automatic HTTPS upgrades, getUserMedia/WebRTC improvements, and a new WKDownload API for in-app file downloads).

The session also introduces an enhancement to `SFSafariViewController` allowing apps to add a custom toolbar button mapped to a share extension.

## Key Topics

**SFSafariViewController Custom Button**
A new API in iOS 15 lets apps add a custom button to `SFSafariViewController`'s toolbar, mapped to one of the app's share extensions (including JavaScript-based extensions), improving discoverability for app-specific actions on displayed web content.

**Theme Color (No JavaScript)**
`WKWebView.themeColor` (read-only) exposes the `<meta name="theme-color">` value set by the web page. A new `underPageBackgroundColor` property provides a calculated fallback color for below-scroll-edge fills and is also writable to override the background color.

**Text Interaction Control**
`WKPreferences.textInteractionEnabled` (default `true`) — setting to `false` disables all text selection UI in the web view, preventing accidental selection when users interact with media controls.

**Media Playback Control (No JavaScript)**
New methods on `WKWebView` allow pausing, suspending, and querying media without JavaScript. `setAllMediaPlaybackSuspended(_:completionHandler:)` is sticky across page reloads (it's a property of the web view, not the content).

**Automatic HTTPS Upgrade**
Starting in iOS 15 and macOS Monterey, `WKWebView` automatically upgrades HTTP requests to HTTPS for sites known to support it. No opt-in required. Can be disabled via `WKWebViewConfiguration.upgradeKnownHostsToHTTPS = false` for local debugging (not recommended in production).

**getUserMedia / WebRTC Improvements**
When loading content from a custom `WKURLSchemeHandler`, the camera/microphone permission prompt shows the app name as origin instead of the URL. A new `WKUIDelegate` method lets apps intercept and control camera/microphone permission prompts (e.g., to skip the prompt for trusted content). Camera and microphone capture state can be read and set without JavaScript.

**WKDownload API**
New download management API supporting three initiation paths: web-content-initiated (JavaScript `download` attribute), server-initiated (`Content-Disposition: attachment` response header), and app-initiated (via `WKWebView.startDownload(using:completionHandler:)`). All paths produce a `WKDownload` object; apps must set its `delegate` to receive file data or the download is cancelled.

## APIs & Frameworks

- **WebKit** — `WKWebView`, `WKPreferences`, `WKWebViewConfiguration`
- `SFSafariViewController` extension:
  - `activityButton: UIActivity?` **[NEW]** — maps a custom share extension to a toolbar button

- `WKWebView`:
  - `var themeColor: UIColor?` **[NEW]** — reads `<meta name="theme-color">`
  - `var underPageBackgroundColor: UIColor` **[NEW]** — calculated/settable background fill color
  - `func pauseAllMediaPlayback(completionHandler:)` **[NEW]** — pauses all media elements
  - `func setAllMediaPlaybackSuspended(_ suspended: Bool, completionHandler:)` **[NEW]** — sticky suspend across reloads
  - `func closeAllMediaPresentations(completionHandler:)` **[NEW]** — closes fullscreen/PiP
  - `func requestMediaPlaybackState(completionHandler:)` **[NEW]** — queries current playback state
  - `func setCameraCaptureState(_ state: WKMediaCaptureState, completionHandler:)` **[NEW]**
  - `func setMicrophoneCaptureState(_ state: WKMediaCaptureState, completionHandler:)` **[NEW]**
  - `var cameraCaptureState: WKMediaCaptureState` **[NEW]**
  - `var microphoneCaptureState: WKMediaCaptureState` **[NEW]**
  - `func startDownload(using request: URLRequest, completionHandler:)` **[NEW]**
  - `func resumeDownload(fromResumeData:completionHandler:)` **[NEW]**

- `WKPreferences`:
  - `var textInteractionEnabled: Bool` **[NEW]** — disables text selection UI

- `WKWebViewConfiguration`:
  - `var upgradeKnownHostsToHTTPS: Bool` **[NEW]** — controls automatic HTTPS upgrade (default `true`)

- `WKMediaCaptureState` **[NEW]** — enum: `.none`, `.active`, `.muted`

- `WKUIDelegate` — extended:
  - `func webView(_ webView: WKWebView, requestMediaCapturePermissionFor origin: WKSecurityOrigin, initiatedByFrame frame: WKFrameInfo, type: WKMediaCaptureType, decisionHandler: @escaping (WKPermissionDecision) -> Void)` **[NEW]**

- `WKPermissionDecision` **[NEW]** — enum: `.prompt`, `.grant`, `.deny`

- `WKDownload` **[NEW]**
  - `var delegate: WKDownloadDelegate?`
  - `func cancel(completionHandler:)`
- `WKDownloadDelegate` **[NEW]**
  - `func download(_:decideDestinationUsing:suggestedFilename:completionHandler:)` — provides destination URL
  - `func download(_:didFailWithError:resumeData:)` — provides resume data

- `WKNavigationDelegate` — extended:
  - `WKNavigationActionPolicy.download` **[NEW]** — triggers download from navigation action
  - `WKNavigationResponsePolicy.download` **[NEW]** — triggers download from response

## Code Highlights

Theme color applied to a native header view:
```swift
headerView.backgroundColor = webView.themeColor
```

Disable text interaction in preferences:
```swift
let preferences = WKPreferences()
preferences.textInteractionEnabled = false
let config = WKWebViewConfiguration()
config.preferences = preferences
```

Sticky media suspension:
```swift
webView.setAllMediaPlaybackSuspended(true) { }
// Remains suspended even after page reload
```

Skip camera/microphone prompt for trusted origin:
```swift
func webView(_ webView: WKWebView,
             requestMediaCapturePermissionFor origin: WKSecurityOrigin,
             initiatedByFrame frame: WKFrameInfo,
             type: WKMediaCaptureType,
             decisionHandler: @escaping (WKPermissionDecision) -> Void) {
    if origin.host == "trusted.example.com" {
        decisionHandler(.grant)
    } else {
        decisionHandler(.prompt)
    }
}
```

Mute camera via capture state:
```swift
webView.setCameraCaptureState(.muted) { }
```

## Takeaways

- Five new non-JavaScript APIs eliminate the need for DOM injection for theme color, text selection, media playback, and capture state — and these work safely with App-Bound Domains and Apple Pay.
- `setAllMediaPlaybackSuspended` is sticky across page reloads (unlike `pauseAllMediaPlayback`), making it the right choice for persistent user preferences.
- `WKDownload` brings full download management (destination, progress, resume) into WKWebView apps for the first time.
- Automatic HTTPS upgrade is on by default in iOS 15/macOS Monterey with no code changes required.

---
_Source: WWDC21 Session 10032 page (abstract, chapter summaries, code samples, and resource links)._
