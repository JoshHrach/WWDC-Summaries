# Enhance Your iPad and iPhone Apps for the Shared Space
**WWDC23 · Session 10094** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10094/)

_Platforms:_ visionOS 1, iOS 17, iPadOS 17

## Overview
Most iPad and iPhone apps run without changes in the visionOS Shared Space, but this session covers targeted enhancements to make them feel truly at home. It focuses on three areas: interaction (hover effects and input handling), visual treatments (appearance and content scaling), and media (camera, microphone, and playback differences).

The session explains how visionOS natural input works — look-to-focus, indirect pinch taps, and direct touch — and why hover effects are now critical for communicating interactability. System controls handle hover effects automatically; custom controls built with `onTapGesture` or custom `ButtonStyle` need explicit adoption. It also details which camera and microphone hardware is and is not available to apps running in the Shared Space, and calls out removed features like `AVRoutePickerView` and Picture in Picture.

## Key Topics

- **Interaction** — Natural input model (look + pinch, direct touch, pointer, accessibility); automatic hover effects on system controls; adding `.hoverEffect()` to custom tap targets; customizing hover effect shape with `.contentShape(.hoverEffect, shape)`; opting out of hover effects on disabled/de-emphasized elements; game controller support for games.
- **Visuals** — iPad and iPhone apps display in light-mode appearance; dynamic content scaling renders images sharp at any angle; vector assets preferred; prompts do not present modally — apps must handle deferred callbacks.
- **Media** — Discovery sessions required to detect available cameras and microphones; `.back` camera returns a black frame (nonfunctional placeholder); `.front` camera returns a composite camera (returns no frames if no Spatial Persona detected); single `.front` location microphone available; `AVRoutePickerView` and Picture in Picture are unavailable; background audio suspended when device is locked/removed; `VNDocumentCameraViewController` automatically uses Continuity Camera.

## APIs & Frameworks

**SwiftUI**
- `.hoverEffect()` modifier **[NEW/enhanced for visionOS]** — adds system-managed hover highlight to interactive views
- `.contentShape(.hoverEffect, shape)` modifier **[NEW]** — customizes shape used for hover effect rendering; accepts any `Shape`
- `ButtonStyle` — custom styles disable default hover effects; must re-add `.hoverEffect()` explicitly
- `.onTapGesture` — does not receive hover effects automatically; wrap in `Button` or add `.hoverEffect()`

**AVFoundation**
- `AVCaptureDevice.DiscoverySession` — required to detect available capture hardware on visionOS
- `AVCaptureDevice.Position.back` — returns black/nonfunctional camera frame (placeholder for app compatibility)
- `AVCaptureDevice.Position.front` — composite spatial camera; returns no frames without Spatial Persona
- `AVCaptureDevice` (microphone) — single `.front` location microphone available
- `AVRoutePickerView` — **unavailable** on visionOS; check before showing
- Picture in Picture (`AVPictureInPictureController`) — **unavailable** on visionOS; check `isPictureInPictureSupported`

**VisionKit**
- `VNDocumentCameraViewController` — automatically uses Continuity Camera on nearby device when visionOS camera is unavailable

**GameController**
- `GCSupportsControllerUserInteraction` (Info.plist key) — marks app as supporting game controllers; adds App Store badge
- Game Controller capability (Xcode entitlement) — required alongside plist key

## Code Highlights

Adding hover effect to a card view with custom tap gesture:
```swift
VStack { /* card content */ }
    .hoverEffect()
    .onTapGesture { /* navigate */ }
```

Customizing hover effect shape for a non-rectangular button:
```swift
Button { /* action */ } label: {
    HoneyComb()
        .fill(.yellow)
        .frame(width: 300, height: 300)
        .contentShape(.hoverEffect, HoneyComb())
}
```

Restricting hover effect bounds for large-target buttons:
```swift
Image(systemName: "goforward.10")
    .contentShape(.hoverEffect, CustomizedRectShape(
        customRect: CGRect(x: 0, y: -40, width: 100, height: 100)))
    .frame(width: 500, height: 834)
```

Re-enabling hover effect in a custom `ButtonStyle`:
```swift
struct MyStyle: ButtonStyle {
    func makeBody(configuration: Configuration) -> some View {
        configuration.label
            .background(/* custom */)
            .hoverEffect()
    }
}
```

## Takeaways

- Add `.hoverEffect()` to every custom interactive control — system controls are already covered, but `onTapGesture` views and custom `ButtonStyle` buttons are not.
- Use `AVCaptureDevice.DiscoverySession` to detect hardware availability before assuming back-camera or full camera access.
- Remove or conditionally hide `AVRoutePickerView` and Picture in Picture controls on visionOS since they are unavailable.
- Games must declare `GCSupportsControllerUserInteraction` and add the Game Controller capability for a good player experience.

---
_Source: WWDC23 Session 10094 page (abstract, chapter summaries, code samples, and resource links)._
