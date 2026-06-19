# Create Accessible Spatial Experiences
**WWDC23 · Session 10034** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10034/)

_Platforms:_ visionOS 1

## Overview
visionOS is designed for everyone, and this session provides a comprehensive guide to making spatial computing apps accessible across four dimensions: vision, motor, cognitive, and hearing. The session emphasizes that visionOS ships with the largest set of accessibility features ever included in a first-generation Apple product, and that developers play a critical role in ensuring these features work well within their apps.

The first half focuses on vision accessibility: VoiceOver on visionOS uses new pinch gesture controls and Spatial Audio to navigate, and RealityKit entities must be made accessible using the new `AccessibilityComponent`. Head-anchored content, motion, and Dynamic Type considerations are specific to spatial computing and require new thinking from developers.

The second half covers motor accessibility (Dwell Control, Pointer Control, Switch Control with new camera position controls), cognitive accessibility (Guided Access, UI complexity, pacing), and hearing accessibility (quality captions with pop-on style, Media Accessibility framework for caption styling).

## Key Topics

### Vision: VoiceOver on visionOS
VoiceOver is reimagined for spatial computing:
- Right index finger pinch = next item; right middle = previous; right ring/left index = activate.
- Spatial Audio cues indicate where items are in 3D space.
- **Direct Gesture Mode** (new): activated with left index finger triple-pinch-and-hold; lets the app receive hand input directly instead of VoiceOver gestures.
- Apps should support both Default Interaction Mode (accessibility actions) and Direct Gesture Mode (spatial audio announcements).

### AccessibilityComponent (RealityKit — NEW)
`AccessibilityComponent` makes RealityKit entities accessible to VoiceOver, Voice Control, and Switch Control:
- `isAccessibilityElement` — marks entity as navigable.
- `traits` — `AccessibilityTraits` values (`.button`, `.playsSound`, etc.).
- `label` — `LocalizedStringResource` name.
- `value` — state description; update in `didSet` handlers via convenience properties.
- `systemActions` — enable system actions: `.activate`, `.adjustable`, etc.
- `customActions`, `customRotors`, `customContent` — advanced accessibility.
Subscribe to `AccessibilityEvents.Activate` (and other events) in `RealityView.content.subscribe(to:)`.

### Announcements
`AccessibilityNotification.Announcement(_:).post()` — post spoken announcements for meaningful events (context changes, object appearances, interactions). Essential for Direct Gesture Mode where visual feedback cannot be read by VoiceOver.

### Head-Anchored Content
Head anchors (content that follows head movement) create problems for users with low vision and Zoom. Prefer world anchors or use lazy repositioning. Observe:
- SwiftUI: `@Environment(\.accessibilityPrefersHeadAnchorAlternative)` **[NEW]**
- UIKit: `AXPrefersHeadAnchorAlternative()`, `NSNotification.Name.AXPrefersHeadAnchorAlternativeDidChange` **[NEW]**

### Reduce Motion
Avoid dizzying motion (rapid transitions, bouncing, spinning, persistent background animation). When Reduce Motion is on, use crossfades or static alternatives.
- SwiftUI: `@Environment(\.accessibilityReduceMotion)`
- UIKit: `UIAccessibility.isReduceMotionEnabled`, `.reduceMotionStatusDidChangeNotification`

### Motor: Dwell Control, Pointer Control, Switch Control
- **Dwell Control** — full pointer interaction without hands; supports tap, scroll, long press, drag. Design apps to support these gestures so nothing is hand-exclusive.
- **Pointer Control** — replaces eye tracking with head, wrist, or index finger as the focus driver. Reinforces why head-anchored content should be avoided/minimized.
- **Switch Control** — new camera position modifiers (new in visionOS) let users reposition their spatial view without physically moving; if experiences require specific positioning, provide bypass options.

### Cognitive: Guided Access and Clarity
- Guided Access restricts to single app, removes distractions. Use Custom Restrictions APIs for app-specific constraints.
- Prefer Apple UI frameworks (SwiftUI) for familiarity.
- Avoid complex gestures that are hard to learn and retain.
- Allow users to pace themselves; avoid time pressure.

### Hearing: Captions
- Prefer **pop-on captions** (full phrase at once) over roll-up (word-by-word) to reduce reading fatigue and motion sickness.
- Support system caption customization (text size, font, color, stroke, background) via AVKit/AVFoundation automatically, or manually via Media Accessibility APIs.
- Check `UIAccessibility.isClosedCaptioningEnabled` and observe `.closedCaptioningStatusDidChangeNotification` to default captions on.
- Captions should represent all audio including music, sound effects, and spatial audio directionality cues.

## APIs & Frameworks

**RealityKit**
- `AccessibilityComponent` **[NEW]** — accessibility properties for RealityKit entities
- `AccessibilityComponent.isAccessibilityElement` **[NEW]**
- `AccessibilityComponent.traits` (`.button`, `.playsSound`, `.adjustable`, etc.) **[NEW]**
- `AccessibilityComponent.label` — `LocalizedStringResource` **[NEW]**
- `AccessibilityComponent.value` — `LocalizedStringResource` **[NEW]**
- `AccessibilityComponent.systemActions` (`.activate`, `.adjustable`) **[NEW]**
- `AccessibilityComponent.customActions` **[NEW]**
- `AccessibilityComponent.customRotors` **[NEW]**
- `AccessibilityComponent.customContent` **[NEW]**
- `Entity.accessibilityValue` — convenience setter (updates `AccessibilityComponent`) **[NEW]**
- `AccessibilityEvents.Activate` **[NEW]** — event published when VoiceOver activate action fires
- `SceneContent.subscribe(to:AccessibilityEvents.Activate.self:)` **[NEW]**

**Accessibility (Framework)**
- `AccessibilityNotification.Announcement(_:)` **[NEW]** — post a spoken announcement
- `AccessibilityNotification.Announcement.post()` **[NEW]**
- `AXPrefersHeadAnchorAlternative()` **[NEW]** — UIKit function
- `NSNotification.Name.AXPrefersHeadAnchorAlternativeDidChange` **[NEW]**

**SwiftUI**
- `@Environment(\.accessibilityPrefersHeadAnchorAlternative)` **[NEW]** — detect head anchor preference
- `@Environment(\.accessibilityReduceMotion)` — reduce motion preference
- `View.accessibilityLabel(_:)`, `.accessibilityValue(_:)`, `.accessibilityTraits(_:)` — standard modifiers

**UIKit / UIAccessibility**
- `UIAccessibility.isClosedCaptioningEnabled`
- `UIAccessibility.closedCaptioningStatusDidChangeNotification`
- `UIAccessibility.isReduceMotionEnabled`
- `UIAccessibility.reduceMotionStatusDidChangeNotification`

**MediaAccessibility**
- Per-attribute caption style APIs — check system caption appearance settings and apply to custom caption implementations

## Code Highlights

Making a RealityKit entity accessible with activate action:
```swift
var accessibilityComponent = AccessibilityComponent()
accessibilityComponent.isAccessibilityElement = true
accessibilityComponent.traits = [.button, .playsSound]
accessibilityComponent.label = "Cloud"
accessibilityComponent.value = "Grumpy"
accessibilityComponent.systemActions = [.activate]
cloud.components[AccessibilityComponent.self] = accessibilityComponent

// Subscribe to activate events in RealityView
content.subscribe(to: AccessibilityEvents.Activate.self, componentType: nil) { activation in
    handleCloudCollision(for: activation.entity, gameModel: gameModel)
}
```

Posting a spatial announcement:
```swift
AccessibilityNotification.Announcement("8 clouds in front of you").post()
```

## Takeaways
- `AccessibilityComponent` is the foundational API for visionOS accessibility: without it, RealityKit entities are invisible to VoiceOver, Voice Control, and Switch Control.
- Support both VoiceOver Default Interaction Mode (activate actions) and Direct Gesture Mode (spatial audio announcements) to cover all VoiceOver usage patterns.
- Head-anchored content must be used sparingly and offer alternatives via `accessibilityPrefersHeadAnchorAlternative`; Zoom users and those using Pointer Control depend on this.
- Motor accessibility on visionOS is built around Dwell Control and Pointer Control—design all interactions to be achievable without eyes or specific hand movements.

---
_Source: WWDC23 Session 10034 page (abstract, chapter summaries, code samples, and resource links)._
