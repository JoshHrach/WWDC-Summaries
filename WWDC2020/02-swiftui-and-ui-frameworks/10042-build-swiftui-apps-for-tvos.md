# Build SwiftUI Apps for tvOS
**WWDC20 · Session 10042** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10042/)

_Platforms:_ tvOS 14

## Overview
This session covers SwiftUI additions specific to tvOS 14: new button styles (`CardButtonStyle`), context menus, focus management APIs (`isFocused` environment variable, `prefersDefaultFocus`, `focusScope`, `resetFocus`), and `LazyHGrid`/`LazyVGrid` for building horizontal shelf layouts common on Apple TV. The session uses a music streaming app as the running demo, building album shelves, a "Now Playing" screen with per-song focus states, and a login screen with dynamic default focus.

The focus system is central to all tvOS interaction. The new `isFocused` environment variable eliminates the need for the older `onFocusChange` callback approach, letting any descendant view in the hierarchy read focus state without being directly focusable itself. Combined with `prefersDefaultFocus` and `focusScope`, developers can express focus intent declaratively and scope it to a specific view subtree to avoid unintended global focus changes.

Lazy grids make horizontal shelf layouts — a defining pattern of the Apple TV interface — trivial to implement in SwiftUI. A `LazyHGrid` inside a horizontal `ScrollView`, combined with `CardButtonStyle` buttons, reproduces the animated card-raise behavior seen in all first-party Apple TV apps.

## Key Topics

**CardButtonStyle (New)**
A tvOS-specific button style that creates a raised platter with directional movement when the user drags on the Siri Remote. Applied with `.buttonStyle(CardButtonStyle())`. Use for content tiles (album art, show posters, etc.).

**Custom Button Styles**
Conform to `ButtonStyle` and implement `makeBody(configuration:)` to return a fully custom view. `configuration.isPressed` and `configuration.label` are available. Use when the default highlight and focus ring from platform button styles are not desired.

**Context Menus on tvOS**
Added via `.contextMenu { }` modifier on any view or button. Triggered by long-press on the Siri Remote. Items are regular `Button` views. New in tvOS 14.

**`isFocused` Environment Variable (New)**
`@Environment(\.isFocused) var isFocused: Bool` — reads whether the nearest focusable ancestor is currently focused. Available to any view in the hierarchy, not just focusable views themselves. Replaces the `onFocusChange` callback on `focusable()` for most use cases.

**`focusable()` Modifier**
Wraps a non-intrinsically-focusable view to make it focusable. Do not apply to views that are already focusable (e.g., `Button`, `List`, UIKit views managing focus), as it adds a redundant focusable wrapper.

**Default Focus Management (New)**
- `prefersDefaultFocus(_:in:)` — declares that a view prefers to be the initial focus target, conditionally based on a Boolean. Only takes effect within the specified namespace.
- `focusScope(_:)` — limits default focus preferences to a specific view subtree, preventing accidental global focus side effects.
- `@Namespace` — dynamic property that generates a unique identifier used to link `focusScope` and `prefersDefaultFocus` calls together.
- `@Environment(\.resetFocus) var resetFocus` — environment action that resets focus to the default target within a namespace.

**Lazy Grids for Shelf Layouts**
`LazyHGrid(rows:)` inside `ScrollView([.horizontal])` creates a horizontally scrolling grid. `GridItem` configures each row's size (`.flexible()`, `.fixed(_:)`) and spacing. Content is loaded lazily as the user scrolls. Stack multiple `ShelfView` instances in a `VStack` to produce the multi-shelf layout common in Apple TV apps.

## APIs & Frameworks

### SwiftUI — Button Styles
- `CardButtonStyle` **[NEW in tvOS 14]** — raised platter with directional movement effect on Siri Remote drag
- `ButtonStyle` protocol — custom button style via `makeBody(configuration:)`; `ButtonStyle.Configuration` provides `.label` and `.isPressed`
- `.buttonStyle(_:)` modifier — applies a button style to any `Button`

### SwiftUI — Context Menus
- `.contextMenu { }` **[NEW on tvOS 14]** — adds a long-press context menu to any view; content is a `ViewBuilder` of `Button` views

### SwiftUI — Focus
- `.focusable(_ isFocusable: Bool = true, onFocusChange:)` — makes a non-intrinsically-focusable view focusable (tvOS 13+); prefer `isFocused` env variable over `onFocusChange` callback
- `@Environment(\.isFocused) var isFocused: Bool` **[NEW in tvOS 14]** — reads focus state of nearest focusable ancestor; available to any descendant view
- `.prefersDefaultFocus(_ prefersDefaultFocus: Bool, in namespace: Namespace.ID)` **[NEW in tvOS 14]** — expresses initial focus preference for a view within a namespace
- `.focusScope(_ namespace: Namespace.ID)` **[NEW in tvOS 14]** — limits focus preference effects to this view's subtree
- `@Namespace private var namespace` — generates a unique `Namespace.ID` for scoping focus APIs
- `@Environment(\.resetFocus) var resetFocus: ResetFocusAction` **[NEW in tvOS 14]** — resets focus to the default target within a namespace
- `resetFocus(in: namespace)` — call to trigger focus reset

### SwiftUI — Layouts
- `LazyHGrid(rows:alignment:spacing:pinnedViews:content:)` **[NEW]** — horizontally scrolling lazy grid
- `LazyVGrid(columns:alignment:spacing:pinnedViews:content:)` **[NEW]** — vertically scrolling lazy grid
- `GridItem(.flexible(), spacing:)` — flexible-size grid item
- `GridItem(.fixed(_:), spacing:)` — fixed-size grid item
- `ScrollView([.horizontal])` — horizontal scroll container wrapping `LazyHGrid`

## Code Highlights

CardButtonStyle applied to a playlist shelf:
```swift
struct ShelfView: View {
    var body: some View {
        ScrollView([.horizontal]) {
            LazyHGrid(rows: [GridItem()]) {
                ForEach(playlists, id: \.self) { playlist in
                    Button(action: goToPlaylist) {
                        Image(playlist.coverImage)
                            .resizable()
                            .frame(width: 300, height: 300)
                    }
                    .buttonStyle(CardButtonStyle())
                }
            }
        }
    }
}
```

`isFocused` to show/hide details in a child view:
```swift
struct DetailsView: View {
    let songName: String
    let artistAndAlbum: String
    let artistName: String

    @Environment(\.isFocused) var isFocused: Bool

    var body: some View {
        VStack {
            Text(songName)
            Text(isFocused ? artistAndAlbum : artistName)
        }
    }
}
```

Default focus scoped to a login form:
```swift
@Namespace private var namespace
@State private var areCredentialsFilled = false
@Environment(\.resetFocus) var resetFocus

var body: some View {
    VStack {
        TextField("Username", text: $username)
            .prefersDefaultFocus(!areCredentialsFilled, in: namespace)
        SecureField("Password", text: $password)
        Button("Log In", action: logIn)
            .prefersDefaultFocus(areCredentialsFilled, in: namespace)
        Button("Clear") {
            username = ""; password = ""
            areCredentialsFilled = false
            resetFocus(in: namespace)
        }
    }
    .focusScope(namespace)
}
```

## Takeaways
- Use `CardButtonStyle` for content tiles on tvOS to get the platform's signature raised-platter and directional-movement effect with a single modifier; reserve custom `ButtonStyle` conformances for cases where the platform highlight effect should be suppressed.
- Read `@Environment(\.isFocused)` in child views to react to focus state without adding `focusable()` wrappers — this is the tvOS 14 replacement for the `onFocusChange` callback and works for any descendant of a focusable view.
- Always pair `prefersDefaultFocus(_:in:)` with `.focusScope(_:)` and a `@Namespace` to prevent default focus declarations from affecting parts of the view hierarchy outside the intended subtree.
- Build tvOS shelf layouts with `LazyHGrid` inside `ScrollView([.horizontal])` — lazy loading, flexible grid items, and `CardButtonStyle` combine to reproduce the standard Apple TV content shelf pattern in minimal code.

---
_Source: WWDC20 Session 10042 page (transcript, code samples, and resource links)._
