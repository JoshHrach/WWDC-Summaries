# What's New for Web Developers
**WWDC19 · Session 518** · [Watch](https://developer.apple.com/videos/play/wwdc2019/518/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15, Safari 13

## Overview
Safari 13 and WebKit in iOS 13 / macOS Catalina ship a broad set of web platform improvements for 2019. The session surveys Dark Mode support via CSS media queries, the Web Share API for invoking the native share sheet, new media capabilities including alpha-channel HEVC video and `getDisplayMedia` for screen capture, the new CPU Timeline in Web Inspector, WebDriver on iOS, the removal of WebSQL, full Apple Pay support via the Payment Request API, and Apple Pay in `WKWebView`.

The session takes a breadth-over-depth approach, pointing to companion blog posts and other WWDC videos for each topic. The key throughline is that web content can now participate more deeply in platform features — Dark Mode, Share Sheet, Apple Pay, screen capture — while the developer tools have grown more powerful for diagnosing performance and regressions.

## Key Topics

**Dark Mode**
The `color-scheme` CSS property signals Dark Mode support; the `prefers-color-scheme: dark` media query adapts custom styles, CSS images, and HTML `<picture>` elements. JavaScript `window.matchMedia('(prefers-color-scheme: dark)')` handles dynamic content. Supported in Safari 13 on macOS and iOS.

**Web Share API**
`navigator.share({ title, url, text })` opens the native iOS/macOS share sheet from a user gesture. Returns a Promise. Already shipping in Safari (iOS 12.2+). Twitter's share buttons are a real-world example.

**Media Capabilities API**
`navigator.mediaCapabilities.decodingInfo({ type, video })` lets sites query whether a codec is supported, smooth, and power-efficient before choosing which video to serve.

**Alpha Channel HEVC Video**
HEVC-encoded video can carry an alpha channel, enabling transparent video that blends with the page background. No special syntax or MIME type required; supported on iOS 13 and macOS Catalina. Verify support using the Media Capabilities API with `{ alphaChannel: true }`.

**getDisplayMedia (Screen Capture)**
`navigator.mediaDevices.getDisplayMedia()` prompts for screen capture permission and returns a standard `MediaStream` of the Safari window. Available in Safari 13; usable with WebRTC for remote screen sharing.

**Web Inspector CPU Timeline**
A new timeline added to Web Inspector (see Session 513 for full coverage) provides actionable CPU usage data.

**WebDriver on iOS**
The W3C WebDriver standard for browser automation is now available on all devices running iOS 13. Enables cross-platform regression testing of web content on mobile.

**WebSQL Removed**
WebSQL is fully removed in Safari 13. Migrate to IndexedDB, which Apple has been making more standards-compliant. All modern browsers support IndexedDB.

**Apple Pay via Payment Request API**
The Payment Request API now fully supports Apple Pay on all platforms, superseding Apple Pay JS. Apple strongly encourages switching; the Payment Request API is more standardized and continues to evolve.

**Apple Pay in WKWebView**
Apple Pay now works inside `WKWebView`. Restrictions: if any script has been injected into the current page load, Apple Pay is disabled. If `canMakePayments` has been called, future script injection is blocked. Both restrictions reset on top-level navigation. App developers should minimize or eliminate script injection; web developers must call `canMakePayments` before showing an Apple Pay button.

## APIs & Frameworks

**CSS / HTML**
- `color-scheme: light dark` CSS property **[NEW in Safari 13]** — signals Dark Mode support; adapts default colors and controls
- `@media (prefers-color-scheme: dark)` CSS media query **[NEW in Safari 13]** — applies dark custom styles
- `<picture>` element with `media="(prefers-color-scheme: dark)"` — swaps images for Dark Mode
- `window.matchMedia('(prefers-color-scheme: dark)')` — JavaScript Dark Mode detection

**Web APIs (JavaScript)**
- `navigator.share({ title, url, text })` **[NEW in Safari / iOS 12.2+]** — Web Share API; opens native share sheet
- `navigator.mediaCapabilities.decodingInfo(config)` **[NEW in Safari 13]** — Media Capabilities API; checks codec support, smoothness, power efficiency
  - `config.video.alphaChannel: true` — checks alpha channel HEVC support **[NEW]**
- `navigator.mediaDevices.getDisplayMedia()` **[NEW in Safari 13]** — W3C Screen Capture API; returns `MediaStream` of display
- `window.matchMedia(query)` — existing API; used for Dark Mode JS detection
- `IndexedDB` — recommended structured storage API; enhanced standards compliance in Safari 13
- WebSQL — **REMOVED** in Safari 13

**Payment**
- Payment Request API with Apple Pay payment method **[fully supported NEW in Safari 13]** — standardized Apple Pay on web
- `PaymentRequest.canMakePayment()` — must be called before showing Apple Pay button in `WKWebView`
- Apple Pay JS — still works but migration to Payment Request API recommended

**WKWebView / WebKit (Swift/ObjC)**
- Apple Pay support in `WKWebView` **[NEW]**
- `WKUserScript` / `evaluateJavaScript(completionHandler:)` — script injection; blocks Apple Pay if called on current page
- `WKWebView` Apple Pay restrictions: script injection before `canMakePayments` disables Apple Pay; `canMakePayments` after script injection is blocked

**Testing**
- WebDriver on iOS **[NEW]** — W3C standard; automates Safari on iOS 13 devices
- Web Inspector CPU Timeline — new tool for diagnosing CPU and power regressions (see Session 513)

## Code Highlights

Dark Mode in CSS:

```css
:root {
    color-scheme: light dark;
}
body { background: white; color: black; }

@media (prefers-color-scheme: dark) {
    body { background: #1c1c1e; color: white; }
}
```

Web Share API:

```javascript
document.querySelector('#share-btn').addEventListener('click', async () => {
    try {
        await navigator.share({ title: document.title, url: location.href });
    } catch (err) {
        console.log('Share cancelled or failed:', err);
    }
});
```

Checking alpha channel HEVC support:

```javascript
const result = await navigator.mediaCapabilities.decodingInfo({
    type: 'file',
    video: { contentType: 'video/mp4; codecs="hvc1"', alphaChannel: true,
             width: 1920, height: 1080, bitrate: 4000000, framerate: 30 }
});
if (result.supported) { /* use alpha video */ }
```

## Takeaways
- Dark Mode support requires both the `color-scheme` property and `prefers-color-scheme` media queries; the property alone only updates default browser chrome colors.
- The Web Share API gives web content direct access to the iOS share sheet with no special entitlements — just a user gesture.
- Apple Pay in `WKWebView` is gated on a no-script-injection policy; app developers must audit `WKUserScript` usage or Apple Pay will silently be unavailable.
- WebSQL is gone in Safari 13 — migrate to IndexedDB immediately; there is no grace period.

---
_Source: WWDC19 Session 518 page (abstract, transcript, and resource links)._
