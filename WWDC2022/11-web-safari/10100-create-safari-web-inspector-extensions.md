# Create Safari Web Inspector Extensions
**WWDC22 · Session 10100** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10100/)

_Platforms:_ macOS Ventura 13, Safari 16

## Overview
Safari 16 introduces the ability to add custom tabs to Web Inspector using the standard Web Extensions DevTools APIs. This session walks through building an "Open Graph" Web Inspector extension that extracts social metadata from the inspected page and displays it in a custom panel within Web Inspector.

The session covers the structural differences between a standard Safari Web Extension and a Web Inspector extension — specifically the addition of a devtools background page and devtools tab page — and explains how each page instance is scoped to a specific Web Inspector window. The core developer-facing API is `browser.devtools.inspectedWindow.eval()`, which evaluates JavaScript in the context of the page being inspected and returns results to the extension's devtools tab.

Best practices around permissions (prefer `activeTab` over broad host permissions), theme matching via CSS `color-scheme`, and inline permission prompting via devtools panels are also covered.

## Key Topics

### Web Inspector Extension Architecture
A Web Inspector extension extends the standard Web Extension manifest with a `devtools_page` entry pointing to a devtools background HTML page. This page:
- Is instantiated once per open Web Inspector window (multiple Web Inspectors = multiple instances)
- Has access to the `browser.devtools.*` APIs and a limited subset of content script APIs
- Creates devtools tab pages via `browser.devtools.panels.create()`

Devtools tab pages are the visible UI shown within Web Inspector. They are created from the devtools background page and can execute JavaScript in the inspected page via `browser.devtools.inspectedWindow.eval()`.

### Creating a Devtools Panel
`browser.devtools.panels.create(name, iconPath, pagePath)` registers a new tab in Web Inspector. The icon should be an SVG for resolution independence. The panel HTML loads the tab's scripts and styles. Permissions for the extension are surfaced inline within the new tab, matching the same permission duration options as regular extensions.

### Evaluating JavaScript in the Inspected Page
`browser.devtools.inspectedWindow.eval(expression, options)` is the preferred way to run JavaScript against the inspected page from a Web Inspector extension. It automatically targets the page associated with the current Web Inspector instance (important for multi-window debugging). The `frameURL` option scopes execution to a specific sub-frame rather than the main frame.

### Listening for Navigation
`browser.devtools.network.onNavigated.addListener(callback)` fires whenever the inspected page navigates, allowing the extension to re-evaluate and refresh its displayed data.

### Theming and Localization
Use `color-scheme: light dark` in CSS to match Web Inspector's appearance automatically. Extension localization strings (`browser.i18n.getMessage()`) work the same inside devtools tab pages as in other extension contexts.

### Distribution
Web Inspector extensions are distributed as Safari App Extensions via a Mac app in the App Store. The Xcode project template "Safari Extension App" provides the scaffolding; existing cross-browser extensions can be imported using `safari-web-extension-converter`.

## APIs & Frameworks

### Web Extensions DevTools APIs (JavaScript)
- `browser.devtools.panels.create(name, iconPath, pagePath) -> Promise<ExtensionPanel>` **[NEW in Safari 16]** — registers a new tab in Web Inspector
- `browser.devtools.inspectedWindow.eval(expression, options?) -> Promise<[result, exceptionInfo]>` **[NEW in Safari 16]** — evaluates JavaScript in the inspected page's context
  - `options.frameURL: string` — evaluate in a specific sub-frame rather than the main frame
- `browser.devtools.network.onNavigated.addListener(callback)` **[NEW in Safari 16]** — fires when the inspected page navigates
- `browser.devtools.inspectedWindow.tabId` — ID of the tab being inspected

### Web Extensions Standard APIs (available in devtools context)
- `browser.i18n.getMessage(messageName)` — retrieves a localized string
- `activeTab` permission — preferred over broad host permissions for Web Inspector extensions

### Manifest Keys
- `"devtools_page": "devtools_background.html"` **[NEW]** — declares the devtools background page
- `"icons"` — extension icons; SVG recommended for the devtools panel icon

### Xcode Tooling
- `safari-web-extension-converter` CLI — converts an existing cross-browser extension to a Safari app project

## Code Highlights

Creating a devtools panel from the devtools background script:
```javascript
// devtools_background.js
browser.devtools.panels.create(
    browser.i18n.getMessage("extension_name"),
    "images/logo.svg",
    "devtools_tab.html"
);
```

Evaluating JavaScript in the inspected page and processing the result:
```javascript
// devtools_tab.js
async function fetchOpenGraphData() {
    const [result, exception] = await browser.devtools.inspectedWindow.eval(`({
        title: document.querySelector('meta[property="og:title"]')?.content,
        description: document.querySelector('meta[property="og:description"]')?.content,
        image: document.querySelector('meta[property="og:image"]')?.content,
        readyState: document.readyState
    })`);
    if (exception) return;
    if (result.readyState !== "complete") {
        setTimeout(fetchOpenGraphData, 500);
        return;
    }
    displayData(result);
}
browser.devtools.network.onNavigated.addListener(fetchOpenGraphData);
fetchOpenGraphData();
```

Evaluating inside a specific sub-frame:
```javascript
const result = await browser.devtools.inspectedWindow.eval("foo.bar()", {
    frameURL: "http://example.com/",
});
```

CSS for matching Web Inspector's theme:
```css
:root {
    color-scheme: light dark;
    font-family: system-ui;
}
```

## Takeaways
- Safari 16 supports Web Inspector extensions using the cross-browser DevTools API (`browser.devtools.*`), enabling custom panels distributable via the App Store.
- The devtools background page is instantiated per Web Inspector window; always create panels from it so permissions are shown inline in the correct Web Inspector.
- `browser.devtools.inspectedWindow.eval()` is the correct API for running code in the inspected page — use `frameURL` when targeting sub-frames.
- Prefer the `activeTab` permission over broad host permissions, and use `color-scheme: light dark` to match Web Inspector's appearance automatically.

---
_Source: WWDC22 Session 10100 page (abstract, chapter summaries, code samples, and resource links)._
