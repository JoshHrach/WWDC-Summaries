# Deliver a Great Playback Experience on tvOS
**WWDC21 · Session 10191** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10191/)

_Platforms:_ tvOS 15

## Overview
tvOS 15 introduces a completely redesigned `AVPlayerViewController` playback UI for Apple TV. The new interface adds a transport bar with discoverable standard and custom controls, a Title View above the transport bar, content tabs for supplementary info and recommendations, and contextual actions (such as Skip Intro / Recap) that appear inline during playback. Apps must link against the tvOS 15 SDK to receive the new UI; apps built against tvOS 14 or earlier continue to use the older interface.

The session covers every new API: `transportBarCustomMenuItems`, `customInfoViewControllers`, `contextualActions`, `transportBarIncludesTitleView`, and the new TVUIKit content configurations `TVMediaItemContentConfiguration` and `TVMonogramContentConfiguration`.

## Key Topics

### Redesigned Playback UI (tvOS 15)
The new UI keeps viewers in the moment with:
- **Transport bar controls**: a row of tappable controls above the scrubber for subtitles, audio language, Picture in Picture, and custom app actions.
- **Title View**: displays the content title and subtitle above the transport bar, with a live badge for live streaming.
- **Content tabs**: tabs below the transport bar for Info (from metadata), Chapters (from navigation marker groups), and custom app-provided tabs.
- **Contextual actions**: overlaid buttons (e.g., Skip Intro, Recap) that appear at specific points during playback.

### Title View
`AVPlayerViewController` automatically populates the Title View from `AVPlayerItem.externalMetadata` (or embedded asset metadata) using `AVMetadataIdentifier.commonIdentifierTitle` and `AVMetadataIdentifier.iTunesMetadataTrackSubtitle`. For live content, the Title View shows a live badge automatically. To suppress the Title View, set `transportBarIncludesTitleView = false`.

### Transport Bar Custom Menu Items
`AVPlayerViewController.transportBarCustomMenuItems: [UIMenuElement]` accepts instances of `UIAction` and `UIMenu`. `UIMenu`s support one level of nesting using the `.displayInline` option. Use these to surface app-specific controls (e.g., Favorites, playback speed, quality selection) alongside the system-provided controls.

### Content Tabs
`AVPlayerViewController` shows:
- **Info tab**: automatically, when `AVPlayerItem.externalMetadata` or embedded metadata is present.
- **Chapters tab**: automatically, when `AVPlayerItem.navigationMarkerGroups` is populated.
- **Custom tabs**: via `customInfoViewControllers: [UIViewController]` **[NEW tvOS 15]** — replaces the deprecated `customInfoViewController` (singular). Set `preferredContentSize.height` or autolayout constraints to control tab height; the system sizes all tabs to the height of the tallest. Set `title` on each view controller — it becomes the tab label.

### TVUIKit Content Configurations
`TVMediaItemContentConfiguration` and `TVMonogramContentConfiguration` (both in `TVUIKit`) are new `UIContentConfiguration` implementations for collection view cells within content tabs.
- `TVMediaItemContentConfiguration.wideCell()` — 16:9 media cell; supports `image`, `text`, `secondaryText`, `badgeText`, `badgeProperties.backgroundColor`, `playbackProgress`
- `TVMonogramContentConfiguration.cell()` — circular/monogram cell for people; supports `image`, `text`

Assign via `cell.contentConfiguration = contentConfiguration`.

### Contextual Actions
`AVPlayerViewController.contextualActions: [UIAction]` **[NEW tvOS 15]** — an array of `UIAction` values displayed as inline controls during playback (typically at relevant content boundaries). Show by setting the property with actions; hide by setting it to an empty array `[]`. Observe `AVPlayer` timing with `addPeriodicTimeObserver` or `addBoundaryTimeObserver` to show/hide at the right moment. Using contextual actions preserves the app's Now Playing status and integrates with proper focus behavior.

## APIs & Frameworks

**AVKit** (`import AVKit`) — **[NEW tvOS 15]**

- `AVPlayerViewController` — standard playback view controller
  - `transportBarCustomMenuItems: [UIMenuElement]` **[NEW]** — array of `UIAction`/`UIMenu` added to transport bar
  - `customInfoViewControllers: [UIViewController]` **[NEW]** — multiple custom content tabs (replaces deprecated `customInfoViewController`)
  - `contextualActions: [UIAction]` **[NEW]** — inline playback action buttons
  - `transportBarIncludesTitleView: Bool` **[NEW]** — show/hide the Title View above transport bar (default `true`)
- `AVPlayerItem.externalMetadata: [AVMetadataItem]` — supply metadata not embedded in the asset
  - `AVMetadataIdentifier.commonIdentifierTitle` — title for Title View
  - `AVMetadataIdentifier.iTunesMetadataTrackSubtitle` — subtitle for Title View

**TVUIKit** (`import TVUIKit`) — **[NEW tvOS 15]**

- `TVMediaItemContentConfiguration` **[NEW]** — `UIContentConfiguration` for media cells
  - `TVMediaItemContentConfiguration.wideCell()` — 16:9 aspect ratio factory
  - `image: UIImage?`
  - `text: String?` — primary title
  - `secondaryText: String?` — secondary label
  - `badgeText: String?` — overlay badge label
  - `badgeProperties.backgroundColor: UIColor`
  - `playbackProgress: Float` — 0.0–1.0 progress overlay
- `TVMonogramContentConfiguration` **[NEW]** — `UIContentConfiguration` for monogram/avatar cells
  - `TVMonogramContentConfiguration.cell()` — factory
  - `image: UIImage?`
  - `text: String?` — display name

**AVFoundation** (timing observers for contextual actions)
- `AVPlayer.addPeriodicTimeObserver(forInterval:queue:using:)` — fire closure at regular intervals
- `AVPlayer.addBoundaryTimeObserver(forTimes:queue:using:)` — fire at specific timestamps

## Code Highlights

Transport bar custom controls:
```swift
let favoriteAction = UIAction(title: "Favorites", image: UIImage(systemName: "heart")) { _ in
    // Add to favorites
}

let speedSubmenu = UIMenu(title: "Speed", options: [.displayInline, .singleSelection], children: [
    UIAction(title: "0.5x") { _ in player.rate = 0.5 },
    UIAction(title: "1x")   { _ in player.rate = 1.0 },
    UIAction(title: "1.5x") { _ in player.rate = 1.5 }
])
let settingsMenu = UIMenu(image: UIImage(systemName: "gearshape"), children: [speedSubmenu])

playerViewController.transportBarCustomMenuItems = [favoriteAction, settingsMenu]
```

Custom content tab:
```swift
let recommendedVC = RecommendedContentViewController()
recommendedVC.title = "Recommended"
recommendedVC.preferredContentSize = CGSize(width: 0, height: 140)
playerViewController.customInfoViewControllers = [recommendedVC]
```

TVMediaItemContentConfiguration for wide media cell:
```swift
import TVUIKit

var config = TVMediaItemContentConfiguration.wideCell()
config.image = UIImage(named: "thumbnail")
config.text = "Episode Title"
config.secondaryText = "Season 2, Episode 5"
config.badgeText = "NEW"
config.badgeProperties.backgroundColor = .systemRed
config.playbackProgress = 0.75
cell.contentConfiguration = config
```

Contextual Skip Intro action with timing observer:
```swift
playerViewController.player = player

// Show Skip Intro at 90s, hide at 120s
let skipAction = UIAction(title: "Skip Intro") { _ in
    player.seek(to: CMTime(seconds: 120, preferredTimescale: 600))
}

player.addBoundaryTimeObserver(
    forTimes: [NSValue(time: CMTime(seconds: 90, preferredTimescale: 600))],
    queue: .main) {
    playerViewController.contextualActions = [skipAction]
}

player.addBoundaryTimeObserver(
    forTimes: [NSValue(time: CMTime(seconds: 120, preferredTimescale: 600))],
    queue: .main) {
    playerViewController.contextualActions = []
}
```

## Takeaways
- The new tvOS 15 playback UI is only available when the app links against the tvOS 15 SDK; set the deployment target to opt in.
- Populate `AVPlayerItem.externalMetadata` with at minimum title and artwork — this feeds both the Title View and Info tab automatically.
- `transportBarCustomMenuItems` replaces any need for gesture recognizer hacks; `UIMenu` with `.displayInline` + `.singleSelection` is the right pattern for mutually exclusive choices (e.g., playback speed).
- `customInfoViewControllers` (plural) replaces the deprecated `customInfoViewController`; set `preferredContentSize.height` or autolayout to control tab height.
- Contextual actions (`contextualActions`) integrate with focus and preserve Now Playing; use `addBoundaryTimeObserver` to show/hide at precise timestamps rather than building custom overlay UI.

---
_Source: WWDC21 Session 10191 page (abstract, chapter summaries, code samples, and resource links)._
