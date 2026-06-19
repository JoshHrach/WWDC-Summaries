# Deliver an Exceptional Accessibility Experience
**WWDC18 · Session 230** · [Watch](https://developer.apple.com/videos/play/wwdc2018/230/)

_Platforms:_ iOS, macOS

## Overview
This session goes beyond basic VoiceOver label compliance to show what an _exceptional_ accessibility experience looks like. Using a sample app called "Exceptional Dogs" — a dog adoption browser with a carousel, stats view, gallery modal, and shelter info — the presenters first audit the app to surface subtle but significant issues: inaccessible action buttons, out-of-order label reads, missing context on call/map actions, and VoiceOver focus leaking behind a modal view. They then fix each issue using targeted UIAccessibility APIs and live-demonstrate the improved VoiceOver experience.

The session also covers visual-design accessibility across four concern areas: transparency/blur, contrast, sizing/dynamic type, motion, and cognitive UI complexity. Each area maps directly to a system setting that apps can and should query to adapt their interfaces.

## Key Topics

### Visual Design Accessibility

- **Reduce Transparency** — check `UIAccessibility.isReduceTransparencyEnabled` (iOS) or `NSWorkspace.shared.accessibilityDisplayShouldReduceTransparency` (macOS); swap blurred backgrounds for solid ones when enabled.
- **Contrast** — minimum recommended ratio is 4.5:1 (WCAG); use Xcode's Accessibility Inspector to measure color pair ratios. Check `UIAccessibility.isDarkerSystemColorsEnabled` (previously "Darken Colors") to provide higher-contrast tints; standard UIKit tinted controls get this for free. macOS: `NSWorkspace.shared.accessibilityDisplayShouldIncreaseContrast`.
- **Sizing & Dynamic Type** — 7 standard content sizes + 5 accessibility sizes. Check `UIApplication.shared.preferredContentSizeCategory` to adapt layouts. `UIAccessibility.isBoldTextEnabled` signals when users want heavier stroke weights; standard system fonts handle this automatically.
- **Reduce Motion** — problematic animation categories: scaling/zoom, spinning/vortex, plane-shifting, multidirectional/multispeed scroll springiness, and peripheral/background animations. Check `UIAccessibility.isReduceMotionEnabled` (iOS) or `NSWorkspace.shared.accessibilityDisplayShouldReduceMotion` (macOS). Do not just remove animation — replace it with an equally engaging but non-triggering alternative (e.g., cross-fades).
- **UI Complexity** — clear cause-and-effect, consistent behavior, minimal navigation barriers. Familiar UIKit controls serve as a design language; custom controls should mirror parallel UIKit controls to be intuitive.

### Custom Accessibility Elements (`UIAccessibilityElement`)

- Use `UIAccessibilityElement` to create virtual elements that represent inaccessible or custom-drawn content, or to reshape the navigation experience.
- Override `accessibilityElements` on a container view to declare the ordered list of accessible sub-elements.
- Set `accessibilityFrame` on each element to the corresponding on-screen rect (converted to screen coordinates).
- Used in the demo to group a carousel, expose per-dog favorite/gallery buttons at the right position in the swipe order, and pair label+value cells in the stats view into single elements with full context.

### Adjustable Trait (Carousel Interaction)

- Add `.adjustable` to `accessibilityTraits` on a custom element to signal that VoiceOver users can swipe up/down to increment/decrement.
- Override `accessibilityIncrement()` and `accessibilityDecrement()` to drive the underlying carousel scroll.
- Post `UIAccessibility.layoutChangedNotification` after each scroll so VoiceOver re-evaluates on-screen elements and announces the new dog.

### Grouping and Context

- Combine related label+value pairs into a single `UIAccessibilityElement` with a combined `accessibilityLabel` (e.g., "Breed: Labrador") to eliminate out-of-order reads and reduce swipe count.
- Group shelter name, open-in-maps, and call into a single accessible element whose label is the shelter name; expose the two actions as `accessibilityCustomActions`.

### Custom Actions (`UIAccessibilityCustomAction`)

- Return an array of `UIAccessibilityCustomAction` from `accessibilityCustomActions` on any view.
- Each action has a `name` (announced to the user) and a `target`/`selector` that executes the action.
- VoiceOver users cycle through actions with a swipe; Switch Control users see them as quick actions.

### Modal Views (`accessibilityViewIsModal`)

- When displaying a non-modal overlay view (added directly to the hierarchy rather than presented as a `UIViewController`), VoiceOver still sees background elements.
- Override `accessibilityViewIsModal` → `true` on the overlay to make assistive technologies ignore all sibling views.
- Post `UIAccessibility.screenChangedNotification` when the overlay appears so VoiceOver moves focus into it.

## APIs & Frameworks

**UIKit / UIAccessibility**
- `UIAccessibility.isReduceTransparencyEnabled` — check reduce transparency setting
- `UIAccessibility.isDarkerSystemColorsEnabled` — check increased contrast setting
- `UIAccessibility.isBoldTextEnabled` — check bold text setting
- `UIAccessibility.isReduceMotionEnabled` — check reduce motion setting
- `UIAccessibilityElement` — virtual element class; `accessibilityContainer`, `accessibilityLabel`, `accessibilityValue`, `accessibilityFrame`, `accessibilityTraits`
- `UIAccessibilityTraits.adjustable` **[NEW usage pattern]** — enable increment/decrement callbacks on custom elements
- `accessibilityIncrement()` / `accessibilityDecrement()` — override to respond to swipe-up/down gestures
- `accessibilityElements` — override on container view to define ordered accessible sub-element list
- `accessibilityCustomActions` — return `[UIAccessibilityCustomAction]`; each has `name` + `target`/`selector`
- `UIAccessibilityCustomAction` — `init(name:target:selector:)`
- `accessibilityViewIsModal` — override on overlay view to confine VoiceOver focus to that view
- `UIAccessibility.post(notification:argument:)` — post `layoutChangedNotification` or `screenChangedNotification`
- `UIAccessibility.Notification.layoutChanged` — signals element order has changed (VoiceOver updates, focus stays near current)
- `UIAccessibility.Notification.screenChanged` — signals major screen change (VoiceOver moves focus to argument element or top)

**AppKit (macOS equivalents)**
- `NSWorkspace.shared.accessibilityDisplayShouldReduceTransparency`
- `NSWorkspace.shared.accessibilityDisplayShouldIncreaseContrast`
- `NSWorkspace.shared.accessibilityDisplayShouldReduceMotion`

## Code Highlights

Subclassing `UIAccessibilityElement` with the adjustable trait for a carousel:
```swift
class CarouselAccessibilityElement: UIAccessibilityElement {
    var currentDog: Dog?

    override var accessibilityLabel: String? { get { "Dog picker" } set {} }
    override var accessibilityValue: String? {
        get { "\(currentDog?.name ?? ""), \(currentDog?.isFavorited == true ? "favorited" : "not favorited")" }
        set {}
    }
    override var accessibilityTraits: UIAccessibilityTraits {
        get { .adjustable } set {}
    }
    override func accessibilityIncrement() { container?.scrollCollectionView(forward: true) }
    override func accessibilityDecrement() { container?.scrollCollectionView(forward: false) }
}
```

Posting a layout-changed notification after carousel scroll:
```swift
func scrollViewDidScroll(_ scrollView: UIScrollView) {
    // ... update current dog ...
    UIAccessibility.post(notification: .layoutChanged, argument: nil)
}
```

Making a modal overlay confine VoiceOver focus, and posting screen-changed:
```swift
// In DogModalView:
override var accessibilityViewIsModal: Bool { get { true } set {} }

// In the view controller, when gallery button is pressed:
UIView.animate(withDuration: 0.3) { self.modalView.alpha = 1 }
UIAccessibility.post(notification: .screenChanged, argument: modalView)
```

## Takeaways
- Accessibility compliance (all elements reachable) is the floor, not the ceiling — the goal is an experience that is also fast, contextual, and cognitive-load-efficient for assistive technology users.
- `UIAccessibilityElement` is the primary tool for reshaping navigation order, combining related content, and enabling custom gestures (adjustable trait) on collection-view or drawing-based UIs.
- Always pair `accessibilityViewIsModal = true` with a `screenChangedNotification` post when showing overlay views outside of `UIViewController` presentation.
- Visual-design accessibility (contrast, motion, transparency, text size) has direct API hooks; querying these settings and adapting the UI benefits users with low vision, vestibular disorders, and cognitive disabilities without requiring them to use VoiceOver.

---
_Source: WWDC18 Session 230 page (abstract, full transcript, and resource links)._
