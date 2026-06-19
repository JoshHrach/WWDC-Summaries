# Embedding and Sharing Visually Rich Links
**WWDC19 · Session 262** · [Watch](https://developer.apple.com/videos/play/wwdc2019/262/)

_Platforms:_ iOS 13, macOS 10.15 Catalina

## Overview
The new Link Presentation framework allows developers to present URLs in a rich, visually appealing way consistent with the experience already available in Messages. Previously reserved for system apps, this framework lets any app fetch and display metadata—title, icon, image, or video—from any URL with minimal code.

The session follows the development of a recipe bookmarking app as it progressively adopts Link Presentation, going from a plain list of URLs to a rich grid of visually distinct cards. Metadata is fetched asynchronously from the web, cached locally for performance, and then displayed using the built-in link view component.

The new Share Sheet in iOS 13 also benefits from this framework: when an app provides pre-fetched metadata, the share sheet preview appears instantly rather than loading asynchronously after presentation.

## Key Topics

**Retrieving Metadata from a URL**
`LPMetadataProvider` fetches rich metadata (title, icon, image, video) from any URL. It handles network access, respects server responses, and returns an `LPLinkMetadata` object via a completion handler. Error handling is important since network failures, slow servers, or missing network connectivity can all cause failures. Local file URLs are also supported, using QuickLook thumbnailing for previews.

**Presenting Rich Links with LPLinkView**
`LPLinkView` takes an `LPLinkMetadata` object and renders the rich link UI automatically. It has an intrinsic size and responds to `sizeThatFits(_:)` to adapt to any layout. It can be embedded directly in table view cells or any other UIView-based hierarchy.

**Accelerating the iOS 13 Share Sheet**
Apps that already hold `LPLinkMetadata` for a URL can pass it to the share sheet by implementing `activityViewControllerLinkMetadata(_:)` on their `UIActivityItemSource`. This causes the share sheet header preview to appear instantly. When the user shares to Messages, the same metadata travels with the share for a seamless experience.

**Pre-populating Metadata Without Network Fetching**
Developers with existing databases of titles and images do not need to re-fetch from the network. They can create an `LPLinkMetadata` object manually, set `originalURL`, `url`, `title`, `imageProvider`, and other fields directly from local data.

## APIs & Frameworks

**LinkPresentation** **[NEW]**
- `LPMetadataProvider` **[NEW]** — fetches metadata from a remote or local URL
  - `func startFetchingMetadata(for url: URL, completionHandler: @escaping (LPLinkMetadata?, Error?) -> Void)` **[NEW]**
- `LPLinkMetadata` **[NEW]** — model object holding title, icon, image, video, originalURL, url
  - Conforms to `NSSecureCoding` for serialization and local caching **[NEW]**
  - `originalURL: URL?`
  - `url: URL?`
  - `title: String?`
  - `iconProvider: NSItemProvider?`
  - `imageProvider: NSItemProvider?`
  - `videoProvider: NSItemProvider?`
- `LPLinkView` **[NEW]** — UIView subclass that renders a rich link card
  - `init(metadata: LPLinkMetadata)` **[NEW]**
  - Responds to `sizeThatFits(_:)` for adaptive layout

**UIKit**
- `UIActivityItemSource` protocol — existing protocol extended with new method
  - `func activityViewControllerLinkMetadata(_:) -> LPLinkMetadata?` **[NEW]** — provides metadata for instant Share Sheet preview

**QuickLook** (used internally by `LPMetadataProvider` for local file thumbnailing)

## Code Highlights

Fetching metadata for a URL:
```swift
let metadataProvider = LPMetadataProvider()
metadataProvider.startFetchingMetadata(for: url) { metadata, error in
    guard error == nil else { return }
    // use metadata
}
```

Presenting a rich link view:
```swift
let linkView = LPLinkView(metadata: metadata)
cell.contentView.addSubview(linkView)
```

Supplying pre-fetched metadata to the Share Sheet:
```swift
func activityViewControllerLinkMetadata(_ activityViewController: UIActivityViewController) -> LPLinkMetadata? {
    return existingMetadata
}
```

Pre-populating metadata from a local database:
```swift
let metadata = LPLinkMetadata()
metadata.originalURL = url
metadata.url = url
metadata.title = recipe.title
metadata.imageProvider = NSItemProvider(object: recipe.image)
```

## Takeaways
- Use `LPMetadataProvider` to fetch and cache rich link metadata; do not re-fetch on every presentation.
- Use `LPLinkView` for a system-consistent, visually rich link presentation with no custom UI work.
- Implement `activityViewControllerLinkMetadata(_:)` to make the iOS 13 Share Sheet preview appear instantly.
- Pre-populate `LPLinkMetadata` manually from existing data to avoid unnecessary network requests.

---
_Source: WWDC19 Session 262 page (abstract, chapter summaries, code samples, and resource links)._
