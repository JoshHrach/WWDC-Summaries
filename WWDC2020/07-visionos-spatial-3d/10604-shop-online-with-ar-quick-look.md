# Shop online with AR Quick Look
**WWDC20 · Session 10604** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10604/)

_Platforms:_ iOS 13.3+, iPadOS 13.3+, iOS 14 (some features)

## Overview
AR Quick Look enables retailers and content publishers to showcase 3D products in augmented reality directly through Safari without requiring a separate app download. This session introduces the new **commerce banner** feature — three styles of interactive purchase/action banners that appear at the bottom of the AR Quick Look viewer — and shows how to integrate them through web URL fragment parameters and a JavaScript `EventListener`.

The banner integration works for both native apps (using `QLPreviewController`) and web pages (HTML `<a rel="ar">` links), making it the simplest path for e-commerce AR integration. Real-world results cited include Bang & Olufsen seeing 4x more store visits from users who engaged with the AR experience.

## Key Topics

### AR Quick Look Background
- System-wide AR viewer for USDZ and Reality File content, available without downloading an app
- Supports horizontal and vertical plane detection, instant AR placement, object occlusion, and face accessory try-on
- Integrated with Reality Composer and Reality Converter for authoring workflows
- Configurable via URL **fragment identifier** parameters (after `#`) — no server-side changes needed

### Three Banner Styles (NEW in iOS 13.3 / fully featured in iOS 14)

#### 1. Apple Pay Banner
- Templated layout: product title + subtitle on left, Apple Pay button on right
- 7 Apple Pay button types: `plain`, `buy`, `check-out`, `book`, `donate`, `subscribe`, and more
- No payment data collected in AR Quick Look — payment sheet appears after AR dismisses, back on the retail page
- Required parameters: `applePayButtonType`, `checkoutTitle`, `checkoutSubtitle`, `price`

#### 2. Custom Action Banner
- Same layout as Apple Pay but with a custom call-to-action text button (e.g., "Add to Cart", "Preorder")
- Required parameters: `callToAction`, `checkoutTitle`, `checkoutSubtitle`
- `price` parameter is **optional in iOS 14** (still required on iOS 13)
- Website performs custom logic (add to cart, open Maps, start Business Chat) after dismissal

#### 3. Custom HTML Banner (most flexible)
- Entire banner replaced with developer-supplied HTML file served over **HTTPS**
- Three predefined heights: `small` (81pt), `medium` (121pt), `large` (161pt) via `customHeight` parameter
- Allows full brand-matching: custom fonts, layouts, graphics
- Can redirect to Business Chat, phone support, or other custom flows

### URL Fragment Parameters
All parameters are appended to the USDZ/Reality file URL as percent-encoded fragment parameters:
- `canonicalWebPageURL` — URL shared when user shares from AR Quick Look (overrides default)
- `allowsContentScaling` — `0` to disable scaling (view at true real-world size)
- `applePayButtonType` — triggers Apple Pay banner; values: `plain`, `buy`, `checkout`, `book`, `donate`, `subscribe`
- `checkoutTitle` / `checkoutSubtitle` — product title and subtitle (percent-encoded)
- `price` — price string with currency symbol (percent-encoded; localize for each locale)
- `callToAction` — custom action button text (triggers custom action banner)
- `custom` — absolute HTTPS URL to HTML file (triggers custom HTML banner)
- `customHeight` — `small` (default, 81pt), `medium` (121pt), `large` (161pt)

### JavaScript Event Listener Integration
- Add `id="ar-link"` to the `<a rel="ar">` element
- Attach a `"message"` event listener to the element
- Check `event.data == "_apple_ar_quicklook_button_tapped"` to confirm the banner was tapped
- On tap, present Apple Pay sheet or run custom logic

### HTML Web Integration
- `<a rel="ar">` — tells Safari this is an AR link; adds AR badge; opens AR Quick Look directly
- Nested `<img>` child provides the thumbnail image displayed in the page

## APIs & Frameworks

- **ARKit** / **AR Quick Look** — system AR viewer
- **QLPreviewController** — native app integration for presenting AR Quick Look
- **USDZ** / **.reality** file formats — 3D model formats
- Web URL fragment parameters **[NEW]**:
  - `applePayButtonType` — Apple Pay button style
  - `callToAction` — custom action text
  - `checkoutTitle`, `checkoutSubtitle`, `price` — product info
  - `custom`, `customHeight` — custom HTML banner
  - `canonicalWebPageURL`, `allowsContentScaling` — existing parameters
- **Apple Pay** — `PKPaymentRequest`, payment sheet triggered via JavaScript after AR dismissal
- **Business Chat** — `businesschat://` / Business Chat URL — launchable from custom banner
- JavaScript `EventListener` — `"message"` event on the AR link element; `event.data == "_apple_ar_quicklook_button_tapped"`
- `<a rel="ar" href="model.usdz#...">` — HTML AR Quick Look anchor tag

## Code Highlights

Apple Pay banner HTML link:
```html
<a rel="ar" id="ar-link"
   href="alarm-clock.usdz#applePayButtonType=plain&checkoutTitle=Retro%20Alarm%20Clock&checkoutSubtitle=Charming%20old-school%20look%20with%20built-in%20FM%20tuner&price=$92.50">
    <img src="alarm-clock-thumbnail.jpg">
</a>
```

Custom action banner:
```html
<a rel="ar" id="ar-link"
   href="kids-slide.usdz#callToAction=Preorder&checkoutTitle=Kids%20Slide&checkoutSubtitle=Enjoy%20the%20playground,%20right%20from%20your%20home&price=$145">
    <img src="kids-slide-thumbnail.jpg">
</a>
```

Custom HTML banner with small height:
```html
<a rel="ar" id="ar-link"
   href="solar-panels.usdz#custom=https://example.com/banner.html&customHeight=small">
    <img src="solar-panels-thumbnail.jpg">
</a>
```

JavaScript event listener for tap events:
```javascript
const linkElement = document.getElementById("ar-link");
linkElement.addEventListener("message", function(event) {
    if (event.data == "_apple_ar_quicklook_button_tapped") {
        // present Apple Pay sheet or add to cart
    }
}, false);
```

## Takeaways

- Adding an AR commerce banner requires only HTML `<a rel="ar">` markup changes and a small JavaScript event listener — no native app code needed for web integrations.
- Always percent-encode fragment parameter values; localize `checkoutTitle`, `checkoutSubtitle`, `price`, and `callToAction` strings for each locale.
- The custom HTML banner (served over HTTPS) provides complete branding flexibility and can trigger any web action — Business Chat, custom forms, or direct redirects.
- No payment or personal data is collected inside AR Quick Look — the payment experience always happens back on the retail web page after AR dismisses, preserving Apple Pay security.

---
_Source: WWDC20 Session 10604 page (abstract, chapter summaries, code samples, and resource links)._
