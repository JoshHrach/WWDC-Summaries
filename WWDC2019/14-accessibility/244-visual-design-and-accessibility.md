# Visual Design and Accessibility
**WWDC19 · Session 244** · [Watch](https://developer.apple.com/videos/play/wwdc2019/244/)

_Platforms:_ iOS 13, iPadOS 13

## Overview
This session covers three visual accessibility improvements introduced in iOS 13: better Dynamic Type support practices, two new motion-reduction settings (Auto-Play Video Previews and Prefer Cross-Fade Transitions), and the Differentiate Without Color API brought to iOS from macOS. Together they help apps serve users with low vision, motion sensitivity, and color-vision deficiencies.

The session emphasizes that accessibility improvements often produce universally better designs — using shape-based indicators instead of color alone tends to look cleaner for all users, and enabling Dynamic Type forces layouts that scale gracefully across all screen sizes. Accessibility is framed not as a compliance checkbox but as inclusive design.

## Key Topics

**Dynamic Type**
Dynamic Type lets users choose their preferred text size system-wide. Apps should make all text dynamic (not just body text), use the full screen width, never truncate growing text, and scale glyphs that appear alongside text. iOS 13 still supports 11 text styles (Large Title down to Caption 2). Custom fonts can be scaled using `UIFontMetrics`. Xcode 11 adds a Dynamic Type slider in the Environment Overrides pane for instant simulator preview.

**Reduce Motion Improvements**
One in three people has some form of motion sensitivity. iOS already throttles parallax and weather animations when Reduce Motion is on. iOS 13 adds two new settings:
1. **Auto-Play Video Previews** (new setting + API): when disabled, videos in apps like the App Store no longer auto-play; user interaction is required. Default is enabled.
2. **Prefer Cross-Fade Transitions** (appears when Reduce Motion is on): UIKit automatically replaces lateral slide transitions with dissolves system-wide. Standard `UINavigationController` and modal presentations get this for free.

**Differentiate Without Color**
Ported from macOS to iOS 13. When enabled, apps must convey state with more than just color (e.g., add a shape, icon, or label). The API is a boolean property on `UIAccessibility` plus a notification for live updates. A peanut butter inventory example shows circles being replaced with check marks, question marks, and X symbols when the setting is on.

## APIs & Frameworks

**UIKit — Dynamic Type**
- `UIFont.preferredFont(forTextStyle:)` — returns a font that scales with content size category
- `UIFontMetrics(forTextStyle:)` — scales a custom `UIFont` to match a text style's proportional sizing
- `UIFontMetrics.scaledFont(for:)` — returns a scaled version of a custom font
- `UILabel.adjustsFontForContentSizeCategory` — auto-updates font when content size category changes
- `UITextField.adjustsFontForContentSizeCategory`
- `UITextView.adjustsFontForContentSizeCategory`
- `UIContentSizeCategory` — enum of all Dynamic Type size categories
- `UIApplication.shared.preferredContentSizeCategory` — current size category
- `UIContentSizeCategoryDidChangeNotification` — fires when user changes text size
- Xcode 11 Environment Overrides pane — Dynamic Type preview slider **[NEW]**

**UIKit — Motion**
- `UIAccessibility.isReduceMotionEnabled` — existing property; check before animating
- `UIAccessibility.reduceMotionStatusDidChangeNotification` — existing notification
- `UIAccessibility.isVideoAutoplayEnabled` **[NEW]** — `false` when Auto-Play Video Previews is disabled
- `UIAccessibility.videoAutoplayStatusDidChangeNotification` **[NEW]** — fires when setting changes
- `UIAccessibility.prefersCrossFadeTransitions` **[NEW]** — `true` when Prefer Cross-Fade Transitions is enabled (requires Reduce Motion on)
- UIKit standard navigation / modal transitions automatically use dissolve when `prefersCrossFadeTransitions` is `true` **[NEW]**

**UIKit — Color**
- `UIAccessibility.shouldDifferentiateWithoutColor` **[NEW]** — `true` when Differentiate Without Color is on
- `UIAccessibility.differentiateWithoutColorDidChangeNotification` **[NEW]** — fires when setting changes

## Code Highlights

Custom font scaled with UIFontMetrics:

```swift
func scaledFont(for style: UIFont.TextStyle) -> UIFont {
    let customFonts: [UIFont.TextStyle: UIFont] = [
        .body: UIFont(name: "MyFont-Regular", size: 17)!,
        .headline: UIFont(name: "MyFont-Bold", size: 17)!
        // ... map all 11 styles
    ]
    let base = customFonts[style] ?? UIFont.preferredFont(forTextStyle: style)
    return UIFontMetrics(forTextStyle: style).scaledFont(for: base)
}
```

Responding to Differentiate Without Color:

```swift
func updateIndicators() {
    if UIAccessibility.shouldDifferentiateWithoutColor {
        // Replace color circles with labeled symbols
        statusView.image = UIImage(systemName: isAvailable ? "checkmark.circle" : "xmark.circle")
    } else {
        statusView.image = nil
        statusView.backgroundColor = isAvailable ? .systemGreen : .systemRed
    }
}

NotificationCenter.default.addObserver(
    self,
    selector: #selector(updateIndicators),
    name: UIAccessibility.differentiateWithoutColorDidChangeNotification,
    object: nil
)
```

Checking Auto-Play Video Previews:

```swift
if UIAccessibility.isVideoAutoplayEnabled {
    player.play()
} else {
    // Show play button; wait for user interaction
}
```

## Takeaways
- Support all 11 Dynamic Type text styles, use `adjustsFontForContentSizeCategory`, and scale glyphs alongside text — this serves low-vision users and produces better adaptive layouts for everyone.
- Check `UIAccessibility.isVideoAutoplayEnabled` before auto-playing any video; observe the notification to respond to live changes.
- Never rely on color alone to convey meaning; check `shouldDifferentiateWithoutColor` and supplement with shapes, icons, or labels.
- Standard UIKit navigation automatically handles cross-fade transitions when `prefersCrossFadeTransitions` is `true` — no extra code needed for most apps.

---
_Source: WWDC19 Session 244 page (abstract, transcript, and resource links)._
