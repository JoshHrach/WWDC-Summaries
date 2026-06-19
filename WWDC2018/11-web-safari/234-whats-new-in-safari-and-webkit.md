# What's New in Safari and WebKit
**WWDC18 · Session 234** · [Watch](https://developer.apple.com/videos/play/wwdc2018/234/)

_Platforms:_ iOS 12, macOS Mojave 10.14, watchOS 5

## Overview
A broad survey of new Safari and WebKit features in 2018, organized into three themes: security, performance, and rich experiences. The session targets web developers, native app developers embedding web content, and Safari extension developers. It is accompanied by a live demo on a birdhouse lifestyle blog, illustrating subresource integrity, the Beacon API, async image decoding, MP4-in-image elements, Drag and Drop Data Transfer, Payment Request API with Apple Pay, and AR Quick Look via USDZ links.

Two major deprecation announcements anchor the session: `UIWebView` is now officially deprecated in favor of `WKWebView`, and legacy Safari EXTZ extensions are blocked in Safari 12 if distributed outside the Safari Extensions Gallery (with full transition to Safari App Extensions planned).

## Key Topics

### Security

**WKWebView replaces UIWebView [DEPRECATED]**
- `UIWebView` officially deprecated; `WKWebView` is the replacement for all platforms
- `WKWebView` advantages: runs in a separate process from the app (web process sandbox), crashes are confined to the web view, out-of-process rendering prevents app stalls, works on both iOS and macOS (shared code), supports modern web standards

**Safari Extension Model Evolution**
- Legacy EXTZ extensions distributed outside the Safari Extensions Gallery are blocked starting in Safari 12
- Extensions using the deprecated `canLoad` API are off by default
- Gallery submissions accepted through end of 2018; eventual full transition to Safari App Extensions
- For ad blockers: use Content Blockers (faster, private, built in Xcode)
- For other extensions: use Safari App Extensions (distributed via App Store)

**Subresource Integrity**
- Adds `integrity` attribute to `<script>` or `<link>` tags; value is `sha256-<base64hash>`
- When the resource is fetched, WebKit computes its hash and compares to the declared value; mismatch blocks the resource from loading
- Protects against CDN compromise: even if a third-party CDN is hacked and the file is modified, it won't execute
- Fallback: provide a `<link onerror>` or a secondary source to load from your own server if the CDN check fails

**Intelligent Tracking Prevention (ITP) Updates**
- Removes the 24-hour general cookie access window for domains with cross-site tracking ability
- All third-party cookies default to isolated storage
- **Storage Access API [NEW]**: domains with cross-site tracking that need cookie access in a third-party context must call `document.requestStorageAccess()` — triggers a user prompt for explicit consent

**Automatic Strong Passwords**
- Heuristics detect sign-up/login pages automatically; no changes required for most sites
- To guarantee behavior: add `autocomplete="new-password"` to password fields
- `passwordrules` attribute — specifies site-specific password requirements (length, character classes); use the Apple Password Rules Validation Tool to test compatibility
- Default generated passwords: 20 characters, mixed case, digits, hyphens

**Security Code AutoFill**
- Heuristics detect one-time code fields automatically
- To guarantee: add `autocomplete="one-time-code"` to input fields
- One-time code appears in QuickType bar from Messages

### Performance

**Font Collections (WOFF2 and TrueType Collections) [NEW]**
- Bundle related fonts with shared character tables into a single collection file
- Eliminates duplicated character map tables; up to 84% file size reduction (e.g., PingFang)

**`font-display` CSS Descriptor [NEW]**
- Controls what happens during the 0–3 second window while a custom font loads
- Values: `swap` (use fallback immediately, swap when font ready), `block` (blank for up to 3s), `fallback`, `optional`
- Prevents invisible text (FOIT) or flash of unstyled text (FOUT) as desired

**MP4 in `<img>` Element [NEW]**
- `<img src="animation.mp4">` — uses hardware video decoding; ~7x smaller file size vs. animated GIF, better quality, lower battery impact
- Also works in CSS `background-image: url('animation.mp4')`
- Use `<picture>` element with `<source>` for fallback image in non-supporting browsers

**Passive Event Listeners (default on document/window/body)**
- Touch event listeners on `document`, `window`, and `body` are passive by default — scrolling is not blocked waiting for event listener completion
- For other elements: `addEventListener('touchstart', handler, { passive: true })`
- Prevents scroll jank from slow event handlers

**Async Image Decoding [NEW default on first page load]**
- Images on the first page load are decoded asynchronously on a background thread by default
- For dynamic content: use `HTMLImageElement.decode()` which returns a Promise that resolves when the image is ready to display without causing a frame delay
- Also available via markup: `<img decoding="async">`

**Beacon API [NEW]**
- `navigator.sendBeacon(url, data)` — sends data asynchronously on page unload; browser guarantees delivery even as page navigates away
- Replaces synchronous XHR in unload handlers (which caused navigation delay)
- Use case: analytics, click tracking, session end reporting

### Rich Experiences

**Drag and Drop — Data Transfer API (iOS) [NEW]**
- `event.dataTransfer.setData(mimeType, data)` in `dragstart` handler — specify what data is transferred
- `event.dataTransfer.getData(mimeType)` in `drop` handler — retrieve the data
- Supports dragging entire directories for upload without compression
- Supports reading/writing MIME types for rich HTML, plain text, and URLs to the system pasteboard

**Payment Request API with Apple Pay [NEW]**
- Standard W3C Payment Request API now supports Apple Pay as a payment method
- `new PaymentRequest(methodData, details, options)` — `methodData` specifies Apple Pay identifier + merchant capabilities
- `paymentRequest.show()` — returns Promise resolving with `PaymentResponse` on Face ID/Touch ID authorization
- `paymentResponse.complete('success' | 'fail')` — finalize the transaction
- Requires server-side Apple Pay payment session; use Apple Pay JS for features not yet in Payment Request (granular errors, co-branded cards, phonetic names)
- Check availability: `ApplePaySession.canMakePayments()`

**Service Workers [NEW]**
- Registered per origin; intercepts fetch requests from all pages on that origin
- Enables offline caching and graceful degradation for poor connectivity
- Works in `SFSafariViewController` as well as Safari tabs
- Multiple tabs share one Service Worker instance

**Fullscreen API for iPad [NEW]**
- Any arbitrary element can enter fullscreen: `element.requestFullscreen()`
- Video auto-detection shows Cancel button that hides after a short delay during playback
- CSS environment variable `env(fullscreen-inset-top)` — avoid content being obscured by Cancel button
- `env(fullscreen-auto-hide-delay)` — synchronize content hide timing with button hide

**AR Quick Look via USDZ [NEW]**
- `<a href="model.usdz" rel="ar"><img src="thumbnail.jpg"></a>` — single link with `rel="ar"` attribute; AR Quick Look icon appears in thumbnail corner
- Tapping opens AR Quick Look in-place; places 3D model in real-world environment
- No JavaScript required; purely declarative HTML

**Websites on watchOS [NEW]**
- Safari renders websites on Apple Watch; responsive designs work automatically
- Optimize further using watchOS-specific CSS media queries

## APIs & Frameworks

**WKWebView (WebKit)**
- `WKWebView` — replaces deprecated `UIWebView` and macOS `WebView`

**Web APIs (new in Safari 12 / iOS 12)**
- `document.requestStorageAccess()` — Storage Access API for ITP-restricted third-party cookie access
- `HTMLImageElement.decode()` → `Promise<void>` — async decode before DOM insertion
- `<img decoding="async|sync|auto">` — async decoding attribute
- `navigator.sendBeacon(url, data)` — beacon API for reliable background data delivery
- `PaymentRequest(methodData, details, options)` — W3C Payment Request API constructor
- `PaymentRequest.prototype.show()` → `Promise<PaymentResponse>` — show payment sheet
- `PaymentResponse.prototype.complete(result)` — finalize transaction
- `ApplePaySession.canMakePayments()` — check Apple Pay capability
- `element.requestFullscreen()` — fullscreen API for iPad
- `ServiceWorkerGlobalScope.fetch` event — intercept network requests in a Service Worker
- `DataTransfer.setData(type, data)` / `getData(type)` — drag and drop data transfer (iOS)

**HTML Attributes (new)**
- `<script integrity="sha256-...">` / `<link integrity="...">` — subresource integrity
- `<input autocomplete="new-password">` / `autocomplete="one-time-code"` — strong passwords / security code autofill
- `<input passwordrules="...">` — password format requirements
- `<img src="video.mp4">` — MP4 in image element
- `<img decoding="async">` — async decode attribute
- `<a href="model.usdz" rel="ar">` — AR Quick Look link

**CSS**
- `font-display: swap | block | fallback | optional | auto` — custom font loading control
- `env(fullscreen-inset-top)` / `env(fullscreen-auto-hide-delay)` — fullscreen CSS environment variables
- `background-image: url('animation.mp4')` — MP4 as CSS background

## Code Highlights

Subresource integrity for a stylesheet:
```html
<link rel="stylesheet" href="https://cdn.example.com/styles.css"
      integrity="sha256-abc123XYZ...base64hash..."
      crossorigin="anonymous">
```

Async image decoding before carousel transition:
```javascript
const img = new Image();
img.src = nextSlide.imageSrc;
img.decode().then(() => {
    performTransition(nextSlide);
});
```

Beacon API replacing synchronous XHR:
```javascript
document.querySelectorAll('a').forEach(link => {
    link.addEventListener('click', e => {
        if ('sendBeacon' in navigator) {
            navigator.sendBeacon('/track', JSON.stringify({ href: link.href }));
        } else {
            // synchronous XHR fallback
        }
    });
});
```

AR Quick Look link (no JavaScript required):
```html
<a href="model.usdz" rel="ar">
    <img src="model-thumbnail.jpg" alt="View in AR">
</a>
```

## Takeaways
- Migrate from `UIWebView` to `WKWebView` immediately — `UIWebView` is deprecated in iOS 12 and the security and stability benefits of the out-of-process model are significant.
- Replace animated GIFs with MP4 in `<img>` elements — identical visual result with ~7x smaller files, hardware decoding, and lower battery use.
- Use `HTMLImageElement.decode()` for dynamically loaded images in carousels or tile maps — eliminates the blank-frame flash that occurs when images are inserted before they finish decoding.
- Add `rel="ar"` to USDZ links for instant AR Quick Look integration — no JavaScript, no SDK, no extra code beyond the anchor tag.

---
_Source: WWDC18 Session 234 page (abstract, full transcript, and resource links)._
