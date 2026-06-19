# Accessibility Inspector
**WWDC19 · Session 257** · [Watch](https://developer.apple.com/videos/play/wwdc2019/257/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15

## Overview
The Accessibility Inspector is an Xcode developer tool that enables developers to find, diagnose, and fix accessibility issues without needing to manually navigate an app with VoiceOver from scratch. This session walks through a live debugging workflow using a sample Peanut Butter Tracking app, demonstrating three of the tool's primary workflows: the Audit tab for automated issue detection, Point Inspection for element-level investigation, and the Color Contrast Calculator for legibility checking.

The session highlights a new macOS feature called Hover Text — a magnifier for low-vision users that enlarges content under the cursor — and shows how the Inspector's VoiceOver simulation (speaker button) lets developers hear exactly what VoiceOver would say for any element without running VoiceOver itself. Combining automated audits with targeted code fixes produces a significantly improved accessibility experience in a short feedback loop.

## Key Topics

### Accessibility Inspector Overview
- Launched from Xcode menu: Xcode > Open Developer Tools > Accessibility Inspector.
- Supports target selection via a drop-down: iOS Simulator, connected iOS/watchOS/tvOS device, or macOS process.
- Three tabs: **Inspector**, **Audit**, and **Settings**.

### Audit Tab
- **Run Audit** button scans the current view and produces a list of potential accessibility issues.
- Issues are color-highlighted in yellow on the connected device/simulator when selected.
- Common detected issues: image file name used as accessibility label, potentially inaccessible custom-drawn elements, insufficient color contrast.
- Each issue has a **Help button** that provides a human-readable suggestion (e.g., "Set a human-readable, localized accessibilityLabel").

### Inspector Tab and Point Inspection
- **Point Inspection button**: with a finger/cursor held on the connected device, hovering over any element focuses the Inspector on it and shows all accessibility properties (label, value, traits, frame, etc.).
- **Speaker button**: simulates what VoiceOver would speak for the focused element — useful without enabling VoiceOver system-wide.
- **Next/Previous buttons**: step through accessible elements in document order to audit the full navigation sequence.
- **Auto Navigate button**: automatically steps through all elements in sequence while reading them aloud (hands-free audit).

### Color Contrast Calculator
- Standalone tool: Window > Show Color Contrast Calculator.
- Pick foreground and background colors via color pickers; the tool calculates the contrast ratio.
- Recommended minimum ratio: **3.0** (WCAG guidelines for large text / UI components). Normal body text requires 4.5.
- Slider in the tool lets developers adjust a color while watching the ratio update in real time.

### Hover Text (macOS new feature)
- **[NEW]** Hover Text: magnifies any content under the mouse cursor to a larger, sharper overlay — useful for low-vision Mac users and for readable screen sharing/presentations.

### Fixing Common Issues
- **Image file names as labels**: set `accessibilityLabel` on `UIButton`/`UIView` to a localized, human-readable string instead of relying on asset names.
- **Custom-drawn text (CATextLayer, Core Graphics)**: set `isAccessibilityElement = true` and provide a meaningful `accessibilityLabel`.
- **Contrast**: use the Color Contrast Calculator to choose colors that meet the ≥ 3.0 ratio before shipping.

## APIs & Frameworks

**UIAccessibility (UIKit)**
- `UIView.accessibilityLabel: String?` — human-readable label for assistive technologies
- `UIView.isAccessibilityElement: Bool` — marks a custom view/layer as an accessible element
- `UIView.accessibilityTraits: UIAccessibilityTraits` — describes element type and state
- `UIView.accessibilityValue: String?` — current value description
- `UIView.accessibilityHint: String?` — additional usage hint

**CATextLayer** — custom text layer; must manually set `isAccessibilityElement` and `accessibilityLabel` since it does not inherit UIKit accessibility support.

**Accessibility Inspector (Xcode Tool)**
- Audit tab with Run Audit **[NEW improvements]**
- Point Inspection mode
- VoiceOver speech simulation (speaker button)
- Auto Navigate
- Color Contrast Calculator (Window > Show Color Contrast Calculator)
- Hover Text **[NEW]** — macOS magnification overlay for low-vision users

## Code Highlights

```swift
// UIKit: adding a meaningful accessibilityLabel to a custom UIButton
favoriteButton.accessibilityLabel = isFavorited ? "Remove from favorites" : "Add to favorites"
cameraButton.accessibilityLabel = NSLocalizedString("Take a photo", comment: "")
buyButton.accessibilityLabel = NSLocalizedString("Buy", comment: "")

// Making a custom CATextLayer accessible
brandNameLayer.isAccessibilityElement = true
brandNameLayer.accessibilityLabel = brandName

// Correcting contrast: switch from light gray to a darker color
// (validated via Accessibility Inspector Color Contrast Calculator — ratio ≥ 3.0)
brandNameLayer.foregroundColor = UIColor.darkGray.cgColor
```

## Takeaways
- The Accessibility Inspector's Audit tab is the fastest way to catch common issues across an entire view before releasing — run it as part of a regular testing checklist.
- Point Inspection + the speaker button let any developer hear exactly what a VoiceOver user hears, with no accessibility experience required.
- Custom-drawn content (Core Graphics, Core Animation layers) requires manual `isAccessibilityElement` and `accessibilityLabel` configuration — UIKit controls handle this automatically.
- The Color Contrast Calculator removes guesswork from color selection; aim for ≥ 3.0 contrast ratio for UI elements and ≥ 4.5 for body text.

---
_Source: WWDC19 Session 257 page (abstract, chapter summaries, code samples, and resource links)._
