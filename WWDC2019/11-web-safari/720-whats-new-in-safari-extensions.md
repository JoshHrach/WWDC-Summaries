# What's New in Safari Extensions
**WWDC19 · Session 720** · [Watch](https://developer.apple.com/videos/play/wwdc2019/720/)

_Platforms:_ macOS Catalina 10.15 (Safari 13)

## Overview
This session is a deep technical dive into the new Safari App Extension and Content Blocker APIs introduced in Safari 13 for macOS. It picks up where Session 515 left off and provides concrete code examples for every major new capability. The four main areas are: content-blocker notification to a paired Safari App Extension, improvements to window/tab/page management APIs, better popover control, and best practices for bidirectional communication between a Safari Extension and its containing Mac app.

A running demo ("Animalify") is used throughout to show all the new APIs in context, culminating in a popover that shows a screenshot of each open tab alongside its blocked-resource count.

## Key Topics

### Distribution and Discovery
- Safari App Extensions and Content Blockers are bundled with Mac apps built in Xcode.
- Mac App Store apps: extensions appear immediately in Safari Preferences once the app is installed.
- Notarized apps distributed from a website: the app must be launched at least once for its extensions to appear in Safari.
- For development testing: enable the Develop menu > Allow Unsigned Extensions.
- Xcode's Safari Extension App template provides a free splash screen with a button that takes users to Safari's extension preferences.

### Content Blocker Notifications **[NEW]**
- A Safari App Extension can now be notified when its associated Content Blocker blocks a resource.
- Setup: add `SFSafariAssociatedContentBlockers` key (array of bundle identifiers) to the `NSExtension` section of the Safari App Extension's `Info.plist`. Content Blocker and Safari App Extension must be in the same containing app.
- Implement `contentBlocker(withIdentifier:blockedResourcesWith:on:)` on the extension's principal object to receive batched notifications.
- Notifications are only sent for URLs the extension has permission to access (per its `website access` `Info.plist` section).
- Users can enable the app extension independently of the content blocker (for statistics) or run the content blocker alone (most private).

### Page Navigation Notifications **[NEW]**
- `page(_:willNavigateTo:)` **[NEW]** — called on the extension's principal object when a page is about to navigate.
- Called even when the extension does not have access to the target URL; in that case `url` is `nil`.
- `url` is also `nil` when the user opens favorites or history.
- Use cases: tracking the redirect chain across a page load, resetting per-page blocked-resource counts, redirecting to a specific website version before navigation completes.

### Window, Tab, and Page Management **[NEW]**
- `SFSafariApplication.getAllWindows(completionHandler:)` **[NEW]** — enumerate all open Safari windows.
- `SFSafariWindow.getAllTabs(completionHandler:)` **[NEW]** — enumerate all tabs in a window.
- `SFSafariTab.getContainingWindow(completionHandler:)` **[NEW]** — get the window containing a tab (returns `nil` for pinned tabs, which belong to all windows).
- `SFSafariPage.getContainingTab(completionHandler:)` **[NEW]** — get the tab containing a page.
- `SFSafariPage.getScreenshot(completionHandler:)` **[NEW]** — get the visible contents of the page as image data (requires extension to have URL access).
- `SFSafariTab.navigate(to:)` **[NEW]** — navigate a tab to a URL directly, without an injected script.
- Extension base URL now available from native code via `Bundle.main.resourceURL` (no longer requires injected script).
- Safari now injects the Safari JavaScript object into frames loaded with content from the extension bundle, enabling those frames to send/receive messages to/from the extension.

### Popover Improvements **[NEW]**
- `SFSafariToolbarItem.showPopover()` **[NEW]** — programmatically show the extension popover from native code.
  - Obtain the toolbar item from the `SFSafariWindow` that will present it.
- `SFSafariExtensionViewController.dismissPopover()` **[NEW]** — dismiss the popover from within the popover's view controller.
- `SFSafariExtensionHandler.popoverWillShow(in:)` — good place to collect window/tab state before the popover appears.

### App–Extension Communication (Best Practices)
- The extension may be launched without the containing app being running.
- **App Group** (recommended for data sharing): add both the extension and the app to the same App Group; use `UserDefaults(suiteName:)` to share preferences and state; use `NSXPCConnection` with named mock services for IPC.
- **App → Extension messaging** (one-directional trigger): `SFSafariApplication.dispatchMessage(withName:toExtensionWithIdentifier:userInfo:completionHandler:)` — sends a message to the extension, launching Safari if needed. Extension must be enabled; must be in the same bundle as the calling code.
- **Extension receiving app message**: implement `messageReceivedFromContainingApp(withName:userInfo:)` on the extension's principal object.

## APIs & Frameworks

### SafariServices (macOS, Safari App Extensions)
- `SFSafariExtensionHandler` — base class for the extension's principal object
- `SFSafariExtensionHandler.contentBlocker(withIdentifier:blockedResourcesWith:on:)` **[NEW]** — content blocker notification
- `SFSafariExtensionHandler.page(_:willNavigateTo:)` **[NEW]** — page navigation callback
- `SFSafariExtensionHandler.messageReceivedFromContainingApp(withName:userInfo:)` — receive message from app
- `SFSafariExtensionHandler.popoverWillShow(in:)` — popover about to appear
- `SFSafariApplication.getAllWindows(completionHandler:)` **[NEW]**
- `SFSafariApplication.dispatchMessage(withName:toExtensionWithIdentifier:userInfo:completionHandler:)` — app to extension messaging
- `SFSafariWindow` — represents a window
- `SFSafariWindow.getAllTabs(completionHandler:)` **[NEW]**
- `SFSafariWindow.activeTab(completionHandler:)` — get active tab
- `SFSafariWindow.toolbarItem(completionHandler:)` — get toolbar item for this window
- `SFSafariTab` — represents a tab
- `SFSafariTab.navigate(to:)` **[NEW]**
- `SFSafariTab.getContainingWindow(completionHandler:)` **[NEW]**
- `SFSafariTab.activePage(completionHandler:)` — get active page
- `SFSafariTab.activate(completionHandler:)` — make this tab the active tab
- `SFSafariPage` — represents a web page
- `SFSafariPage.getScreenshot(completionHandler:)` **[NEW]**
- `SFSafariPage.getContainingTab(completionHandler:)` **[NEW]**
- `SFSafariPage.getPropertiesWithCompletionHandler(_:)` — get URL, title, etc.
- `SFSafariToolbarItem.showPopover()` **[NEW]**
- `SFSafariExtensionViewController.dismissPopover()` **[NEW]**

### Info.plist Keys
- `SFSafariAssociatedContentBlockers` (array of strings) **[NEW]** — in `NSExtension` section of Safari App Extension plist; lists bundle IDs of associated content blockers

### App Groups / XPC
- `UserDefaults(suiteName:)` — shared data between app and extension in the same App Group
- `NSXPCConnection` — IPC between extension, XPC service, and containing app within an App Group

## Code Highlights

Associating a Content Blocker and handling block notifications:
```xml
<!-- Safari App Extension Info.plist -->
<key>NSExtension</key>
<dict>
    ...
    <key>SFSafariAssociatedContentBlockers</key>
    <array>
        <string>com.example.app.contentblocker</string>
    </array>
</dict>
```

```swift
// In SFSafariExtensionHandler subclass
override func contentBlocker(withIdentifier contentBlockerIdentifier: String,
                              blockedResourcesWith urls: [URL],
                              on page: SFSafariPage) {
    var resources = blockedResourcesMap[page] ?? []
    resources.append(contentsOf: urls)
    blockedResourcesMap[page] = resources
    updateBadge(for: page)
}

override func page(_ page: SFSafariPage, willNavigateTo url: URL?) {
    blockedResourcesMap[page] = nil
}
```

Updating toolbar badge and showing popover:
```swift
func updateBadge(for page: SFSafariPage) {
    page.getContainingTab { tab in
        tab?.getContainingWindow { window in
            window?.toolbarItem { item in
                let count = self.blockedResourcesMap[page]?.count ?? 0
                item?.setBadgeText(count > 0 ? "\(count)" : nil)
            }
        }
    }
}
```

Popover pre-population before showing:
```swift
override func popoverWillShow(in window: SFSafariWindow) {
    window.getAllTabs { tabs in
        // Collect screenshots, titles, and blocked-resource counts
        // Pass state to popover view controller
        self.popoverViewController?.populateWith(tabs: tabs, blockedResources: self.blockedResourcesMap)
    }
}
```

Tab activation on popover row selection:
```swift
func tableViewSelectionDidChange(_ notification: Notification) {
    let row = tableView.selectedRow
    selectedTab?.activate { self.dismissPopover() }
}
```

## Takeaways
- Content Blocker notifications are trivially enabled by adding two `Info.plist` entries and one delegate method — a compelling way to build privacy dashboards without compromising the pure content-blocker mode.
- The full window/tab/page graph is now navigable from native extension code, enabling rich UI like tab-overview popovers with screenshots and statistics.
- Programmatic popover show/dismiss makes extension UI feel native rather than rigidly toolbar-button-driven.
- App Groups plus `SFSafariApplication.dispatchMessage` provide the complete bidirectional communication toolkit; the extension never needs to assume the containing app is running.

---
_Source: WWDC19 Session 720 page (abstract, chapter summaries, code samples, and resource links)._
