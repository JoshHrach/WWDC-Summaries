# Explore Multiview Video Playback in visionOS
**WWDC24 · Session 10116** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10116/)

_Platforms:_ visionOS 2

## Overview
visionOS introduces multiview — the ability to display up to five simultaneous video screens in an immersive space, built entirely on top of the existing `AVPlayerViewController` / `AVExperienceController` stack. The feature is ideal for sports broadcasts (multiple camera angles or multiple live games at once) and entertainment scenarios where a person wants side-by-side streams. A familiar single-screen experience transitions seamlessly into multiview by tapping a new multiview button in the player UI.

The session explains the `AVExperienceController` experience lifecycle (embedded → expanded → multiview), the `AVMultiViewManager` architecture for coordinating multiple screens, and the app-supplied content browser pattern for discovering and adding streams. Design guidance covers how to build an intuitive content browser and how to use progressive disclosure so viewers are never overwhelmed.

## Key Topics

### Introducing Multiview
The multiview button appears in the top-left corner of `AVPlayerViewController`'s UI when `.multiView` is added to `allowedExperiences`. Tapping it reveals a full-width content browser supplied by the app. Selecting additional content places it alongside the primary video. Screens cast light effects on their surroundings, layouts auto-adjust up to five slots, and drag-to-rearrange lets viewers customize which video gets center focus.

### Experience Controller and Experiences
`AVExperienceController` (a new counterpart to `AVPlayerViewController`) defines the set of allowed playback experiences for a given player: `.embedded`, `.expanded`, and the new `.multiView`. Calling `transition(to:)` moves the player between experiences asynchronously. Main-focus semantics (louder audio, more prominent visual) follow whichever video the viewer taps.

### AVMultiViewManager
`AVMultiViewManager.default` is the hub that tracks all experience controllers in multiview, manages the video screen layout, and surfaces the content browser at the right moments. Apps attach a custom `contentSelectionViewController` early in the lifecycle so the manager can display the browser whenever needed.

### Content Browser Design
The content browser is a `UIViewController` subclass that manages available content and vends the selection UI. Best practices: use accurate-aspect thumbnails and concise titles, highlight currently-playing items, match accent colors to the rest of the app, and avoid aggressive reordering that disrupts navigation. The browser area is spatially limited relative to the video screens, so minimalism is key.

### Delegate and Lifecycle Events
`AVExperienceController`'s delegate receives transition events however they originate (user tapping multiview button, closing a screen, or exiting multiview from playback controls). A delegate method allows the app to perform async work before a transition completes, enabling smooth state preparation.

## APIs & Frameworks

**AVKit**
- `AVPlayerViewController` — existing; now surfaces a multiview button when `.multiView` is allowed **[NEW behavior]**
- `AVExperienceController` **[NEW]** — new counterpart to `AVPlayerViewController`
  - `allowedExperiences` — specifies which experiences are available
  - `.multiView` experience value **[NEW]**
  - `transition(to:)` async method **[NEW]**
  - `AVExperienceController.Delegate` **[NEW]** — receives transition progress events; can perform async work before transition begins
- `AVMultiViewManager` **[NEW]**
  - `AVMultiViewManager.default` singleton
  - `contentSelectionViewController` property — attach a custom `UIViewController` for the content browser **[NEW]**

**AVFoundation**
- `AVFoundation` — underlying playback engine; interfaces with RealityKit on visionOS for high-quality rendering and spatial audio
- `AVQueuePlayer` — mentioned as the appropriate alternative for sequential content (e.g., TV episode playlists)

**RealityKit**
- Used internally by AVKit on visionOS for performant video rendering and light-casting effects

## Code Highlights

Supply the content browser early:
```swift
import AVKit

AVMultiViewManager
    .default
    .contentSelectionViewController = multiViewController()
```

Add content to multiview when a user selects it:
```swift
import AVKit

let controller = AVPlayerViewController()

let experienceController = controller.experienceController
experienceController.allowedExperiences = .recommended(including: [.multiView])

await experienceController.transition(to: .multiView)
```

Remove content from multiview:
```swift
import AVKit

let experienceController = …

await experienceController.transition(to: .embedded)
```

## Takeaways
- Add `.multiView` to `allowedExperiences` to unlock the multiview button; the system handles screen layout, chrome, and light-casting effects automatically.
- Attach a custom `contentSelectionViewController` to `AVMultiViewManager.default` early in the app lifecycle so the browser is always ready.
- Use `AVExperienceController.Delegate` to react to all transition events, including those triggered by the viewer directly (tapping close, exiting multiview).
- Design the content browser with restraint: accurate thumbnails, concise titles, and clear highlighting of currently-playing items — the limited spatial area rewards minimalism.

---
_Source: WWDC24 Session 10116 page (abstract, chapter summaries, code samples, and resource links)._
