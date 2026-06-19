# Data Flow Through SwiftUI
**WWDC19 · Session 226** · [Watch](https://developer.apple.com/videos/play/wwdc2019/226/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15, watchOS 6, tvOS 13

## Overview
This session establishes the foundational data-flow model for SwiftUI. Two guiding principles underpin every tool: every piece of data read in a view creates a dependency on that data, and every piece of data has exactly one source of truth. Violating the second principle — duplicating state across views — is the root cause of inconsistency bugs; SwiftUI's property wrappers make it structurally hard to make that mistake.

The session walks through building a podcast player step by step, introducing each data tool as a new requirement arises. It covers the full spectrum from plain Swift properties for read-only derived data, through `@State` for view-owned mutable values, `@Binding` for shared mutable references, `BindableObject` + `@ObjectBinding` for external reference-type models, and `@EnvironmentObject` for implicit dependency injection down a hierarchy. Combine Publishers are introduced as the bridge between external events and SwiftUI's update cycle.

## Key Topics

- **Two principles** — (1) reading data in a view creates a dependency; (2) every piece of data has a single source of truth. All SwiftUI data tools enforce these principles.
- **Views are a function of state, not a sequence of events** — State changes flow down through the hierarchy; SwiftUI diffs and re-renders only what changed.
- **`@State`** — View-local, value-type, framework-allocated persistent storage. Views should mark state `private`. Every `@State` declaration is a new source of truth owned by that view.
- **`@Binding`** — A first-class reference to a source of truth owned elsewhere. Allows read and write without ownership. Derived from `@State` (or other sources) using the `$` prefix. Components like `Toggle`, `TextField`, and `Slider` all accept `Binding`.
- **External events via `Publisher`** — The `onReceive(_:perform:)` modifier subscribes a view to a Combine `Publisher`; the closure mutates state, which SwiftUI propagates. Publishers must emit on the main thread (`receive(on: RunLoop.main)`).
- **`BindableObject` protocol** — Conformance requires a single `didChange` publisher (a `PassthroughSubject`); call `send()` whenever the model mutates. Used for reference-type models you own and manage (databases, network models, etc.).
- **`@ObjectBinding`** — Property wrapper that subscribes a view to a `BindableObject`. Creates an explicit dependency visible in the view's initializer.
- **`@EnvironmentObject`** — Indirect injection: write a `BindableObject` into the environment with `.environmentObject(_:)` on an ancestor; any descendant that declares `@EnvironmentObject` automatically receives it. Avoids passing the model hop-by-hop.
- **The Environment** — A general-purpose container for indirect data (accent color, layout direction, Dynamic Type size, Dark Mode, and custom values). Declaring a dependency on an environment value causes automatic re-render on change.
- **Reusable component design** — Prefer read-only Swift properties or environment values; use `@Binding` when mutation is needed; reach for `@State` only when data is truly owned by that view (e.g., button highlight state).

## APIs & Frameworks

### SwiftUI **[ALL NEW]**
- `@State` property wrapper — view-local persistent mutable storage
- `@Binding` property wrapper — shared mutable reference without ownership
- `$propertyName` syntax — derives a `Binding` from `@State`, `@ObjectBinding`, or another `Binding`
- `BindableObject` protocol — `var didChange: PassthroughSubject<Void, Never>` (or any `Publisher`)
- `@ObjectBinding` property wrapper — subscribes view to a `BindableObject`
- `@EnvironmentObject` property wrapper — implicit dependency on a `BindableObject` in the environment
- `.environmentObject(_:)` view modifier — writes a `BindableObject` into the environment
- `.onReceive(_:perform:)` view modifier — subscribes view to a Combine `Publisher`
- `.animation(_:)` / `withAnimation(_:_:)` — drives animation from state mutation
- `Toggle(isOn:label:)` — accepts `Binding<Bool>`
- `TextField(_:text:)` — accepts `Binding<String>`
- `Slider(value:)` — accepts `Binding<Double>`
- `Button(action:label:)` — uses internal `@State` for highlight; callers need not manage it
- `VStack`, `Text`, `Image` — basic layout/display primitives

### Combine **[NEW]**
- `PassthroughSubject<Output, Failure>` — publisher for imperative `send()` calls
- `Publisher.receive(on:)` — route emissions to main thread before SwiftUI use

## Code Highlights

View-local toggle state:

```swift
struct PlayerView: View {
    let episode: Episode
    @State private var isPlaying: Bool = false

    var body: some View {
        Button(action: { self.isPlaying.toggle() }) {
            Image(systemName: isPlaying ? "pause.circle" : "play.circle")
        }
    }
}
```

Extracting a reusable component with `@Binding`:

```swift
struct PlayButton: View {
    @Binding var isPlaying: Bool   // no initial value; owned by parent

    var body: some View {
        Button(action: { self.isPlaying.toggle() }) {
            Image(systemName: isPlaying ? "pause.circle" : "play.circle")
        }
    }
}

// In parent — $ derives a Binding from @State
PlayButton(isPlaying: $isPlaying)
```

Subscribing to an external publisher:

```swift
@State private var currentTime: TimeInterval = 0

var body: some View {
    Text(currentTime.formatted())
        .onReceive(timePublisher) { newTime in
            self.currentTime = newTime
        }
}
```

`BindableObject` conformance:

```swift
class PodcastPlayerStore: BindableObject {
    var didChange = PassthroughSubject<Void, Never>()

    var episodes: [Episode] = [] {
        didSet { didChange.send() }
    }
}
```

Using `@EnvironmentObject`:

```swift
// In a top-level view or scene:
ContentView().environmentObject(PodcastPlayerStore())

// In any descendant:
struct PlayerView: View {
    @EnvironmentObject var store: PodcastPlayerStore
    // ...
}
```

## Takeaways

- Every `@State` declaration creates a new source of truth; lift state to the lowest common ancestor when multiple views need it, and pass `@Binding` downward.
- `@ObjectBinding` and `@EnvironmentObject` both subscribe views to a `BindableObject`; prefer `@EnvironmentObject` to avoid threading the model through intermediate views that don't use it.
- SwiftUI's data tools eliminate the need for a view controller: no manual invalidation, no target-action wiring, no notification observation — just describe the dependency and the framework handles updates.
- Before reaching for `@State`, ask whether the data is truly owned by that view; most app data belongs in an external model conforming to `BindableObject`.

---
_Source: WWDC19 Session 226 page (abstract, full transcript, and resource links)._
