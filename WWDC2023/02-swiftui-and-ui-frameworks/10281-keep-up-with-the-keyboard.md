# Keep Up with the Keyboard
**WWDC23 · Session 10281** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10281/)

_Platforms:_ iOS 17, iPadOS 17

## Overview
This session covers two significant changes to the iOS keyboard in 2023: a major architectural change (out-of-process keyboard) and a new text entry feature (inline predictions). It also provides updated guidance for handling the keyboard correctly in Stage Manager environments, where apps are no longer guaranteed to be full screen and naive keyboard notification handling can produce incorrect layouts.

The keyboard has been moved to its own process on iPhone in iOS 17, running outside the app process for improved security, privacy, reduced memory usage, and future extensibility. For most apps this is transparent, but introduces asynchronous timing differences in keyboard lifecycle events that time-sensitive code must account for.

## Key Topics

### Out-of-Process Keyboard (iOS 17, iPhone)
- Keyboard UI now runs in its own process, completely outside the app process.
- Previously synchronous keyboard initialization is now asynchronous.
- The keyboard process coordinates animations with the app process independently.
- Security/privacy benefit: typing data is isolated from the app process.
- Memory benefit: one keyboard instance system-wide instead of one per app.
- For most apps: completely transparent, no adoption required.
- For time-sensitive code: be aware that keyboard notifications and text insertions may have slightly different timing relative to `becomeFirstResponder`.

### Stage Manager and Keyboard Layout Considerations
- Apps in Stage Manager are no longer full screen; the keyboard's coordinate space and the app's coordinate space diverge.
- The raw keyboard height from notifications is no longer a reliable adjustment value in Stage Manager.
- The correct adjustment is the intersection of the keyboard frame and the app's view, not the keyboard's total height.
- Hardware keyboard assistant toolbar: full-size toolbar acts as part of the keyboard; mini toolbar behavior differs inside vs. outside Stage Manager.

### Keyboard Layout Guide (Recommended Approach)
- `UIKeyboardLayoutGuide` (introduced iOS 15) automatically handles all scenarios, including Stage Manager.
- New in iOS 17: three new customization properties on `UIKeyboardLayoutGuide`:
  - `followsUndockedKeyboard` **[NEW]** — when `true`, guide tracks the floating keyboard even when not docked; default is `false`.
  - `usesBottomSafeArea` **[NEW]** — when `false`, guide's baseline is the view's bottom edge (not safe area); useful for input-accessory-like views with full-bleed backgrounds.
  - `keyboardDismissPadding` **[NEW]** — padding above the keyboard that activates the scroll-to-dismiss gesture; fixes issue where dismiss gesture only triggered inside the keyboard frame.

### Keyboard Notifications (Manual Approach)
- `UIResponder.keyboardWillShowNotification`, `keyboardDidShowNotification`, `keyboardWillHideNotification`, `keyboardDidHideNotification` — still available.
- New in iOS 16.1: notifications include a `UIScreen` as the `object`; use to verify the keyboard is on the same screen as the app.
- Correct handling for Stage Manager:
  1. Check that `notification.object as? UIScreen` matches `view.window?.screen`.
  2. Convert `UIResponder.keyboardFrameEndUserInfoKey` frame from screen coordinate space to view coordinate space using `convert(_:to:)`.
  3. Calculate the intersection of the view's bounds with the converted keyboard frame; use intersection height as the layout adjustment.

### SwiftUI Keyboard Handling
- SwiftUI automatically includes the keyboard in the safe area.
- No keyboard-specific code required — views resize automatically.
- `SafeAreaRegions` can be used to fine-tune which regions affect layout.
- `FocusState` manages focus-driven keyboard presentation.

### Inline Predictions (iOS 17, New Feature)
- English keyboard now shows next-word predictions inline in text fields.
- Predictions generated entirely on-device using only the context of the focused field.
- New `UITextInputTraits.inlinePredictionType` property **[NEW]**:
  - `.default` — system decides (active in most fields, disabled in search/password fields).
  - `.yes` — explicitly enable.
  - `.no` — explicitly disable.

## APIs & Frameworks
- `UIKeyboardLayoutGuide` — auto layout guide tracking keyboard frame; iOS 15+
- `UIKeyboardLayoutGuide.followsUndockedKeyboard` **[NEW]** — track floating keyboard
- `UIKeyboardLayoutGuide.usesBottomSafeArea` **[NEW]** — control baseline for dismissed state
- `UIKeyboardLayoutGuide.keyboardDismissPadding` **[NEW]** — extend dismiss gesture trigger zone
- `UIResponder.keyboardWillShowNotification` — keyboard will appear
- `UIResponder.keyboardDidShowNotification` — keyboard appeared
- `UIResponder.keyboardWillHideNotification` — keyboard will disappear
- `UIResponder.keyboardDidHideNotification` — keyboard disappeared
- `UIResponder.keyboardFrameEndUserInfoKey` — keyboard's end frame in screen coordinates
- `UIScreen.coordinateSpace` — screen coordinate space for frame conversion
- `UICoordinateSpace.convert(_:to:)` — converts a rect between coordinate spaces
- `UITextInputTraits.inlinePredictionType` **[NEW]** — controls inline prediction display
- `UITextInlinePredictionType` **[NEW]** — enum: `.default`, `.yes`, `.no`
- `UITextView.inlinePredictionType` **[NEW]** — sets inline prediction on a text view
- `UITextField.inlinePredictionType` **[NEW]** — sets inline prediction on a text field
- `UIResponder.becomeFirstResponder()` — triggers keyboard presentation
- `FocusState` (SwiftUI) — manages focus and keyboard in SwiftUI views
- `SafeAreaRegions` (SwiftUI) — controls which safe area regions affect layout

## Code Highlights

One-line keyboard layout guide constraint:
```swift
view.keyboardLayoutGuide.topAnchor
    .constraint(equalTo: textView.bottomAnchor).isActive = true
```

Input-accessory-like view using `usesBottomSafeArea = false`:
```swift
view.keyboardLayoutGuide.usesBottomSafeArea = false
// Backdrop extends to bottom; text field stays above keyboard
view.keyboardLayoutGuide.topAnchor
    .constraint(greaterThanOrEqualToSystemSpacingBelow: textField.bottomAnchor, multiplier: 1.0)
    .isActive = true
view.keyboardLayoutGuide.topAnchor
    .constraint(equalTo: backdrop.bottomAnchor).isActive = true
```

Correct Stage Manager notification handling:
```swift
func handleWillShowOrHideKeyboardNotification(notification: NSNotification) {
    guard let screen = notification.object as? UIScreen,
          screen.isEqual(view.window?.screen) else { return }

    let endFrameKey = UIResponder.keyboardFrameEndUserInfoKey
    guard let keyboardFrameEnd = (notification.userInfo?[endFrameKey] as? CGRect) else { return }

    let convertedKeyboardFrameEnd = screen.coordinateSpace.convert(keyboardFrameEnd, to: view)
    let viewIntersection = view.bounds.intersection(convertedKeyboardFrameEnd)
    let bottomOffset = viewIntersection.isEmpty ? view.safeAreaInsets.bottom : viewIntersection.height
    movingBottomConstraint.constant = bottomOffset
}
```

Enabling inline predictions:
```swift
let textView = UITextView(frame: frame)
textView.inlinePredictionType = .yes
```

## Takeaways
- The keyboard's out-of-process architecture is transparent for most apps, but time-sensitive keyboard notification code should account for asynchronous timing changes.
- Use `UIKeyboardLayoutGuide` with its new iOS 17 properties (`followsUndockedKeyboard`, `usesBottomSafeArea`, `keyboardDismissPadding`) instead of raw notification values — it handles Stage Manager correctly out of the box.
- When handling keyboard notifications manually in Stage Manager, always convert the keyboard frame to the view's coordinate space and use the intersection height, not the raw keyboard height.
- Inline word predictions (`UITextInlinePredictionType`) are on by default in most text fields in iOS 17; explicitly set `.no` for fields where predictions would be inappropriate.

---
_Source: WWDC23 Session 10281 page (abstract, chapter summaries, code samples, and resource links)._
