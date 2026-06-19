# Meet WebKit for SwiftUI
**WWDC25 · Session 231** · [Watch](https://developer.apple.com/videos/play/wwdc2025/231/)

_Platforms:_ iOS 26, iPadOS 26, macOS Tahoe 26, visionOS 26

## Overview
WebKit gets a first-class SwiftUI integration in 2025 with two new types: `WebView` (a SwiftUI view) and `WebPage` (an `Observable` class). These replace the common pattern of wrapping `WKWebView` in `UIViewRepresentable`/`NSViewRepresentable`, eliminating hundreds of lines of boilerplate for the most common web embedding scenarios while maintaining full access to WebKit's capabilities.

`WebPage` is the model object: it holds navigation state, JavaScript evaluation, and scheme handling. `WebView` is the declarative rendering surface. The Observable conformance means SwiftUI views automatically update when page title, URL, loading state, or scroll position change — no manual KVO or delegate bridging needed.

The session covers navigation decisions, custom URL scheme handling, JavaScript interop, scroll control, and platform-specific behaviors like visionOS "look to scroll."

## Key Topics

### WebView (SwiftUI View)
The new `WebView` accepts a `WebPage` binding and renders it inline in any SwiftUI layout. It supports standard SwiftUI modifiers like `findNavigator` (for in-page search) and `scrollBounceBehavior`. On visionOS, `webViewScrollInputBehavior` enables "look to scroll" — scrolling by gazing at the view.

### WebPage (Observable)
`WebPage` is an `@Observable` class that acts as the model for a `WebView`. It exposes `url`, `title`, `isLoading`, `estimatedProgress`, and `currentNavigationEvent` as observable properties. Navigation is initiated by calling `webPage.load(URLRequest(...))` or by the user interacting with the view.

### Navigation Decisions
Conforming to `WebPage.NavigationDeciding` allows the app to intercept navigation events and decide whether to allow, cancel, or redirect them — equivalent to `WKNavigationDelegate` but expressed as a Swift protocol with async-compatible methods.

### Custom URL Scheme Handling
`URLSchemeHandler` protocol **[NEW]** and `URLScheme` let apps intercept navigations to custom URL schemes and return data from app code. This enables loading local resources, custom protocols, or server-side content without running an embedded HTTP server.

### JavaScript Interop
`webPage.callJavaScript(_:)` **[NEW]** evaluates a JavaScript expression and returns the result as a Swift value. The call is async and throws on script errors, eliminating the callback-based `evaluateJavaScript(_:completionHandler:)` pattern.

### Scroll Geometry
`webViewScrollPosition` modifier and `onScrollGeometryChange` modifier allow SwiftUI views outside the `WebView` to track and respond to the page's scroll position — useful for showing/hiding overlays tied to scroll depth.

## APIs & Frameworks

- **WebView** **[NEW]** — SwiftUI view that renders a `WebPage`
- **WebPage** **[NEW]** — `@Observable` model class for web content and navigation state
  - `url`, `title`, `isLoading`, `estimatedProgress` — observable properties
  - `currentNavigationEvent` **[NEW]** — current navigation lifecycle event
  - `load(_:)` — initiate navigation
- **WebPage.NavigationDeciding** **[NEW]** — protocol for intercepting navigation decisions
- **URLSchemeHandler** protocol **[NEW]** — handle custom URL schemes in app code
- **URLScheme** **[NEW]** — type-safe custom scheme registration
- **WebPage.Configuration** **[NEW]** — scheme handlers and other WebKit configuration
- **callJavaScript(_:)** **[NEW]** — async JavaScript evaluation returning Swift values
- **webViewScrollPosition** modifier **[NEW]** — bind scroll position to external state
- **onScrollGeometryChange** modifier **[NEW]** — observe scroll geometry changes
- **webViewScrollInputBehavior** **[NEW]** — visionOS look-to-scroll control
- **findNavigator** (existing SwiftUI modifier) — in-page text search, works with WebView
- **scrollBounceBehavior** (existing SwiftUI modifier) — works with WebView

## Code Highlights

```swift
// Basic WebView with Observable WebPage
@State private var webPage = WebPage()

var body: some View {
    WebView(webPage)
        .onAppear {
            webPage.load(URLRequest(url: URL(string: "https://example.com")!))
        }
    Text(webPage.title ?? "Loading…")
}
```

```swift
// Custom URL scheme handler
struct AssetSchemeHandler: URLSchemeHandler {
    func handle(_ task: URLSchemeTask) {
        let data = loadLocalAsset(for: task.request.url!)
        task.didReceive(URLResponse(url: task.request.url!, ...))
        task.didReceive(data)
        task.didFinish()
    }
}

let config = WebPage.Configuration()
config.register(AssetSchemeHandler(), for: URLScheme("myapp"))
let webPage = WebPage(configuration: config)
```

```swift
// Async JavaScript evaluation
let count = try await webPage.callJavaScript("document.querySelectorAll('p').length")
```

## Takeaways

- `WebPage` + `WebView` eliminate the `UIViewRepresentable` wrapper for most web embedding use cases — migrate existing `WKWebView` wrappers to the new API.
- `WebPage`'s `@Observable` conformance means loading state, title, and URL drive SwiftUI views reactively with no extra plumbing.
- `URLSchemeHandler` is the clean way to serve local or generated content into a web view — no embedded HTTP server required.
- `callJavaScript(_:)` is async and throws, making JavaScript interop first-class Swift concurrency rather than completion-handler callbacks.

---
_Source: WWDC25 Session 231 page (abstract, chapter summaries, code samples, and resource links)._
