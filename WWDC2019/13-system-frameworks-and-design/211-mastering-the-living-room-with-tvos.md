# Mastering the Living Room With tvOS
**WWDC19 · Session 211** · [Watch](https://developer.apple.com/videos/play/wwdc2019/211/)

_Platforms:_ tvOS 13

## Overview
This session covers three major areas of the tvOS 13 developer story: the redesigned Top Shelf with the new Carousel style, multiuser support via `TVUserManager`, and a full-screen browsing layout in TVUIKit. The underlying design philosophy throughout is "content first" — getting users into content with minimum friction, maximum immersion, and reduced cognitive load.

The new Top Shelf Carousel replaces static inset tiles with full-screen images and trailer videos that play on the Apple TV home screen before the user even opens the app. Two Carousel styles are available — Actions (minimal UI) and Details (rich metadata including genre, summary, media options, named attributes). A companion full-screen collection view layout (`TVCollectionViewFullScreenLayout`) brings the same immersive edge-to-edge browsing inside apps.

tvOS 13 also introduces multiuser support at the OS level via Control Center user switching. The `TVUserManager` API lets profile-based apps map OS user identifiers to in-app profiles, enabling seamless profile selection on app launch without redundant UI.

## Key Topics
- **Top Shelf Carousel** — new immersive full-screen extension style with autoplay trailer video, swipe-up to enter full-screen, and on-demand metadata reveal; 5–10 items recommended **[NEW tvOS 13]**
  - Actions Carousel style — title, context title, Play and More Info buttons
  - Details Carousel style — adds summary, genre, duration, media options, named key-value attributes
  - Sectioned and Inset Content styles still supported for non-video content
- **TVTopShelfExtension** — new extension target type replacing old API **[NEW tvOS 13]**
- **Multiuser / TVUserManager** — maps OS user identifiers to app profiles; system-provided confirmation UI and preference panel **[NEW tvOS 13]**
- **New Tab Bar style** — translucent tab bar that scrolls with content; `tabBarObservedScrollView`; `leadingAccessoryView` / `trailingAccessoryView`; `standardAppearance` **[NEW tvOS 13]**
- **TVCollectionViewFullScreenLayout** — edge-to-edge collection view layout with masking, parallax, and wipe transition effects **[NEW tvOS 13]**
- **TVCollectionViewFullScreenCell** — `maskedBackgroundView`, `maskedContentView`, `maskedAmount`, `contentBleed`, `parallaxOffset` **[NEW tvOS 13]**
- **Design best practices** — remove barriers to entry, reduce cognitive load, use familiar navigation cues, limit animations, use safe areas, 1920×1080 images, HLS video 2–5 minutes

## APIs & Frameworks
- **TVServices framework**
  - `TVTopShelfContentProvider` — root extension object; `loadTopShelfContent(completionHandler:)` **[NEW]**
  - `TVTopShelfContent` protocol **[NEW]**
  - `TVTopShelfCarouselContent` — Carousel content object; `.actions` or `.details` style **[NEW]**
  - `TVTopShelfSectionedContent` — Sectioned style content **[NEW]**
  - `TVTopShelfInsetContent` — Inset style content **[NEW]**
  - `TVTopShelfCarouselItem` — item for Carousel with `contextTitle`, `previewVideoURL`, `summary`, `genre`, `duration`, `mediaOptions`, `namedAttributes` **[NEW]**
  - `TVTopShelfItemCollection` — groups items in Carousel **[NEW]**
  - `TVTopShelfSectionedItem` — item for Sectioned style **[NEW]**
  - `TVTopShelfContentItem` — item for Inset style **[NEW]**
  - `TVTopShelfItem.displayAction` — action when item is selected while focused
  - `TVTopShelfItem.playAction` — action when Play button pressed on focused item
  - `TVUserManager` **[NEW]**
    - `currentUserIdentifier` — identifier for the active OS user
    - `currentUserIdentifierDidChangeNotification` — notification when user switches
    - `userIdentifiersForCurrentProfile` — set of identifiers mapped to current profile
    - `shouldStorePreference(forCurrentUser:)` — presents system confirmation UI
    - `presentProfilePreferencePanel(currentProfiles:availableProfiles:completion:)` — preference editing UI
- **TVUIKit framework**
  - `TVCollectionViewFullScreenLayout` — new collection view layout **[NEW]**
    - `maskedInset` — controls inset amount when browsing
    - `parallaxFactor` — relative motion speed of background vs content layers
  - `TVCollectionViewFullScreenCell` — cell subclass **[NEW]**
    - `maskedBackgroundView: UIView` — opaque background layer
    - `maskedContentView: UIView` — UI overlay layer
    - `maskedAmount: CGFloat` — 0 = full screen, 1 = inset
    - `contentBleed: UIEdgeInsets` — from layout attributes
    - `parallaxOffset: UIOffset` — from layout attributes
  - `TVBrowserViewController` — TVMLKit equivalent accessing full-screen layout
- **UIKit (tvOS)**
  - `UITabBarController.tabBarObservedScrollView` — content-scroll-tracking tab bar **[NEW]**
  - `UITabBar.leadingAccessoryView: UIView` **[NEW]**
  - `UITabBar.trailingAccessoryView: UIView` **[NEW]**
  - `UITabBar.standardAppearance: UITabBarAppearance` **[NEW]**
  - `UICollectionViewFlowLayout` → replace with `TVCollectionViewFullScreenLayout`
  - `UICollectionViewLayoutAttributes` subclassed by full-screen layout

## Code Highlights

```swift
// Top Shelf Extension — Carousel adoption
class ContentProvider: TVTopShelfContentProvider {
    override func loadTopShelfContent(completionHandler: @escaping (TVTopShelfContent?) -> Void) {
        MoviesClient.fetchFeatured { result in
            switch result {
            case .success(let response):
                completionHandler(response.makeTopShelfContent())
            case .failure:
                completionHandler(nil)
            }
        }
    }
}

extension MoviesResponse {
    func makeTopShelfContent() -> TVTopShelfCarouselContent {
        let items = movies.map { $0.makeCarouselItem() }
        let collection = TVTopShelfItemCollection(items: items)
        return TVTopShelfCarouselContent(collections: [collection], style: .details)
    }
}

extension Movie {
    func makeCarouselItem() -> TVTopShelfCarouselItem {
        let item = TVTopShelfCarouselItem(identifier: id)
        item.title = title
        item.contextTitle = "Featured Movies"
        item.previewVideoURL = trailerURL
        item.summary = synopsis
        item.setImageURL(thumbnailURL, for: .screenScale1x)
        item.setImageURL(thumbnail2xURL, for: .screenScale2x)
        return item
    }
}
```

```swift
// TVUserManager — profile mapping
let userManager = TVUserManager()
let currentID = userManager.currentUserIdentifier

if let profile = UserDefaults.standard.profile(for: currentID) {
    // Launch directly to content — no profile picker needed
    showContent(for: profile)
} else {
    showProfilePicker { selectedProfile in
        userManager.shouldStorePreference(forCurrentUser: selectedProfile) { store in
            if store { UserDefaults.standard.save(profile: selectedProfile, for: currentID) }
        }
    }
}
```

```swift
// Full-screen collection view layout
let layout = TVCollectionViewFullScreenLayout()
collectionView.collectionViewLayout = layout
collectionView.register(MyFullScreenCell.self, forCellWithReuseIdentifier: "cell")

// In cell: override applyLayoutAttributes for parallax/mask
override func apply(_ layoutAttributes: UICollectionViewLayoutAttributes) {
    super.apply(layoutAttributes)
    if let attrs = layoutAttributes as? TVCollectionViewFullScreenLayoutAttributes {
        titleLabel.alpha = attrs.maskedAmount
    }
}
```

## Takeaways
- The new Carousel Top Shelf style turns the home screen into an immersive content preview with autoplay video — adopt it by updating your Top Shelf Extension to use `TVTopShelfCarouselContent`.
- `TVUserManager` eliminates redundant profile pickers by mapping tvOS OS user identifiers to app profiles; the system handles user switching via Control Center.
- `TVCollectionViewFullScreenLayout` enables edge-to-edge browsing with built-in masking and parallax effects in just a few lines of code.
- The new Tab Bar in tvOS 13 scrolls with content automatically; use `tabBarObservedScrollView` and safe area layout guides to integrate it correctly.

---
_Source: WWDC19 Session 211 page (abstract, chapter summaries, code samples, and resource links)._
