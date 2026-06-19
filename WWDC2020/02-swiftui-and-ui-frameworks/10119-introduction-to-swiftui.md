# Introduction to SwiftUI
**WWDC20 · Session 10119** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10119/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, tvOS 14, watchOS 7

## Overview
This session provides an end-to-end walkthrough of building a real multi-platform SwiftUI app ("Sandwiches") entirely from scratch using Xcode 12. It covers the fundamental declarative programming model — views as value types, state as a source of truth, and SwiftUI's automatic dependency tracking — as well as practical Xcode tooling: the live canvas, the library, command-click refactoring, and on-device previews.

The second half of the session explains _why_ SwiftUI's model is architecturally superior to traditional event-driven UIKit programming. By forcing all UI updates through a single `body` property derived from an authoritative source of truth, SwiftUI eliminates the class of UI inconsistency bugs (race conditions, stale views, overlooked event orderings) that have historically plagued imperative frameworks. The session introduces `@State`, `@StateObject`, `@ObservedObject`, `ObservableObject`, and `@Published` as the primary data flow primitives.

The app also demonstrates that a single SwiftUI codebase runs well on iPhone, iPad, and Mac, with the framework automatically adapting `NavigationView` to a split view on wider screens and toolbar items to platform conventions.

## Key Topics

### Declarative View Building
- Views are Swift structs conforming to `View`, not subclasses of `UIView`; they are value types allocated on the stack
- The canvas and code editor are synchronized representations of the same source; edits in either update both
- The library provides draggable views and modifiers; Xcode inserts correct SwiftUI syntax automatically
- Command-click on any view to embed in a stack, extract a subview, or open the inspector

### Composing Views
- `VStack`, `HStack` — vertical and horizontal layout containers
- `List` + `ForEach` — data-driven list with optional static elements mixed in
- `NavigationView` + `NavigationLink` — push navigation; automatically becomes a split view on iPad/Mac
- `Label` — a paired title + icon view that adapts its appearance to context (lists, menus, toolbars)
- `Spacer` — flexible space that expands to fill available room; `minLength: 0` removes the default minimum

### Modifiers
- `.resizable()` — enables image scaling
- `.aspectRatio(contentMode:)` — `.fit` or `.fill`
- `.cornerRadius(_:)` — rounds image corners
- `.foregroundColor(_:)` — sets text/symbol color
- `.font(_:)` — applies a dynamic type style (`.subheadline`, `.headline`, etc.)
- `.ignoresSafeArea(_:edges:)` — expands a view into the safe area on specified edges
- `.navigationTitle(_:)` — sets the navigation bar title
- `.toolbar { ... }` — adds views as toolbar items
- `.onTapGesture { }` — attaches a tap handler
- `withAnimation { }` — wraps a state mutation to produce an animated transition
- `.transition(_:)` — specifies how a view enters/exits (e.g., `.move(edge:)`)
- `.onMove(perform:)` / `.onDelete(perform:)` — list editing callbacks
- `.background(_:)` — places a view behind the modified view
- `.padding()` — adds spacing around a view

### Data Flow
- `@State` — creates persistent storage for a value-type source of truth within a view; private
- `@StateObject` — creates a persistent reference to an `ObservableObject`; used in app/view as the owning source of truth
- `@ObservedObject` — observes an externally owned `ObservableObject` for changes
- `ObservableObject` protocol — marks a class as observable by SwiftUI
- `@Published` — marks a property on an `ObservableObject` so changes trigger view updates
- `Identifiable` protocol — required by `List`/`ForEach` for element identity tracking
- Binding — a read-write derived value (introduced conceptually; not shown in code in this session)
- Environment — set with `.environment(keyPath, value)`; flows down the hierarchy; used to configure previews (Dynamic Type size, color scheme, locale, layout direction)

### Xcode Previews
- `PreviewProvider` — protocol for defining preview structs
- Multiple previews can be defined in one `PreviewProvider` body
- `.environment(\.sizeCategory, .accessibilityExtraExtraExtraLarge)` — tests large Dynamic Type
- `.environment(\.colorScheme, .dark)` — tests Dark Mode
- `.environment(\.locale, Locale(identifier: "ar"))` — tests localization
- `.environment(\.layoutDirection, .rightToLeft)` — tests RTL layout
- Live preview mode (play button) — runs real app code interactively in the canvas; no build-and-run needed
- On-device preview — sends canvas preview directly to a connected device

### Multi-Platform
- A single shared codebase works across iOS, iPadOS, macOS; `NavigationView` adapts automatically
- `#if os(iOS)` — conditional compilation for platform-specific toolbar items
- Multi-platform app template in Xcode 12

### Automatic Localization
- `Text("literal")` is automatically treated as a localizable key
- `Text(stringVariable)` is used as-is (not localized)
- String interpolation in `Text` localizes correctly

## APIs & Frameworks

- **SwiftUI**
  - `View` protocol — requires `var body: some View`
  - `VStack`, `HStack` — layout containers
  - `List` — data-driven list view
  - `ForEach` — creates views from a collection; supports `onMove` and `onDelete`
  - `NavigationView` — navigation container (split view on iPad/Mac)
  - `NavigationLink(destination:label:)` — pushes a destination view
  - `Label(_:systemImage:)` — icon + text pair
  - `Spacer(minLength:)` — flexible space
  - `Image(_:)` — displays an image asset or SF Symbol
  - `Image.resizable()` — enables scaling
  - `Image.aspectRatio(_:contentMode:)` — controls scaling mode (`.fit`/`.fill`)
  - `Text(_:)` — displays localized or raw string
  - `Button(action:label:)` — tappable button
  - `EditButton()` — toggles edit mode for enclosing list
  - `@State` property wrapper
  - `@StateObject` property wrapper
  - `@ObservedObject` property wrapper
  - `ObservableObject` protocol
  - `@Published` property wrapper
  - `Identifiable` protocol
  - `withAnimation(_:_:)` — animates a state change
  - `.transition(_:)` modifier — e.g., `.move(edge: .bottom)`
  - `.onTapGesture(perform:)` modifier
  - `.toolbar(content:)` modifier
  - `.onMove(perform:)` modifier
  - `.onDelete(perform:)` modifier
  - `.navigationTitle(_:)` modifier
  - `.ignoresSafeArea(_:edges:)` modifier
  - `.environment(_:_:)` modifier
  - `.cornerRadius(_:)` modifier
  - `.foregroundColor(_:)` modifier
  - `.font(_:)` modifier — `.subheadline`, `.headline`
  - `.padding()` modifier
  - `.background(_:)` modifier
  - `PreviewProvider` protocol — for Xcode previews
  - `App` protocol — entry point struct with `@main`
  - `WindowGroup` — scene for multi-window apps
  - `Scene` protocol

## Code Highlights

Basic view with `@State` and tap gesture:
```swift
struct SandwichDetail: View {
    let sandwich: Sandwich
    @State private var zoomed = false

    var body: some View {
        Image(sandwich.imageName)
            .resizable()
            .aspectRatio(contentMode: zoomed ? .fill : .fit)
            .onTapGesture { zoomed.toggle() }
    }
}
```

Animated state change with a custom transition:
```swift
if !zoomed {
    SpicyBanner()
        .transition(.move(edge: .bottom))
}
// Called inside:
withAnimation { zoomed.toggle() }
```

Connecting an `ObservableObject` model in the App entry point:
```swift
@main
struct SandwichesApp: App {
    @StateObject var store = SandwichStore()

    var body: some Scene {
        WindowGroup {
            ContentView(store: store)
        }
    }
}
```

## Takeaways
- SwiftUI views are value types; extracting subviews has virtually zero runtime cost — refactor freely.
- `@State` and `@StateObject` are sources of truth; everything else is derived — eliminating the entire class of UI inconsistency bugs caused by mismatched event handler orderings.
- Xcode previews support multiple simultaneous configurations (Dark Mode, Dynamic Type, locale, RTL) using `.environment` modifiers — most apps can be built and debugged without ever running the full simulator.
- A single SwiftUI codebase adapts to iPhone, iPad, and Mac with minimal platform-specific conditional code.

---
_Source: WWDC20 Session 10119 page (abstract, transcript, and code samples)._
