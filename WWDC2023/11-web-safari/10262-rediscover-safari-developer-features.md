# Rediscover Safari developer features
**WWDC23 · Session 10262** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10262/)

_Platforms:_ macOS Sonoma 14, iOS 17, iPadOS 17, visionOS 1, tvOS 17

## Overview
This session provides a comprehensive tour of Safari's developer tooling for web developers and designers. It covers the full workflow from local Mac inspection through cross-device debugging on iPhone, iPad, Apple TV, and — new in 2023 — visionOS devices. Key highlights include a redesigned Responsive Design Mode with direct "Open with Simulator" integration (including visionOS simulators), wireless pairing with visionOS devices, the ability to make `WKWebView` and `JSContext` content inspectable in release builds, and a redesigned Feature Flags panel replacing the old Experimental Features menu.

The session is an onboarding resource for developers who have not used Safari's built-in developer tools before, while also surfacing new capabilities for experienced users. No web-specific APIs are introduced; the focus is entirely on tooling and workflow.

## Key Topics

### Enabling Developer Features
Safari developer tools are hidden by default. Enable them at Safari > Settings > Advanced > "Show features for web developers." This unlocks the Develop menu, Web Inspector, Responsive Design Mode, and Feature Flags.

### Web Inspector
Opened via Develop menu ("Show Web Inspector") or via right-click > "Inspect Element." The element selection tool shows margin, shape outlines, and accessibility roles on hover. The Styles sidebar includes an integrated color picker with on-screen sampling. The Develop menu has been redesigned for easier navigation to key tools.

### Responsive Design Mode (Redesigned)
Accessed from Develop > "Enter Responsive Design Mode." Supports viewport sizes larger than the physical screen. Live resize handles, editable width/height fields, scale factor display, and pixel ratio (device pixel ratio / DPR) controls. Used to test HTML `srcset`, CSS `image-set()`, and CSS `@media (min-resolution: ...)` queries.

### Open with Simulator (New)
A new "Open with Simulator" button in Responsive Design Mode lets developers launch the current page directly in any installed iOS, iPadOS, or visionOS simulator. Running simulators appear first in the list. Xcode (free) is required; the menu includes links to setup docs if Xcode is not installed.

### Device Inspection (iOS, iPadOS, tvOS, visionOS)
Web content on connected devices appears in the Develop menu. Enable on iOS/iPadOS: Settings > Safari > Advanced > Web Inspector. Connect via cable, then optionally enable "Connect via Network" for wireless inspection (same Wi-Fi network required).

### visionOS Pairing (New)
visionOS devices cannot connect via cable; instead they pair over the network. Steps: Settings > Apps > Safari > Advanced > enable Web Inspector; Settings > General > Remote Devices (keep screen visible); on Mac, Develop menu > device submenu > "Use for Development"; enter the 6-digit code shown on device. Once paired, web content and even element selection mode work remotely.

### Inspectable WKWebView and JSContext in Release Builds (New)
Previously, in-app web content was only inspectable in debug builds. In macOS 13.3, iOS/iPadOS 16.4, and visionOS 1, `WKWebView.isInspectable` and `JSContext.isInspectable` allow opting in to inspection in release builds. Providing a `name` to `JSContext` makes it identifiable in the Develop menu when multiple contexts are active.

### WebDriver for Automated Testing
Safari includes a local HTTP WebDriver server. Works cross-browser and cross-platform via the W3C WebDriver standard. Integrates with Selenium (Python, Java, PHP, JavaScript, etc.). Automated windows show an orange title bar. Tests can run against macOS Safari, iOS/iPadOS simulators, and physical iOS/iPadOS devices.

### Feature Flags (Redesigned, replacing Experimental Features)
Develop > Feature Flags. Organized by topic (Animation, CSS, JavaScript, Media, etc.) and searchable. Four status categories:
- **Stable** — recently shipped, on by default; toggle to test graceful degradation
- **Testable** — in-progress, off by default; early feedback for standards
- **Preview** — nearly complete, off by default in Safari but on by default in Safari Technology Preview
- **Developer** — internal WebKit behavior toggles and deprecated API re-enablement
Enabled features are shown in bold. All feature flags reset to defaults on Safari update.

## APIs & Frameworks

- **Web Inspector** — element inspector, Styles sidebar, color picker with screen sampling, network tab, console, etc.
  - "Show Web Inspector" in Develop menu
  - "Inspect Element" in right-click context menu
  - Element selection mode
- **Responsive Design Mode** — viewport size, DPR simulation **[REDESIGNED]**
  - Viewport width/height fields
  - Scale factor display
  - Pixel ratio selector
  - "Open with Simulator" button **[NEW]**
- **Simulator integration** — iOS, iPadOS, visionOS simulators (requires Xcode)
- **Device inspection** — iOS, iPadOS, tvOS, visionOS via Develop menu
  - Wired connection (Lightning / USB-C)
  - "Connect via Network" wireless inspection
  - Remote Devices + 6-digit pairing code for visionOS **[NEW]**
- `WKWebView.isInspectable: Bool` — opt in to Safari inspection in release builds **[NEW]** (macOS 13.3, iOS/iPadOS 16.4, visionOS 1)
- `JSContext.isInspectable: Bool` — opt in to Safari inspection for JavaScript contexts **[NEW]** (macOS 13.3, iOS/iPadOS 16.4, tvOS 16.4, visionOS 1)
- `JSContext.name: String?` — display name shown in Safari's Develop menu
- **WebDriver** — W3C standard cross-browser automation protocol
  - safaridriver (local HTTP server)
  - Selenium Python bindings: `webdriver.Safari()`, `driver.get()`, `driver.find_element()`, `element.send_keys()`, `driver.quit()`
- **Feature Flags** panel — Develop > Feature Flags **[REDESIGNED from Experimental Features]**
  - Status categories: Stable, Testable, Preview, Developer
  - Searchable feature list organized by topic
- **Safari Technology Preview** — biweekly release with Preview-status features enabled by default
- HTML `srcset` attribute — responsive images
- CSS `image-set()` function — responsive background images
- CSS `@media (min-resolution: Ndppx)` — DPR-based styling

## Code Highlights

Making WKWebView and JSContext inspectable in release builds:
```swift
let webView = WKWebView(frame: .zero, configuration: WKWebViewConfiguration())
if #available(macOS 13.3, iOS 16.4, *) {
    webView.isInspectable = true
}

let jsContext = JSContext()
jsContext?.name = "My Context"
if #available(macOS 13.3, iOS 16.4, tvOS 16.4, *) {
    jsContext?.isInspectable = true
}
```

WebDriver automated test with Selenium (Python):
```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.safari.options import Options as SafariOptions

options = SafariOptions()
driver = webdriver.Safari(options=options)
driver.get("https://webkit.org/web-inspector/")
search_element = driver.find_element(by=By.ID, value="search")
search_element.send_keys("device")
assert(driver.find_element(by=By.LINK_TEXT, value="Device Settings"))
driver.quit()
```

## Takeaways

- Enable Safari developer features once at Safari > Settings > Advanced; this unlocks the complete tooling suite including Web Inspector, Responsive Design Mode, Feature Flags, and device inspection.
- The new "Open with Simulator" button in Responsive Design Mode makes it trivial to test in iOS, iPadOS, and visionOS simulators directly from a desktop browser session.
- Set `WKWebView.isInspectable = true` and `JSContext.isInspectable = true` in release builds to enable inspection of in-app web content — gate it on a debug flag or developer setting so it is not on for all users.
- visionOS device debugging requires network pairing (no cable support); the Develop menu > "Use for Development" flow with a 6-digit code replaces the physical connection step.

---
_Source: WWDC23 Session 10262 page (abstract, chapter summaries, code samples, and resource links)._
