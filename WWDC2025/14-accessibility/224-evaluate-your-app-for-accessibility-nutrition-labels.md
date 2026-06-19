# Evaluate Your App for Accessibility Nutrition Labels
**WWDC25 · Session 224** · [Watch](https://developer.apple.com/videos/play/wwdc2025/224/)

_Platforms:_ iOS, iPadOS, macOS, tvOS, watchOS, visionOS (App Store — all platforms)

## Overview
Accessibility Nutrition Labels are a new App Store feature that lets developers declare which accessibility features their app supports, helping users — especially those who depend on assistive technologies — discover apps that work for them. This session walks through the complete evaluation framework: defining common tasks, testing each task against every supported accessibility feature, and then indicating support in App Store Connect.

The session covers seven features: Sufficient Contrast, Dark Interface, Larger Text, Differentiate Without Color Alone, Reduced Motion, Voice Control, VoiceOver, Captions, and Audio Descriptions.

## Key Topics

### Accessibility Nutrition Labels Overview
- Appear on an app's App Store product page, visible before download.
- Each label represents a specific accessibility capability the developer attests the app supports.
- Accuracy is critical: Apple provides evaluation criteria documentation to standardize what "support" means for each feature.
- Added in App Store Connect under the app's product page. An optional link to a developer accessibility website can also be included.

### Evaluation Methodology
1. **Define common tasks** — primary functionalities (what users download the app for) plus fundamental tasks (first launch, login, purchase, settings).
2. **Test each feature against every common task** on every supported device type.
3. **Do not indicate support** if the feature is not relevant to the app's functionality.

### Features to Evaluate

**Sufficient Contrast**
- Foreground/background color pairs must meet minimum contrast ratios by default, or when "Increase Contrast" is enabled.
- Test in both light and dark appearances.

**Dark Interface**
- App must present a predominantly dark background in Dark Mode.
- Test with Smart Invert enabled — ensure media images are not inadvertently color-inverted.

**Larger Text**
- Text must scale to at least 200%; ideally support larger (e.g., the presenters note 310% as a real-world need).
- Use Dynamic Type; avoid hard-coded font sizes.
- All text should wrap rather than truncate; text fields should grow to accommodate larger sizes.

**Differentiate Without Color Alone**
- Every instance where color conveys information must also use a shape, icon, or text label.
- Common examples: status indicators, error states, selected states.

**Reduced Motion**
- Remove or substitute known motion triggers when "Reduce Motion" is enabled: zooming/sliding transitions, auto-playing animations, parallax, flashing/blinking.
- Consult HIG documentation for the specific list of known triggers.

**Voice Control**
- All interactive elements must be operable by voice command.
- Every tappable element must have an `accessibilityLabel` that matches what a user would naturally speak.
- Test by completing all common tasks using only voice commands and `show names` overlays — never touch the screen.

**VoiceOver**
- All interactive elements must be navigable and activatable by VoiceOver gestures (swipe right to move focus, double-tap to activate, three-finger swipe to scroll).
- Test on every supported device; use a real VoiceOver user or internal testers to surface issues not visible to sighted developers.
- VoiceOver requires TextKit2 for full feature support in text views.

**Captions**
- If the app has video or audio-only content, users must be able to enable captions.
- If the app has no audio or video content, do not claim Captions support.

**Audio Descriptions**
- If video content conveys information visually without audio narration, audio descriptions must be available.
- If the app has no such content, do not claim Audio Descriptions support.

## APIs & Frameworks

### SwiftUI / UIKit Accessibility
- `.accessibilityLabel(_:)` — **required** for all non-text interactive elements (images, icon-only buttons).
- `.accessibilityValue(_:)` — communicate current state (e.g., slider value).
- `.accessibilityHint(_:)` — optional; explains what action does.
- Dynamic Type: `Font.body`, `.headline`, etc. — scale automatically; prefer over custom font sizes.
- `UIAccessibility.isReduceMotionEnabled` — check to conditionally suppress animations.
- `UIAccessibility.isDarkerSystemColorsEnabled` / `isReduceTransparencyEnabled` — for contrast adaptations.
- `traitCollection.userInterfaceStyle == .dark` — detect dark mode.
- Smart Invert: mark media content with `accessibilityIgnoresInvertColors = true`.

### App Store Connect
- Accessibility Nutrition Labels section — available in product page metadata.
- Evaluation criteria documentation — linked in Resources.

## Code Highlights

```swift
// Add an accessibility label to an icon-only button
Button {
    shareContent()
} label: {
    Image(systemName: "square.arrow.up")
}
.accessibilityLabel("Share")
```

## Takeaways
- Define your app's common tasks before starting evaluation — every feature is tested against every common task, so having a clear list prevents gaps.
- Do not claim a label unless users can complete all common tasks with that feature enabled; partial support is not sufficient.
- Partner with real users who depend on these features during testing — developers who do not use assistive technologies daily miss issues that are immediately apparent to those who do.
- Start with VoiceOver and Voice Control: the API work (accessibility labels, roles, values) done for one substantially reduces the work for the other.

---
_Source: WWDC25 Session 224 page (abstract, chapters, full transcript, and code samples)._
