# What's New in WebKit for Safari 27

**WWDC26 · Session 204** · [Watch](https://developer.apple.com/videos/play/wwdc2026/204/)

_Platforms:_ Safari 27, WebKit, iOS 27, iPadOS 27, macOS 27, visionOS 27, web standards

## Overview

Safari 27 and WebKit represent a year of concentrated quality work, shipping over 1,100 browser engine fixes and improvements across compatibility, foundational layout algorithms, SVG rendering, web standards alignment, and cross-feature integration. The WebKit team identified and resolved long-standing bugs — such as an emoji keyboard input regression — by rebuilding lower-level layout foundations like block-in-inline handling, delivering reliability improvements that cascade across many sites without requiring developer changes.

On top of quality, Safari 27 ships several headline new features: CSS Grid Lanes for masonry-style layouts, Customizable Select for fully styled yet accessible form controls, the HTML `<model>` element expanding from visionOS to all Apple platforms, and immersive website environments on visionOS powered by a new Immersive API. Web Extensions now have a streamlined submission path that requires no Mac or Xcode, and MapKit JS continues to offer privacy-preserving interactive maps across all browsers.

The session serves as a tour of the year's work and points developers toward the companion deep-dive sessions covering each new feature in detail.

## Key Topics

### A Year of Quality Improvements (1:07)
The WebKit team organized quality investment around five themes:

- **Compatibility** — tracked and fixed real-world site breakage, including an emoji input bug that required rebuilding block-in-inline layout from scratch.
- **Rebuilding foundations** — block-in-inline layout rewrite cascaded fixes to many unrelated-seeming rendering issues.
- **Going deep on subsystems** — SVG received particular focus with many spec-compliance and rendering improvements.
- **Web standards alignment** — implemented and updated behavior to match the latest WHATWG/W3C specifications.
- **Feature integration** — ensured newer capabilities (like container queries, cascade layers) work correctly together and with older features.

### CSS Grid Lanes (9:06)
Shipped in Safari 26.4. A new CSS layout mode (`display: grid-lanes`) enabling masonry-style "waterfall" and "brick wall" designs without JavaScript. Items pack into the shortest column automatically. See Session 314 for the full deep dive.

### Customizable Select (10:06)
A new `appearance: base-select` CSS value opts `<select>` elements into a fully styleable mode. New pseudo-elements (`::picker-icon`, `::picker(select)`, `::checkmark`) and the `:open` pseudo-class give fine-grained control. Arbitrary HTML content can be placed inside `<option>` elements. See Session 315 for the full deep dive.

### HTML Model Element (11:24)
The `<model>` element, previously visionOS-only, now ships on iOS, iPadOS, and macOS in Safari 27. It embeds interactive 3D USDZ models directly in web pages via HTML attributes and JavaScript. See Session 215 for the full deep dive.

### Immersive Website Environments (12:51)
In visionOS 27, `<model>` elements can launch into a full immersive environment using a new Immersive API modeled after the Fullscreen API. Use cases include immersive game previews and venue seat-view experiences.

### Web Extensions (13:38)
Safari Web Extensions can now be submitted to the App Store via App Store Connect from any browser on any operating system — no Mac or Xcode required. The Safari Web Extension Packager automates packaging. See Session 216 for the full deep dive.

### MapKit JS (15:18)
MapKit JS provides privacy-preserving interactive maps embeddable on any website, working across all browsers and operating systems.

## APIs & Frameworks

**CSS Layout**
- **[NEW]** `display: grid-lanes` — new Grid Lanes layout mode
- **[NEW]** `flow-tolerance` property for Grid Lanes item ordering
- `grid-template-columns`, `grid-template-rows` — supported in Grid Lanes context
- `gap` — supported in Grid Lanes

**CSS Customizable Select**
- **[NEW]** `appearance: base-select` — opt-in to fully styleable select
- **[NEW]** `::picker-icon` pseudo-element — the dropdown arrow
- **[NEW]** `::picker(select)` pseudo-element — the popup listbox
- **[NEW]** `::checkmark` pseudo-element — option checkmark
- **[NEW]** `:open` pseudo-class — select open state
- `<selectedcontent>` element — mirrors rich content of selected option into button

**HTML Model Element**
- **[NEW]** `<model>` element — iOS, iPadOS, macOS support (was visionOS-only)
- `src` attribute on `<model>`
- `<source>` child element with `type` attribute (MIME type)
- `stagemode="orbit"` attribute
- `entityTransform` property (DOMMatrix)
- `ready` promise
- `play()` method, `playbackRate` property
- `HTMLModelElement` interface (W3C)

**Immersive API (visionOS 27)**
- **[NEW]** Immersive API for `<model>` — modeled on the Fullscreen API
- Launches `<model>` content into a full immersive visionOS environment

**Web Extensions**
- **[NEW]** Safari Web Extension Packager — cross-platform, no Xcode required
- App Store Connect submission from any OS/browser
- `browser.declarativeNetRequest` API
- `browser.scripting` API
- `browser.storage` API
- `browser.permissions` API
- `browser.runtime.sendNativeMessage()` / `SFExtensionMessageKey`
- Manifest v3

**MapKit JS**
- MapKit JS — cross-browser interactive maps with privacy preservation

**WebKit Quality / Standards**
- Block-in-inline layout rebuild
- SVG rendering improvements
- WHATWG/W3C spec alignment across multiple subsystems

## Code Highlights

Grid Lanes container (3 lines):
```css
.container {
  display: grid-lanes;
  grid-template-columns: repeat(3, 1fr);
  gap: 10px;
}
```

Customizable Select opt-in:
```css
select {
  appearance: base-select;
}
::picker(select) {
  appearance: base-select;
}
```

Model element embed:
```html
<model src="product.usdz">
  <img src="product.png" alt="Product fallback">
</model>
```

## Takeaways

- Safari 27 shipped 1,100+ fixes; a foundational block-in-inline layout rewrite underlies broad compatibility gains.
- Three major new authoring features — Grid Lanes, Customizable Select, and `<model>` — each have companion deep-dive sessions (314, 315, 215).
- Web Extension submission now requires no Mac or Xcode; any developer on any OS can publish to Safari users via App Store Connect.
- visionOS 27 introduces an Immersive API that lets `<model>` elements expand into fully immersive spatial environments, mirroring the pattern of the Fullscreen API.

---
_Source: WWDC26 Session 204 page (abstract, chapter summaries, code samples, and resource links)._
