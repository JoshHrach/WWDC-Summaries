# Add Shared with You to Your App
**WWDC22 · Session 10094** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10094/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13

## Overview
Shared with You surfaces links and content shared in Messages directly within third-party apps for the first time in iOS 16. Previously limited to first-party apps (Safari, News, Music, Podcasts, TV, Photos), the feature is now open to developers via the new `SharedWithYou` framework.

The system uses Universal Links to route shared content to the correct app, and presents a ranked list of highlights based on Siri Suggestions signals, pinning status, and recency. Users maintain granular control over sharing permission on a per-conversation, per-app, and global basis — pinning a link acts as implicit permission to surface it in Shared with You.

Key UI elements are: a "Shared with You shelf" in the app's browsing experience (showing ranked, attributed content previews), and an `SWAttributionView` on content detail screens that shows sender avatars and allows in-app reply to Messages conversations.

## Key Topics

### Shared with You Shelf
- A dedicated browsing section surfacing content shared in Messages, ordered by: Siri Suggestions (top), pinned items, then chronological recency
- Each shelf item shows a rich preview (thumbnail, title, subtitle) plus an `SWAttributionView`
- Include a "Show More" element to expand or navigate to full Shared with You content

### SWHighlightCenter
- Single object for accessing Shared with You content for the app
- `highlights` property returns an array of `SWHighlight` objects
- `SWHighlightCenterDelegate` protocol notifies the app when highlights are added, removed, or updated

### SWHighlight
- Model wrapping a URL to the shared content
- Use the `url` property to generate rich previews and navigate within the app

### SWAttributionView
- Out-of-process view that securely renders sender names, avatars, and pinning indicators
- Interactive: tapping navigates to the corresponding Messages conversation
- Context menu includes "Reply" and "Remove" (customizable title via `menuTitleForHideAction`)
- `displayContext` property provides Siri Suggestions ranking feedback (`.summary` vs `.detail`)

### Privacy and Security
- Attribution views drawn out of process; apps never have access to Messages recipients or conversations
- Content protected by Universal Links association — only the designated app can receive its highlights

## APIs & Frameworks

**SharedWithYou** (new framework) **[NEW]**
- `SWHighlightCenter` **[NEW]**
  - `.highlights: [SWHighlight]` — ordered list of shared content for this app
  - `.delegate: SWHighlightCenterDelegate?`
- `SWHighlightCenterDelegate` protocol **[NEW]**
  - `highlightCenterHighlightsDidChange(_:)` — called on content updates
- `SWHighlight` **[NEW]**
  - `.url: URL` — URL of the shared content
- `SWAttributionView` **[NEW]** (UIView subclass on iOS/iPadOS; NSView on macOS)
  - `.highlight: SWHighlight?` — triggers out-of-process attribution rendering
  - `.preferredMaxLayoutWidth: CGFloat` — maximum width for the view
  - `.horizontalAlignment: SWAttributionView.HorizontalAlignment` — `.leading`, `.center`, `.trailing`
  - `.displayContext: SWAttributionView.DisplayContext` — `.summary` (browsing) or `.detail` (active consumption)
  - `.backgroundStyle: SWAttributionView.BackgroundStyle` — `.color` or `.material`
  - `.menuTitleForHideAction: String` — custom title for the "Remove" context menu item
  - `.supplementalMenu: UIMenu` — context menu items to append to app's existing context menus
  - `.minimumContentSizeCategory` / `.maximumContentSizeCategory` (inherited from UIView)

**Entitlements / Capabilities**
- "Shared with You" capability (new Xcode capability) **[NEW]**
- `Associated Domains` entitlement (required for Universal Links)

**Universal Links (prerequisite)**
- `NSUserActivityTypes` / `application(_:continue:restorationHandler:)` — handle incoming universal links
- Associated Domains entitlement + `apple-app-site-association` JSON on web server

## Code Highlights

Enumerating highlights:
```swift
class SharedWithYouViewController: UIViewController, SWHighlightCenterDelegate {
    let highlightCenter = SWHighlightCenter()

    override func viewDidLoad() {
        super.viewDidLoad()
        highlightCenter.delegate = self
    }

    func highlightCenterHighlightsDidChange(_ highlightCenter: SWHighlightCenter) {
        for highlight in highlightCenter.highlights {
            let url = highlight.url
            // Generate a rich preview for each highlight
        }
    }
}
```

Showing the attribution view with context:
```swift
let attributionView = SWAttributionView()
attributionView.highlight = highlightCenter.highlights[index]
attributionView.preferredMaxLayoutWidth = maxWidth
attributionView.horizontalAlignment = .center
attributionView.displayContext = .detail  // user actively consuming content
attributionView.backgroundStyle = .material
```

Adding the supplemental context menu:
```swift
attributionView.menuTitleForHideAction = "Remove Item"
let contextMenuConfig = UIContextMenuConfiguration(identifier: nil, previewProvider: nil) { _ in
    let additionalMenu = attributionView.supplementalMenu
    // Append additionalMenu items to your content's existing menu
}
```

## Takeaways
- Adopt Universal Links first — Shared with You is built on top of the same deep-linking infrastructure.
- Add the new "Shared with You" Xcode capability, then use `SWHighlightCenter` + `SWAttributionView` to surface and attribute content in three straightforward steps.
- Set `displayContext = .detail` when the user is actively consuming content to give Siri Suggestions accurate ranking feedback.
- Attribution views are rendered out of process, ensuring user privacy — apps see only URLs, never Messages participants or conversation data.

---
_Source: WWDC22 Session 10094 page (abstract, chapter summaries, code samples, and resource links)._
