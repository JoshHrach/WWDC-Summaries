# Build for the iPadOS Pointer
**WWDC20 · Session 10093** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10093/)

_Platforms:_ iPadOS 14

## Overview
iPadOS 13.4 introduced general pointing device support with a fluid, adaptive pointer that morphs into controls and changes shape based on context. This session covers how to adopt pointer customization APIs in apps using a top-down strategy: start with high-level `UIButton` and `UIBarButtonItem` built-in effects, then layer in `UIPointerInteraction` for custom views.

The session builds on a quilting app demo to show two real-world scenarios: customizing the effect shape for a bar button (fixing incorrect region sizing), and adding a custom crosshair pointer shape with axis-constrained regions to a canvas view that assists users in making straight stitches. Region entrance/exit animation coordination via `UIPointerInteractionAnimating` is also demonstrated.

A key design philosophy from the session: iPadOS pointer support should feel like a natural extension of touch, not a port of macOS cursor behavior. The pointer morphs into controls (locking accuracy) and changes shape to provide contextual hints.

## Key Topics

**Automatic Pointer Support**
Many UIKit controls get pointer behavior automatically in iPadOS 13.4+: `UIBarButtonItem`, `UISegmentedControl`, `UIMenuController`, scroll views (two-finger pan, wheel scroll, pinch-to-zoom), collection/table swipe actions (two-finger pan), `UITextInteraction`, `UIDragInteraction` (click-drag instead of long-press), and `UIContextMenuInteraction` (secondary click for compact menu).

**UIButton Pointer Support**
`UIButton.isPointerInteractionEnabled = true` enables automatic effect. `UIButton.pointerStyleProvider` closure customizes the proposed effect and shape. The closure receives a `proposedEffect: UIPointerEffect` and `proposedShape: UIPointerShape` and returns a `UIPointerStyle?`.

**UIPointerStyle Categories**
1. Content effects: `UIPointerEffect` (`.highlight`, `.lift`, `.hover`) + `UIPointerShape` — pointer morphs into the view and applies visual treatment.
2. Shape customizations: `UIPointerShape` + `constrainedAxes: UIAxis` — pointer changes shape and is constrained along specified axes (used for text beam, crosshair guides).

**UIPointerInteraction for Custom Views**
Attach a `UIPointerInteraction` to any view. The delegate is optional. Without a delegate, the entire view gets an automatic effect. Implement `regionFor(request:defaultRegion:)` to return custom `UIPointerRegion` objects for subregions. Implement `styleFor(_:)` to return a `UIPointerStyle` for each region.

**Region and Animation Coordination**
Contiguous regions enable seamless pointer transitions without reverting to the system cursor between UI elements. `willEnter(_:animator:)` and `willExit(_:animator:)` delegate methods receive a `UIPointerInteractionAnimating` object for coordinating additional animations (e.g., fading separators) synchronized with the pointer effect.

## APIs & Frameworks

### UIKit **[NEW in iPadOS 13.4]**
- `UIButton.isPointerInteractionEnabled` **[NEW]** — enables built-in pointer interaction
- `UIButton.pointerStyleProvider` **[NEW]** — closure `(UIButton, UIPointerEffect, UIPointerShape) -> UIPointerStyle?`
- `UIPointerStyle(effect:shape:)` **[NEW]** — content effect style
- `UIPointerStyle(shape:constrainedAxes:)` **[NEW]** — shape customization style
- `UIPointerEffect` **[NEW]** — visual treatment applied to a view
  - `.highlight(_:)` — subtle highlight + parallax (for bar buttons)
  - `.lift(_:)` — lift + shadow (for intrinsic-shaped controls)
  - `.hover(_:prefersShadow:prefersScaledContent:)` — hover effect
- `UIPointerShape` **[NEW]** — pointer shape descriptor
  - `.roundedRect(_:radius:)` — rounded rectangle
  - `.path(_:)` — arbitrary Bezier path
  - `.verticalBeam(length:)` — vertical text insertion beam
  - `.horizontalBeam(length:)` — horizontal beam
- `UIPointerInteraction` **[NEW]** — attach to any `UIView` for custom pointer behavior
- `UIPointerInteractionDelegate` protocol **[NEW]**
  - `pointerInteraction(_:regionFor:defaultRegion:) -> UIPointerRegion` **[NEW]**
  - `pointerInteraction(_:styleFor:) -> UIPointerStyle?` **[NEW]**
  - `pointerInteraction(_:willEnter:animator:)` **[NEW]**
  - `pointerInteraction(_:willExit:animator:)` **[NEW]**
- `UIPointerRegion` **[NEW]** — defines the area where a pointer style is active
  - `UIPointerRegion(rect:identifier:)` **[NEW]**
- `UIPointerInteractionAnimating` **[NEW]** — animator for coordinated animations
  - `addAnimations(_:)` **[NEW]**
  - `addCompletion(_:)` **[NEW]**
- `UITargetedPreview` **[NEW in iOS 13]** — visual preview for effects; requires view + `UIPreviewParameters`
  - `UIPreviewParameters` — configure shadow path, background color, etc.
- `UIAxis` — `.horizontal`, `.vertical`, `.both` — axis constraint for shape customizations
- `UIHoverGestureRecognizer` **[NEW]** — responds directly to pointer movement without visual effects

## Code Highlights

Enabling and customizing a UIButton pointer effect:
```swift
myButton.isPointerInteractionEnabled = true
myButton.pointerStyleProvider = { button, proposedEffect, proposedShape in
    let rect = button.bounds.insetBy(dx: -8, dy: -4)
    return UIPointerStyle(effect: .highlight(proposedEffect.preview),
                          shape: .roundedRect(rect))
}
```

Custom crosshair shape for a canvas view:
```swift
let interaction = UIPointerInteraction(delegate: self)
canvasView.addInteraction(interaction)

func pointerInteraction(_ interaction: UIPointerInteraction,
                        styleFor region: UIPointerRegion) -> UIPointerStyle? {
    let path = crosshairPath()
    return UIPointerStyle(shape: .path(path))
}
```

Axis-constrained pointer for straight-line guides:
```swift
func pointerInteraction(_ interaction: UIPointerInteraction,
                        styleFor region: UIPointerRegion) -> UIPointerStyle? {
    if isStraightLineMode {
        let path = crosshairPath()
        return UIPointerStyle(shape: .path(path), constrainedAxes: .vertical)
    }
    return UIPointerStyle(shape: .path(crosshairPath()))
}
```

Coordinated animation on region enter/exit:
```swift
func pointerInteraction(_ interaction: UIPointerInteraction,
                        willEnter region: UIPointerRegion,
                        animator: UIPointerInteractionAnimating) {
    animator.addAnimations { self.separatorView.alpha = 0.0 }
}
func pointerInteraction(_ interaction: UIPointerInteraction,
                        willExit region: UIPointerRegion,
                        animator: UIPointerInteractionAnimating) {
    animator.addAnimations { self.separatorView.alpha = 1.0 }
}
```

## Takeaways
- Start pointer adoption top-down: use `UIBarButtonItem` and `UIButton.isPointerInteractionEnabled` for chrome first, then add `UIPointerInteraction` for custom views.
- `UIPointerStyle` has two modes: content effects (pointer morphs into the view with `.highlight`, `.lift`, or `.hover`) and shape customizations (pointer changes shape and optionally snaps to an axis with `constrainedAxes`).
- Provide contiguous `UIPointerRegion` objects to eliminate gaps where the pointer would revert to the system cursor between elements; the interaction can be placed on a parent view to manage regions for all children.
- Use `willEnter/willExit` delegate methods with `UIPointerInteractionAnimating` to coordinate UI changes (e.g., hiding separators) in sync with the pointer morph animation.

---
_Source: WWDC20 Session 10093 page (abstract, chapter summaries, code samples, and resource links)._
