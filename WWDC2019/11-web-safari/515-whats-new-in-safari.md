# What's New in Safari
**WWDC19 · Session 515** · [Watch](https://developer.apple.com/videos/play/wwdc2019/515/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15 (Safari 13)

## Overview
This session covers three areas of change in Safari 13: desktop-class browsing on iPad (implemented at the WebKit/WKWebView layer and covered in detail in Session 203), a significant round of new Safari App Extension APIs, and guidance for Mac app developers on link-following behavior including the arrival of universal links on macOS.

Legacy Safari extensions (the old 2010-era format) are dropped in Safari 13, and the session makes clear that content blockers and Safari App Extensions are the supported path forward for all extension needs.

## Key Topics

### Desktop-Class Browsing on iPad
- iOS 13 makes fundamental changes so that iPad requests desktop websites by default.
- `SFSafariViewController` gets desktop-class browsing automatically.
- Apps using `WKWebView` or custom in-app browsers should consult Session 203 ("Introducing Desktop-class Browsing on iPad") for detailed guidance and best practices.
- The iPad User-Agent now identifies as desktop (Mac), which has implications for web content and MDM/enrollment flows.

### Legacy Safari Extensions Removed
- Legacy Safari extensions (introduced 2010, deprecated 2018) are no longer loaded in Safari 13.
- All legacy extensions must be migrated to content blockers, share extensions, or Safari App Extensions.
- Migration guide available on developer.apple.com.

### Safari App Extensions — New APIs (macOS, Safari 13)

**Windows and Tabs APIs** **[NEW]**
- Navigate a tab directly from the app extension process to full-page content (e.g., a dashboard).
- Enumerate all open windows and tabs (e.g., for bookmarking services).
- Get a reference to the containing tab and window when handling a message from an injected script.
- Retrieve the visible contents of a page (e.g., for custom visual tab representations).
- Programmatically show and dismiss popovers at the appropriate moment.

**Page Navigation Notifications** **[NEW]**
- New callback notifying the extension of the full redirect chain when a page navigates.
- Use case: redirect users to a different version of a website before it loads.

**Content Blocker Association and Notifications** **[NEW]**
- Associate a content blocker with a Safari App Extension so the extension receives callbacks when content is blocked.
- Enables extensions to display statistics on blocked trackers, cryptocurrency miners, and other scripts.
- Users can enable the app extension independently from the content blocker for maximum privacy (content blocker runs alone) or enable both for statistics.

### Link Following on macOS

**iPad Apps on Mac (Catalyst)**
- On macOS, in-app web browsing via `SFSafariViewController` automatically opens links in the user's default browser and immediately calls `safariViewControllerDidFinish` — matching native macOS behavior.
- Apps with custom in-app browsers should open links directly in the system browser on macOS rather than presenting a web view.

**Universal Links on macOS** **[NEW]**
- Universal links (HTTPS URLs that open in the corresponding app if installed) now work on macOS Catalina.
- Initial flow: link opens in Safari; if the corresponding app is installed, Safari shows a banner allowing the user to open in the app; subsequent follows go directly to the app.
- Strongly preferred over custom URL schemes (which fail silently when the app is not installed).
- Details in Session 717 ("What's New in Universal Links").

## APIs & Frameworks

### SafariServices
- `SFSafariViewController` — on Mac (Catalyst): automatically opens links in the system browser, calls `safariViewControllerDidFinish` immediately **[NEW behavior]**
- `SFContentBlockerManager` — manage content blockers

### Safari App Extensions (macOS, `SafariServices`)
- `SFSafariApplication` — top-level application object
- `SFSafariApplication.getAllWindows(completionHandler:)` **[NEW]** — enumerate open windows
- `SFSafariWindow` — represents a Safari window
- `SFSafariWindow.getAllTabs(completionHandler:)` **[NEW]** — enumerate tabs in a window
- `SFSafariTab` — represents a tab
- `SFSafariTab.navigate(to:)` **[NEW]** — navigate tab to a URL from the extension process
- `SFSafariTab.getContainingWindow(completionHandler:)` **[NEW]**
- `SFSafariPage.getPropertiesWithCompletionHandler(_:)` — get page properties
- `SFSafariPage.getScreenshotWithCompletionHandler(_:)` **[NEW]** — get visible page contents as image
- `SFSafariExtensionHandler` — base class for extension handler
- `SFSafariExtensionHandler.page(_:willNavigateTo:)` **[NEW]** — redirect chain callback
- `SFSafariExtensionHandler.contentBlocker(_:blockedResourcesWith:on:)` **[NEW]** — called when content blocker blocks resources
- `SFSafariExtensionViewController.dismissPopover()` **[NEW]** — programmatic popover dismiss
- `SFSafariApplication.showPopover()` **[NEW]** — programmatic popover show
- Content blocker ↔ app extension association configured in extension's `Info.plist` **[NEW]**

### WebKit / WKWebView
- `WKWebView` — automatically serves desktop websites on iPad in iOS 13 (via updated default user-agent and viewport behavior); see Session 203 for full API guidance

### Foundation / NSWorkspace
- Universal links on macOS: handled via `NSWorkspace.open(_:)` / Associated Domains entitlement (same mechanism as iOS); Safari shows upgrade banner for installed apps **[NEW on macOS]**

## Code Highlights

Handling content-blocker notifications in a Safari App Extension:
```swift
// In SFSafariExtensionHandler subclass
override func contentBlocker(withIdentifier contentBlockerIdentifier: String,
                              blockedResourcesWith urls: [URL],
                              on page: SFSafariPage) {
    // Update statistics, badge toolbar item, etc.
    page.getPropertiesWithCompletionHandler { properties in
        // Correlate blocked resources with the page URL
        let pageURL = properties?.url
        // Update your blocked-content statistics store
    }
}
```

Navigating a tab to a dashboard page:
```swift
SFSafariApplication.getAllWindows { windows in
    windows.first?.getAllTabs { tabs in
        tabs.first?.navigate(to: URL(string: "https://myextension.example/dashboard")!)
    }
}
```

Getting visible page screenshot:
```swift
page.getScreenshotWithCompletionHandler { screenshot in
    guard let image = screenshot else { return }
    // Use NSImage for custom tab thumbnail display
}
```

## Takeaways
- Legacy Safari extensions are gone in Safari 13; migrate immediately to content blockers or Safari App Extensions.
- Safari App Extensions gain meaningful new power: full window/tab enumeration and navigation, programmatic popovers, page screenshots, redirect chain callbacks, and content-blocker statistics.
- iPad apps on Mac should let `SFSafariViewController` open links in the system browser, or use direct `NSWorkspace.open(_:)` calls; do not present inline web views.
- Universal links now work on macOS, making HTTPS-based deep linking the correct cross-platform approach across iPhone, iPad, and Mac.

---
_Source: WWDC19 Session 515 page (abstract, chapter summaries, code samples, and resource links)._
