# What's New for Web Developers
**WWDC20 · Session 10663** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10663/)

_Platforms:_ Safari 14 (iOS 14, iPadOS 14, macOS Big Sur 11); some features in Safari 13.1

## Overview
This session provides a comprehensive tour of Safari and WebKit updates arriving in Safari 13.1 and Safari 14. A theme throughout is improved interoperability: WebKit now passes 140,000 additional web platform test cases, narrowing gaps with other browsers in service workers, Fetch, pointer events, CSS, SVG, and WebAssembly. Performance improvements are equally substantial—page loads for recently visited sites are up to 42% faster, IndexedDB is up to 10x faster, `for-of` loops 5x faster, and scrolling CPU usage reduced by 3x.

New JavaScript APIs—BigInt, nullish coalescing (`??`), optional chaining (`?.`), logical assignment operators (`&&=`, `||=`, `??=`), public class fields, and `String.prototype.replaceAll`—arrive alongside a range of new Web APIs: Web Animations, ResizeObserver, Async Clipboard, EventTarget constructor, CSS Shadow Parts, and Web Authentication support for Touch ID/Face ID. Media capabilities expand with WebP image support, automatic image aspect ratio reservation, Remote Playback API, Picture in Picture API, timed metadata in HLS, and HDR video detection via `dynamic-range: high`. CSS gains system UI font families (`ui-sans-serif`, `ui-serif`, `ui-monospace`, `ui-rounded`), `:is()`, `:where()`, `line-break: anywhere`, and default `image-orientation: from-image`.

## Key Topics
- **Web Animations API** — `element.animate()` with keyframes and easing; query/seek/control animations; Web Inspector Graphics tab visualization **[NEW in Safari 13.1]**
- **ResizeObserver** — observe element size changes independent of viewport; toggle CSS classes reactively **[NEW in Safari 13.1]**
- **Async Clipboard API** — `navigator.clipboard.writeText/readText()` for plain text; `clipboard.read/write()` for rich types including HTML and images **[NEW in Safari 13.1]**
- **EventTarget constructor** — extend EventTarget for non-DOM objects in libraries **[NEW in Safari 13.1]**
- **CSS Shadow Parts** — `part` attribute on component internals; `::part()` pseudo-element for external styling **[NEW in Safari 13.1]**
- **Web Authentication / Touch ID & Face ID** — standards-based passkey login; hardware keys introduced in Safari 13; Touch ID/Face ID added in Safari 14 **[NEW]**
- **System font families** — `ui-sans-serif` (SF), `ui-serif` (New York), `ui-monospace` (SF Mono), `ui-rounded` (SF Rounded) **[NEW]**
- **WebP image support** — lossy and lossless; transparent; animated **[NEW in Safari 14]**
- **Remote Playback API** — `videoElement.remote.prompt()` for AirPlay and connected TV devices **[NEW]**
- **Picture in Picture API** — `videoElement.requestPictureInPicture()` across iOS/iPadOS/macOS **[NEW]**
- **Timed metadata in HLS** — `EXT-X-DATERANGE` tag and event message boxes in fragmented MP4 **[NEW in Safari 14]**
- **BigInt, optional chaining, nullish coalescing, logical assignment, public class fields, `replaceAll`** — all new in Safari 14

## APIs & Frameworks

**Web Animations API**
- `element.animate(keyframes, options)` — creates and plays an animation; returns `Animation` object **[NEW in Safari 13.1]**

**ResizeObserver**
- `new ResizeObserver(callback)` — observer; `observe(element)` — begin observing **[NEW in Safari 13.1]**
- `ResizeObserverEntry.contentRect.width` — observed element's new content width

**Async Clipboard API**
- `navigator.clipboard.writeText(string)` → `Promise` **[NEW in Safari 13.1]**
- `navigator.clipboard.readText()` → `Promise<string>`
- `navigator.clipboard.write([ClipboardItem])` / `navigator.clipboard.read()` — rich types (HTML, images)

**CSS (new in Safari 13.1 / Safari 14)**
- `font-family: ui-sans-serif | ui-serif | ui-monospace | ui-rounded` — system fonts **[NEW]**
- `line-break: anywhere` — break at any point before overflow **[NEW]**
- `:is(selector-list)` — matches list with highest specificity **[NEW in Safari 14]**
- `:where(selector-list)` — matches list with zero specificity **[NEW in Safari 14]**
- `image-orientation: from-image | none` — EXIF orientation; `from-image` is new default **[NEW]**
- `@media (dynamic-range: high)` — detect HDR displays **[NEW in Safari 14]**

**HTML / Markup**
- `<img width="…" height="…">` — browser now reserves aspect-ratio space before image loads **[NEW behavior in Safari 13.1]**
- `enterkeyhint="done|go|send|search|next|previous"` attribute — customizes virtual keyboard Enter key label **[NEW in Safari 13.4]**
- `<picture><source type="image/webp">` — WebP with fallback **[NEW in Safari 14]**
- `<meta name="apple-itunes-app" content="app-clip-bundle-id=…">` — App Clip smart banner **[NEW]**

**JavaScript (Safari 14)**
- `BigInt(value)` / `42n` literal — arbitrary-precision integers **[NEW]**
- `a ?? b` — nullish coalescing: returns `b` only if `a` is `null`/`undefined` **[NEW]**
- `a?.b` / `a?.[i]` / `a?.()` — optional chaining **[NEW]**
- `a &&= b` / `a ||= b` / `a ??= b` — logical assignment operators **[NEW]**
- Public class fields — `fieldName = defaultValue;` at class body level **[NEW]**
- `String.prototype.replaceAll(search, replacement)` — replace all occurrences **[NEW in Safari 13.1]**

**Media APIs**
- `videoElement.remote.prompt()` → `Promise` — Remote Playback API **[NEW]**
- `videoElement.requestPictureInPicture()` → `Promise<PictureInPictureWindow>` — PiP API **[NEW]**
- `window.matchMedia("(dynamic-range: high)")` — HDR detection in JavaScript **[NEW]**

**Web Authentication**
- `navigator.credentials.create()` / `.get()` with `publicKey` — FIDO2/WebAuthn **[NEW Touch ID/Face ID support in Safari 14]**

## Code Highlights

Web Animations API — spin a logo needle on click:
```javascript
logo.addEventListener("click", () => {
    needle.animate({
        transform: ["rotateX(35deg) rotateZ(13deg)", "rotateX(35deg) rotateZ(733deg)"],
        easing: ["ease-out"],
    }, 800);
});
```

ResizeObserver — collapse toolbar labels when container narrows:
```javascript
let formatPanelObserver = new ResizeObserver((entries) => {
    entries.forEach((entry) => {
        let container = entry.target;
        container.classList.toggle("small", entry.contentRect.width < 175);
    });
});
formatPanelObserver.observe(document.getElementById("format-panel"));
```

CSS `:is()` to simplify repetitive heading selectors:
```css
:is(h1, h2, h3, h4, h5, h6) + :is(h1, h2, h3, h4, h5, h6) {
    margin-top: 0;
}
```

Nullish assignment operator:
```javascript
element.innerHTML ??= "Hello World!";  // only assigns if innerHTML is null/undefined
```

## Takeaways
- Add explicit `width` and `height` attributes to all `<img>` tags; Safari 13.1+ uses them to compute an aspect ratio box and reserve space before the image loads, eliminating layout shift.
- Use `ui-sans-serif`, `ui-serif`, `ui-monospace`, and `ui-rounded` font families when building web apps that should feel at home on Apple platforms; they map to SF, New York, SF Mono, and SF Rounded automatically.
- The nullish coalescing (`??`) and optional chaining (`?.`) operators—now in Safari 14—simplify defensive null/undefined guards; pair them with logical assignment (`??=`) to write concise, non-destructive default assignments.
- Adopt the Remote Playback and Picture in Picture APIs to give users standards-based controls for AirPlay and PiP in custom web video players; both require only a single method call in response to user interaction.

---
_Source: WWDC20 Session 10663 page (transcript, code samples, and resource links)._
