# What's New in Safari and WebKit
**WWDC22 · Session 10048** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10048/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13

## Overview
Safari and WebKit delivered 162 new web platform features and improvements across seven releases in the past year. This session covers the headline additions: HTML features like the `<dialog>` element and `inert` attribute, major CSS enhancements including container queries, cascade layers, `:has()`, subgrid, new viewport units, offset-path, and typography improvements, significant Web Inspector upgrades (Flexbox Inspector, alignment editor, CSS fuzzy autocompletion, and Developer Tool Extensions support), Web APIs like Web Push, Broadcast Channel, File System Access, and Display P3 canvas colors, JavaScript/WebAssembly improvements, and security/privacy enhancements.

Container queries shipping in Safari 16 is the top-requested web feature, enabling component-level responsive layouts based on container size rather than viewport size. Web Push launches in Safari 16 on macOS Ventura with full standards interoperability, coming to iOS/iPadOS in 2023.

## Key Topics

### HTML
- **`<dialog>` element** — accessible modal overlay with `::backdrop` pseudo-element for styling
- **`inert` attribute** — disables all interactions (including assistive technologies) on non-active elements
- **Lazy image loading** — `loading="lazy"` on `<img>` defers off-screen image loads

### CSS Architecture
- **Container queries (Safari 16)** — `@container` rules; `container-type` and `container-name` properties; both size queries and container query units
- **Cascade layers** — `@layer` rule; independent specificity per layer; layer order determines precedence
- **`:has()` pseudo-class** — parent/relational selector; works with siblings, attributes, form states

### CSS Layout and Visual
- **Subgrid** — `grid-template-rows: subgrid` or `grid-template-columns: subgrid` ties child grids to parent grid tracks
- **New viewport units** — `svh`/`svw` (small), `lvh`/`lvw` (large), `dvh`/`dvw` (dynamic); also block, inline, min, max variants
- **`offset-path`** — animate elements along SVG-style paths; with `offset-distance` in keyframes
- **`scroll-behavior`** — `smooth` or `auto` for anchor link scrolling
- **`:focus-visible`** — style focus indicator while preserving browser accessibility heuristics
- **`accent-color`** — tint native form controls (checkboxes, radios, etc.)
- **`font-palette`** — select/customize color palettes in color fonts (`dark`, `light`, `@font-palette-values`)
- **`text-decoration-skip-ink`** — control underline/overline intersection with characters
- **`ic` unit** — CJK character grid alignment unit
- WebKit prefix removals: `backface-visibility`, `print-color-adjust`, `text-align: match-parent`, `mask`, `text-combine-upright`, `appearance`

### Web Inspector
- **Flexbox Inspector** — visualize flex container spacing, alignment, and gaps
- **Alignment editor** — toggle `align-items` and `justify-content` values inline in Styles tab
- **CSS fuzzy autocompletion** — autocomplete CSS variable names by substring
- **Developer Tool Extensions** — Safari Web Inspector now supports third-party extensions using the same APIs as other browsers

### Web APIs
- **Web Push (Safari 16 on macOS Ventura)** — fully standards-based (`PushManager`, service workers); no Apple Developer account required; iOS/iPadOS 16 support coming in 2023
- **Web App Manifest icon** — `icons` in manifest now controls Home Screen icon; `apple-touch-icon` for Apple-specific override
- **Broadcast Channel** — `BroadcastChannel.postMessage()` for cross-tab/window communication
- **File System Access API** — origin private file system (`navigator.storage.getDirectory()`), `FileSystemFileHandle.getFile()`, `getFileHandle()` with `create`
- **Display P3 canvas colors** — P3 wide-gamut color support in `<canvas>`
- **Shadow Realms** — isolated JS execution environments
- **Web Locks API** — coordinate resource access across tabs/workers
- **`ResizeObserverSize` interface** — observe changes to element box-sizing properties
- **`getUserDisplay()`** — capture a specific Safari window via screen capture
- **WebRTC Perfect Negotiation** — standardized offer/answer renegotiation
- **In-band chapter tracks** — video chapter support
- **`requestVideoFrameCallback()`** — per-frame video rendering callback

### JavaScript and WebAssembly
- **Shared Workers** — `SharedWorker` interface for a single worker shared across browsing contexts with the same origin
- **`Array.prototype.findLast()`** / **`findLastIndex()`** — search arrays from the end
- **`Array.prototype.at()`** — negative index array access
- **`Intl` enhancements** — `NumberFormat` numbering systems, `Locale` calendars, `DisplayNames`, `Intl.Enumeration` API for currencies
- **WebAssembly** — 4GB addressable memory; zero-cost exception handling

### Security and Privacy
- **Cross-Origin Opener Policy (COOP)** + **Cross-Origin Embedder Policy (COEP)** headers — opt-in to process isolation; enables secure WebAssembly threading
- **Content Security Policy Level 3** — `strict-dynamic` source expression; nonce-based trust propagation

## Code Highlights

```css
/* Container queries */
.container { container-type: inline-size; container-name: clothing-card; }
@container clothing-card (width > 250px) {
    .content { grid-template-columns: 1fr 1fr; }
}

/* :has() parent selector */
form:has(input[type="checkbox"]:checked) { background: #ff927a; }

/* Subgrid */
article { display: grid; grid-row: span 5; grid-template-rows: subgrid; }

/* offset-path animation */
.element { offset-path: circle(9vw at 5vw 50%); }
@keyframes move { 100% { offset-distance: 100%; } }
```

```js
// File System Access API
const root = await navigator.storage.getDirectory();
const handle = await root.getFileHandle("Draft.txt", { create: true });

// Broadcast Channel
const channel = new BroadcastChannel("updates");
channel.postMessage("Item is unavailable");

// Shared Worker
const worker = new SharedWorker("SharedWorker.js");
worker.port.postMessage("hello");
```

## Takeaways

- Container queries ship in Safari 16 — use `container-type`, `container-name`, and `@container` to build truly responsive components regardless of where they are placed.
- Cascade layers (`@layer`) and `:has()` provide powerful CSS architecture tools for large projects; `:has()` acts as a long-awaited parent/relational selector with no JavaScript needed.
- Web Push is now fully interoperable in Safari 16 on macOS Ventura; implement once and it works across browsers with no Apple Developer account required.
- Web Inspector gains Flexbox visualization, alignment editing, CSS fuzzy autocomplete, and third-party Developer Tool Extension support — use them to streamline layout debugging.

---
_Source: WWDC22 Session 10048 page (abstract, chapter summaries, code samples, and resource links)._
