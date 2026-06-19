# Design for the iPadOS Pointer
**WWDC20 · Session 10640** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10640/)

_Platforms:_ iPadOS 14

## Overview
This session from Apple's design team (Brandon Walkin, Marcos Alonzo, CC Wan, Dylan Edwards) explains the design principles behind the new iPadOS pointer introduced in iPadOS 13.4 and fully elaborated in iPadOS 14. It covers the core concept of Adaptive Precision, the mechanics behind pointer behavior (snapping, recentering, inertia, magnetism), the three standard pointer effects (Highlight, Lift, Hover), how to design custom pointer shapes, and how to design gesture interactions specific to trackpad input.

This is the design companion to "Build for the iPadOS pointer" (10093), which covers the API implementation.

## Key Topics

**Why a Pointer on iPad**
Two primary motivations: ergonomics (hands stay on the keyboard plane rather than repeatedly reaching to the touch screen) and text editing precision (the finger is a coarse input; the pointer enables character-level text selection without drag handles).

**Adaptive Precision**
The central design concept. The iPadOS pointer dynamically adjusts its precision to match the precision required by the UI element being targeted. Standard mouse pointers move at pixel-level precision across the entire screen — a mismatch when the underlying UI only cares about button-level or line-level precision. Adaptive Precision closes this gap:
- When hovering over interactive controls, the pointer snaps to the control and morphs to its shape
- For text, the pointer morphs to an I-beam with vertical precision matching line height — impossible to land between lines
- For custom views (e.g., a day planner with 15-minute time slots), developers can implement snapping to expose only the precision the view supports

**Pointer Mechanics**
- **Model pointer**: an invisible second pointer tracking the true physical position; the visible pointer is a display of the model's interpretation
- **Snapping**: the visible pointer morphs and animates to the nearest interactive element once the model pointer enters its hit area
- **Recentering**: after lifting from the trackpad, the pointer centers on the control to reduce click errors on next contact
- **Auto-hide**: the pointer disappears after a short delay to encourage fluid switching between touch, Pencil, and pointer
- **Inertia**: a flick gesture gives the pointer momentum so large distances can be covered in one motion
- **Magnetism**: on finger lift, the system projects where the pointer would land with inertia, scans nearby controls in the direction of travel, and snaps to the closest one — users can flick to distant targets without overshooting

**The Three Pointer Effects**
1. **Highlight** — for small controls without a background (toolbar buttons, tab bar items). The pointer moves behind the control's icon/label and adopts a lighter background tint. On click, contents scale down to center. Use for button-sized controls adjacent to other Highlight controls.
2. **Lift** — for medium controls with their own background (app icons, Control Center modules). The pointer disappears behind the element; the element scales up with a shadow and a "radiosity" color bleed effect. On click, the element scales with a parallax specular highlight showing true pointer position. Use when you want to convey elevation. Requires providing correct size and corner radius; shadow may clip adjacent layers.
3. **Hover** — for larger elements that would behave poorly if the pointer morphed into their shape (large cards, events in Calendar). The pointer retains its circle shape; the element changes appearance (scale, shadow, tint, or combination). Highly customizable. Can also expose a custom pointer shape with snapping.

**Using the Automatic Effect**
Always try `.automatic` first — the system selects the best effect based on object type, location, size, and shape. The rules may change across OS updates, so `.automatic` ensures future-compatibility. Only override when the automatic choice is contextually wrong (e.g., a bookmark icon in a toolbar next to Highlight buttons should use Highlight even if Lift might be automatic).

**Hit Region Design**
Hit regions should be sized for comfort: add ~12 pt padding around elements with a bezel, ~24 pt around elements without. Leave no gaps between adjacent controls' hit regions — a gap causes the pointer to revert to circular shape between elements, creating an unwanted animation. Extended hit regions also improve touch usability.

**Custom Pointer Shapes**
Two APIs: `UIPointerShape.path(_:)` for arbitrary shapes; `UIPointerShape.roundedRect(_:radius:)` for rectangle morphs. Design guidelines:
- Keep shapes simple and instantly understandable
- Use solid fills (color-adaptive material constantly changes; strokes become illegible)
- Size close to the default 19 pt circle; large differences create jarring transitions
- Match anchor point to intent; prefer center anchoring for symmetric shapes
- Match the shape's anchor points to the circle's for smooth morphing transitions
- Apply standard pointer behaviors to custom pointers (snapping, effects)

**Pointer Interactions and Gestures**
- Use pointer presence (hover state) to proactively show/hide UI (e.g., Books shows toolbars on hover without requiring a click)
- Use pointer inactivity as a trigger to hide UI in full-screen media experiences
- Pointer enables new "drag from rest" interactions (drag-and-drop, drag-select) that previously required long-press disambiguation
- Two-finger trackpad gestures are available for app-custom interactions; gestures should act relative to the view under the pointer
- Secondary click (two-finger click or Control+click) provides accelerated access to context menus — apps get this for free for views with `UIContextMenuInteraction`
- Zoom and rotate gestures anchor to the pointer position for natural feel
- Avoid adding features that only work with the pointer; instead use pointer precision to make existing features easier

## APIs & Frameworks

### UIKit — Pointer Interactions
- `UIPointerInteraction` — attach to any view to add pointer behavior **[NEW]**
- `UIPointerInteractionDelegate` — provides `UIPointerStyle` for each region
- `UIPointerStyle` — combines a `UIPointerEffect` and an optional `UIPointerShape`
- `UIPointerEffect` — `.automatic`, `.highlight(UITargetedPreview)`, `.lift(UITargetedPreview)`, `.hover(UITargetedPreview, prefersShadow:prefersScaledContent:)` **[NEW]**
- `UIPointerShape` — `.path(UIBezierPath)` for arbitrary shapes; `.roundedRect(CGRect, radius:)` for rect morphs **[NEW]**
- `UIPointerRegion` — defines a hit region within a view that triggers a specific pointer style **[NEW]**
- `UIPointerRegion.rect` — the CGRect defining the active region
- `UIPointerStyle.hidden()` — explicitly hides the pointer over a region

### UIKit — Gesture Recognizers
- `UIHoverGestureRecognizer` — fires on pointer hover without click; use for proactive UI reveal **[NEW]**
- `UIContextMenuInteraction` — provides secondary-click context menu automatically **[NEW in iOS 13, trackpad integration automatic in iOS 14]**
- Two-finger trackpad drag: use `UIPanGestureRecognizer` with `allowedScrollTypesMask = .continuous`

### Human Interface Guidelines
- [HIG: Pointing Devices](https://developer.apple.com/design/human-interface-guidelines/pointing-devices) — canonical reference for effect selection, hit region sizing, custom pointer guidelines

## Code Highlights

No specific code samples were shown in this design session. For implementation, see "Build for the iPadOS pointer" (10093), which covers `UIPointerInteraction`, `UIPointerStyle`, and `UIHoverGestureRecognizer` in detail.

Conceptual pointer interaction attachment:
```swift
let interaction = UIPointerInteraction(delegate: self)
button.addInteraction(interaction)

// Delegate provides style per region
func pointerInteraction(_ interaction: UIPointerInteraction,
                        styleFor region: UIPointerRegion) -> UIPointerStyle? {
    let preview = UITargetedPreview(view: button)
    return UIPointerStyle(effect: .automatic(preview))
}
```

## Takeaways
- Adaptive Precision — the pointer adjusting its behavior to match the precision required by the UI — is the core design concept; every pointer effect and behavior follows from it. Design hit regions and effects to reinforce this, not to fight it.
- Always start with `.automatic` for `UIPointerEffect`; only override it when context requires consistency with surrounding controls (e.g., using Highlight instead of Lift to match adjacent toolbar buttons).
- Hit regions should have no gaps between adjacent controls; a gap causes a visual reversion to the circle pointer that creates an unwanted animation and poor UX.
- Use `UIHoverGestureRecognizer` to reveal hidden UI proactively on hover — this turns the pointer's position into a signal of intent and removes the need for taps to show/hide toolbars or controls.

---
_Source: WWDC20 Session 10640 page (abstract and full transcript)._
