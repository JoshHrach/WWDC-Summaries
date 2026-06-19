# Build a UIKit App with the New Design
**WWDC25 · Session 284** · [Watch](https://developer.apple.com/videos/play/wwdc2025/284/)

_Platforms:_ iOS 26, iPadOS 26

## Overview
This session provides a comprehensive guide to adopting iOS 26's new Liquid Glass design system in UIKit apps. Recompiling with the iOS 26 SDK automatically applies the new glass aesthetic to standard controls, but the session goes further — covering tab view and split view updates, navigation bar and toolbar behavior changes, new presentation transitions, a revamped search placement system, updated controls, and the APIs for applying Liquid Glass to custom views.

The session is structured as a domain-by-domain audit of UIKit, showing exactly which APIs changed, which require developer action, and how to use new APIs like `UIGlassEffect`, `UIGlassContainerEffect`, `UIBackgroundExtensionView`, and `UIScrollEdgeElementContainerInteraction`.

## Key Topics

### Tab Views and Split Views
Tab bars float above content on iPhone with Liquid Glass. `.tabBarMinimizeBehavior = .onScrollDown` hides the tab bar on scroll. `UITabAccessory` creates a bottom accessory view (e.g., mini-player) above the tab bar; it animates inline when the tab bar minimizes. `UIBackgroundExtensionView` extends a content image seamlessly behind the sidebar on iPad, with `automaticallyPlacesContentView` for fine-grained control.

### Navigation and Toolbars
Navigation bars and toolbars are transparent by default — remove any `UIBarAppearance` background customization. Bar button items automatically group with shared glass backgrounds; use a `fixedSpace(0)` item to break groups. `UIBarButtonItem.style = .prominent` tints the button background. `hidesSharedBackground = false` on `flexibleSpace` items groups all items under one background. `UINavigationItem` gains `subtitle` and `largeSubtitleView` properties. `UIScrollEdgeElementContainerInteraction` applies edge effects to custom floating button containers. Hard edge style: `scrollView.topEdgeEffect.style = .hard`.

### Presentations
Menus and popovers automatically morph from glass source buttons. Sheets adopt zoom transition via `preferredTransition = .zoom { _ in barButtonItem }`. Action sheets on iPhone now anchor to their `sourceItem` or `sourceView` (just like iPad) and no longer show a cancel button when anchored inline.

### Search
`navigationItem.searchBarPlacementBarButtonItem` places search in the toolbar. `navigationItem.searchBarPlacementAllowsExternalIntegration = true` moves it to the navigation bar trailing edge (iPad). `UITabBarController` now supports a dedicated search tab. `tab.automaticallyActivatesSearch = true` activates search field on tab tap. `navigationItem.preferredSearchBarPlacement = .integratedCentered` centers search in the navigation area.

### Controls
`UIButtonConfiguration.glass()` and `.prominentGlass()` apply Liquid Glass styles. Sliders support `UISlider.TrackConfiguration` with `allowsTickValuesOnly`, `neutralValue`, and `numberOfTicks`. `slider.sliderStyle = .thumbless` renders a progress-bar style slider. Control sizes are updated — verify layouts accommodate new default sizes.

### Custom Liquid Glass
`UIGlassEffect` is applied to `UIVisualEffectView` inside an animation block (materialize animation). Shape is customized via `cornerConfiguration` (`.fixed(8)`, `.containerRelative()`). `tintColor` on `UIGlassEffect` applies vibrant tinting. `isInteractive = true` adds touch scale/bounce feedback. Animate out by setting `effectView.effect = nil` (dematerialize animation). `UIGlassContainerEffect` groups glass views so they merge/split like water droplets; `spacing` controls the merge threshold.

## APIs & Frameworks

**UIKit (iOS 26, iPadOS 26)**
- **[NEW]** `UITabBarController.tabBarMinimizeBehavior` — `.onScrollDown`
- **[NEW]** `UITabAccessory(contentView:)` — mini-player / bottom accessory
- **[NEW]** `UITabBarController.bottomAccessory`
- **[NEW]** `UITraitTabAccessoryEnvironment` — trait for detecting inline tab accessory
- **[NEW]** `UIBackgroundExtensionView` — extend content image behind sidebar
- **[NEW]** `UIBackgroundExtensionView.automaticallyPlacesContentView`
- **[NEW]** `UINavigationItem.subtitle` — subtitle below navigation title
- **[NEW]** `UINavigationItem.largeSubtitleView` — custom view below large title
- **[NEW]** `UIBarButtonItem.style = .prominent` — tinted glass background
- **[NEW]** `UIBarButtonItem.hidesSharedBackground` — group control on flexibleSpace
- **[NEW]** `UIScrollEdgeElementContainerInteraction` — apply edge effect to custom containers
- **[NEW]** `UIScrollView.topEdgeEffect.style = .hard` — hard edge style
- **[NEW]** `UIGlassEffect` — Liquid Glass material for `UIVisualEffectView`
- **[NEW]** `UIGlassEffect.tintColor` — vibrant tint
- **[NEW]** `UIGlassEffect.isInteractive` — touch scale/bounce
- **[NEW]** `UIVisualEffectView.cornerConfiguration` — `.fixed(_:)`, `.containerRelative()`
- **[NEW]** `UIGlassContainerEffect` — merge/split glass views; `spacing` property
- **[NEW]** `UIButtonConfiguration.glass()` — standard glass button style
- **[NEW]** `UIButtonConfiguration.prominentGlass()` — tinted glass button style
- **[NEW]** `UISlider.TrackConfiguration` — `allowsTickValuesOnly`, `neutralValue`, `numberOfTicks`
- **[NEW]** `UISlider.sliderStyle = .thumbless` — progress bar slider
- **[NEW]** `UIViewController.preferredTransition = .zoom { _ in sourceBarButtonItem }` — zoom sheet transition
- **[NEW]** `navigationItem.searchBarPlacementBarButtonItem` — search placement in toolbar
- **[NEW]** `navigationItem.searchBarPlacementAllowsExternalIntegration` — nav bar trailing search
- **[NEW]** `navigationItem.preferredSearchBarPlacement = .integratedCentered`
- **[NEW]** `UITab.automaticallyActivatesSearch`

## Code Highlights
Minimize tab bar and add accessory:
```swift
tabBarController.tabBarMinimizeBehavior = .onScrollDown
let accessory = UITabAccessory(contentView: NowPlayingView())
tabBarController.bottomAccessory = accessory
```

Custom Liquid Glass view:
```swift
let effectView = UIVisualEffectView()
addSubview(effectView)
let glassEffect = UIGlassEffect()
glassEffect.isInteractive = true
UIView.animate { effectView.effect = glassEffect }
```

Merge glass views with a container:
```swift
let container = UIGlassContainerEffect()
container.spacing = 20
let containerView = UIVisualEffectView(effect: container)
containerView.contentView.addSubview(view1)
containerView.contentView.addSubview(view2)
// Animate merging:
UIView.animate { view1.frame = view2.frame }
```

## Takeaways
- Remove all `UIBarAppearance` background customization — bars are transparent by default; custom backgrounds break the glass look.
- Use `UIGlassEffect` + `UIVisualEffectView` for floating controls (maps-style buttons); keep glass to the most important interactive layer.
- Pair `UIGlassContainerEffect` with animated layout changes to get droplet-merge / split animations automatically.
- Set `sourceItem`/`sourceView` on all action sheets regardless of device — behavior is now unified between iPhone and iPad.

---
_Source: WWDC25 Session 284 page (abstract, chapter summaries, code samples, and resource links)._
