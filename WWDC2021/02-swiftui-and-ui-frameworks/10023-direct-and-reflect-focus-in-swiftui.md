# Direct and reflect focus in SwiftUI
**WWDC21 · Session 10023** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10023/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, tvOS 15, watchOS 8

## Overview
This session introduces the two new focus management APIs in SwiftUI: `@FocusState` and `focusSection()`. Together they give developers programmatic control over keyboard focus, keyboard dismissal, and directional navigation—capabilities that previously required UIKit interop or platform-specific workarounds.

The session uses a cross-platform vacation planner app to demonstrate: moving focus to an invalid form field after submission, showing/hiding an error border based on current focus, dismissing the keyboard programmatically, and making a remote button on Apple TV reachable from non-adjacent form fields using focus sections.

## Key Topics

### Focus in SwiftUI — Background
- Focus is the system through which non-touch inputs (keyboards, TV remotes, game controllers, Switch Control) are routed to a specific view.
- SwiftUI manages default focus behavior for standard components based on platform conventions.
- Two new APIs allow apps to take custom control: `@FocusState` for programmatic focus changes, and `focusSection()` for extending focusable regions.

### `@FocusState` Property Wrapper **[NEW]**
- Holds a value of any hashable optional type; its value reflects (and can set) which view is currently focused.
- The value is `nil` when focus is in an unrelated part of the screen—setting it to `nil` also dismisses the keyboard.
- Works bidirectionally: focus placement updates the FocusState value, and writing to the FocusState value moves focus.
- Works with any focusable view, not just text fields—applicable across iOS, tvOS, watchOS, and macOS.

### `focused(_:equals:)` Modifier **[NEW]**
- Binds a specific focusable view to a value in a `@FocusState` property.
- When focus lands on that view, the `@FocusState` variable is set to the specified value.
- When the `@FocusState` variable is set to that value programmatically, focus moves to that view.
- Uses enum, String, Int, or any `Hashable` type as the identifier.

### Dismissing the Keyboard
- Set the `@FocusState` variable to `nil` to remove focus from all local views and dismiss the software keyboard.
- Replaces the common UIKit workaround of calling `resignFirstResponder()` on the active text field.

### Reading Focus for Visual Feedback
- Because `@FocusState` is readable state, it can be used in view modifier conditions directly.
- Example: show a red border on a text field only while that field is focused AND its content is invalid.

### `focusSection()` Modifier **[NEW]**
- Applies to any container view (e.g., `VStack`, `HStack`).
- Makes the entire frame of that container a valid focus target for directional navigation, as long as it contains at least one focusable subview.
- Allows focus to jump between non-adjacent regions (e.g., login form on the left, Browse button at the bottom right on tvOS) by swiping in the direction of the section.
- Adding `focusSection()` to both containers enables round-trip navigation (left/right swipe between the two regions).

## APIs & Frameworks

**SwiftUI**
- `@FocusState` property wrapper — tracks and controls focus placement within a view **[NEW]**
- `.focused(_:equals:)` modifier — binds a view to a `@FocusState` value **[NEW]**
- `.focused(_:)` modifier — boolean-binding variant for single focusable view (implied) **[NEW]**
- `.focusSection()` modifier — extends a container's frame as a directional focus navigation target **[NEW]**
- `.onSubmit {}` — called when the user submits a form field (connected to `submitLabel`) **[NEW in iOS 15]**
- `.submitLabel(_:)` — sets the Return key label (`.next`, `.go`, etc.) **[NEW in iOS 15]**
- `TextField` / `SecureField` — standard focusable form fields **[existing]**
- `SignInWithAppleButton` — focusable button **[existing]**

## Code Highlights

Declaring a typed `@FocusState` and binding text fields:
```swift
enum Field: Hashable {
    case email
    case password
}

struct LoginView: View {
    @FocusState private var focusedField: Field?
    @State private var email = ""
    @State private var password = ""

    var body: some View {
        VStack {
            TextField("Email", text: $email)
                .focused($focusedField, equals: .email)
                .border(Color.red,
                        width: (focusedField == .email && !isEmailValid) ? 2 : 0)

            SecureField("Password", text: $password)
                .focused($focusedField, equals: .password)
        }
        .onSubmit {
            if !isEmailValid {
                focusedField = .email  // move focus back to email field
            } else {
                focusedField = nil     // dismiss keyboard
            }
        }
    }
}
```

Using `focusSection()` on tvOS to connect non-adjacent regions:
```swift
HStack {
    VStack {
        // Login fields
        TextField("Email", ...)
        SecureField("Password", ...)
    }
    .focusSection()  // makes this VStack a directional focus target

    VStack {
        Image(...)
        BrowsePhotosButton()
    }
    .focusSection()  // swipe right from login fields lands here
}
```

## Takeaways
- `@FocusState` + `.focused(_:equals:)` provide full bidirectional focus control: read which field is active, move focus programmatically, and dismiss the keyboard by setting the value to `nil`.
- `@FocusState` can drive any state-dependent visual (borders, colors, error messages) without additional `@State` variables.
- `focusSection()` is the tvOS/spatial solution for making distant UI reachable via directional swipe; apply it to both source and target containers for bidirectional navigation.
- These APIs work across iOS, iPadOS, macOS, tvOS, and watchOS for any focusable view—not just text fields.

---
_Source: WWDC21 Session 10023 page (abstract, full transcript, code samples, and resource links)._
