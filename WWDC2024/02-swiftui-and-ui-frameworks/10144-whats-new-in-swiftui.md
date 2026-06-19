# What's New in SwiftUI
**WWDC24 · Session 10144** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10144/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia 15, visionOS 2, watchOS 11, tvOS 18

## Overview
SwiftUI's 2024 update is organized into three themes: **Fresh apps** (TabView redesign, sheet sizing, zoom transition, custom controls, Swift Charts function/vectorized plots, TableColumnForEach, MeshGradient, document launch, SF Symbols 6), **Harnessing the platform** (macOS window customization, new input APIs, Live Activities on watchOS), and **Framework foundations** (custom containers, ease-of-use improvements including `@Previewable`, scrolling enhancements, Swift 6 language mode support, UIKit/AppKit animation interoperability). The session closes with visionOS-specific additions: volume baseplate control, immersion level and passthrough effects, and a new `TextRenderer` protocol for custom text effects.

## Key Topics

### TabView Redesign
`TabView` gets a new type-safe `Tab` view builder syntax. Applying `.tabViewStyle(.sidebarAdaptable)` makes iPadOS apps adapt between a floating tab bar (compact) and a full sidebar (regular), with full programmatic customization support via `TabViewCustomization`. The tab bar floats above content on iOS 18. On tvOS, the sidebar style is also supported; on macOS, tab view can appear as a sidebar or segmented control in the toolbar. Individual tabs get a `customizationID` for user reordering and hiding.

### Presentation Sizing and Zoom Transition
`.presentationSizing(.form)` and `.presentationSizing(.page)` unify sheet sizing across platforms. A new zoom navigation transition (`.navigationTransition(.zoom(sourceID:in:))` paired with `.matchedTransitionSource(id:in:)`) produces an expand-from-source animation when pushing a destination view.

### Custom Controls (Control Center / Lock Screen)
`ControlWidget` is a new widget type that surfaces in Control Center, the Lock Screen, and can be activated by the Action Button on iPhone. Built with App Intents, `ControlWidgetButton` and `ControlWidgetToggle` require just a few lines of code. (See session 10157 for the full Controls API.)

### Swift Charts: Function and Vectorized Plots
`LinePlot(x:y:)` accepts a closure returning a `Double`, drawing a mathematical function as a continuous line. Vectorized plotting enables performance-efficient rendering of large datasets.

### TableColumnForEach
`TableColumnForEach` allows a dynamic number of `TableColumn` instances based on data — enabling tables with a variable number of columns determined at runtime.

### MeshGradient
`MeshGradient(width:height:points:colors:)` creates a 2D gradient from a grid of control points and colors. Interpolation between points produces smooth, organic color fields — ideal for backgrounds, invitations, and hero imagery.

### Document Launch Experience
A new `DocumentLaunchScene` type lets document-based apps customize their launch screen with a bold title, custom background, and accessory views.

### SF Symbols 6 Integration
SwiftUI surfaces three new symbol animation presets: `.wiggle`, `.breathe`, and `.rotate`. The Replace animation now defaults to `.magic` behavior (smooth badge/slash transitions). All applied via `.symbolEffect(_:)`.

### macOS Window Customization
`.windowStyle(.plain)` removes the default window chrome for floating utility windows. `defaultWindowPlacement` positions windows based on display and content size at first launch. `WindowDragGesture` enables drag-to-reposition from any view. New scene type: `UtilityWindow`.

### Input Methods
- **visionOS**: New closure-based `.hoverEffect` modifier controls how views transition between active and inactive hover states
- **Keyboard**: `modifierKeyAlternate` adds Option-key alternative menu items; `onModifierKeysChanged` lets any view respond to modifier key state
- **Pointer**: `pointerStyle(_:)` customizes the system pointer appearance (e.g., `.frameResize` on resize anchors)
- **Apple Pencil**: `.onPencilSqueeze` handles the Apple Pencil Pro squeeze gesture, including hover location

### Live Activities on watchOS
iOS Live Activities automatically appear on Apple Watch without any code changes. The new `.supplementalActivityFamily(.small)` allows tailored watchOS content. `.handGestureShortcut(.primaryAction)` activates actions via double tap on Apple Watch.

### Framework Foundations
**Custom containers**: `ForEach(subviewsOf:)` and `Group(subviewsOf:)` let custom container views iterate, inspect, and re-compose their children — enabling List-like containers with custom layout.

**Ease of use**: `@Previewable` macro declares `@State` and `@Query` inline in `#Preview` blocks. `.searchFocused(_:)` controls search field focus. `Color(cgColor:)` initializer. `@Entry` macro simplifies custom `EnvironmentValues` keys. `containerBackground` works on watchOS. `NavigationLink` and `List` improvements. `Text` reference date style for countdowns.

**Scrolling**: `onScrollGeometryChange` observes scroll position; `scrollPosition(id:anchor:)` for programmatic anchor-relative scrolling; `onScrollVisibilityChange` fires when items enter/leave the visible region.

**Swift 6**: SwiftUI views and view modifiers are `@MainActor`-isolated in Swift 6 mode — eliminating most data-race warnings in UI code. New `UIView.animate(body:)` and `NSAnimationContext.animate(body:)` bridge SwiftUI animations into UIKit/AppKit; `UIViewRepresentable`/`NSViewRepresentable` context gains animation bridging API.

### visionOS Crafting Experiences
**Volumes**: `.volumeBaseplateVisibility(.hidden)` suppresses the system baseplate. `.onVolumeViewpointChange` reacts when the viewer moves to a different side.

**Immersive spaces**: `ImmersiveSpace` now supports `allowedImmersionRange` for min/max immersion levels. `.preferredSurroundingsEffect(.dim)` or `.colorMultiply(_:)` apply passthrough video effects around progressive immersive spaces.

**TextRenderer protocol**: Custom `TextRenderer` implementations receive a `Text.Layout` and can produce arbitrary `GraphicsContext` drawing — enabling effects like word-by-word karaoke highlighting, glows, and blurs on text.

## APIs & Frameworks

**SwiftUI — Navigation & Tabs**
- `Tab` view **[NEW]** — type-safe tab declaration with `customizationID`
- `.tabViewStyle(.sidebarAdaptable)` **[NEW]**
- `TabViewCustomization` **[NEW]** — programmatic reorder/hide control
- `.navigationTransition(.zoom(sourceID:in:))` **[NEW]**
- `.matchedTransitionSource(id:in:)` **[NEW]**
- `.presentationSizing(.form)` / `.presentationSizing(.page)` **[NEW]**
- `DocumentLaunchScene` **[NEW]**

**SwiftUI — Controls & Widgets**
- `ControlWidget` protocol **[NEW]**
- `ControlWidgetButton` **[NEW]**
- `ControlWidgetToggle` **[NEW]**
- `StaticControlConfiguration` **[NEW]**

**SwiftUI — Visual Effects**
- `MeshGradient(width:height:points:colors:)` **[NEW]**
- `WiggleSymbolEffect` / `BreatheSymbolEffect` / `RotateSymbolEffect` **[NEW]** (via `.symbolEffect`)
- `.replace.magic(fallback:)` **[NEW]** — Magic Replace for symbol transitions

**Swift Charts**
- `LinePlot(x:y:)` **[NEW]** — function plotting
- `BarPlot` / vectorized plot APIs **[NEW]**

**SwiftUI — Tables**
- `TableColumnForEach` **[NEW]** — dynamic columns

**SwiftUI — Custom Containers**
- `ForEach(subviewsOf:)` **[NEW]**
- `Group(subviewsOf:)` **[NEW]**

**SwiftUI — Ease of Use**
- `@Previewable` macro **[NEW]**
- `@Entry` macro **[NEW]** — environment value key declaration
- `.searchFocused(_:)` **[NEW]**
- `Text` reference date style **[NEW]**

**SwiftUI — Scrolling**
- `onScrollGeometryChange(_:action:)` **[NEW]**
- `onScrollVisibilityChange(threshold:_:)` **[NEW]**
- `scrollPosition(id:anchor:)` **[NEW]**

**SwiftUI — Windows**
- `.windowStyle(.plain)` — borderless chrome **[NEW]**
- `defaultWindowPlacement(_:)` **[NEW]**
- `WindowDragGesture` **[NEW]**
- `UtilityWindow` scene type **[NEW]**
- `pushWindow` environment action **[NEW]**

**SwiftUI — Input**
- `.hoverEffect(_:)` (closure-based) **[NEW]**
- `modifierKeyAlternate(_:_:)` **[NEW]**
- `onModifierKeysChanged(_:)` **[NEW]**
- `pointerStyle(_:)` **[NEW]**
- `.onPencilSqueeze(_:)` **[NEW]**

**SwiftUI — watchOS / Live Activities**
- `.supplementalActivityFamily(.small)` **[NEW]**
- `.handGestureShortcut(.primaryAction)` **[NEW]**

**SwiftUI — visionOS**
- `.volumeBaseplateVisibility(_:)` **[NEW]**
- `.onVolumeViewpointChange(_:)` **[NEW]**
- `allowedImmersionRange` on `ImmersiveSpace` **[NEW]**
- `.preferredSurroundingsEffect(_:)` **[NEW]**
- `TextRenderer` protocol **[NEW]** — custom text rendering and effects

**UIKit / AppKit Interop**
- `UIView.animate(body:)` **[NEW]** — animate UIKit changes with SwiftUI animations
- `NSAnimationContext.animate(body:)` **[NEW]**
- `UIViewRepresentable` / `NSViewRepresentable` animation bridging context **[NEW]**

## Code Highlights

TabView with sidebar and customization:
```swift
@State var customization = TabViewCustomization()

TabView {
    Tab("Parties", image: "party.popper") { PartiesView() }
        .customizationID("karaoke.tab.parties")
    Tab("Song List", image: "music.note.list") { SongListView() }
        .customizationID("karaoke.tab.songlist")
}
.tabViewStyle(.sidebarAdaptable)
.tabViewCustomization($customization)
```

Zoom navigation transition:
```swift
NavigationLink {
    PartyDetailView(party: party)
        .navigationTransition(.zoom(sourceID: party.id, in: namespace))
} label: { Text("Party!") }
.matchedTransitionSource(id: party.id, in: namespace)
```

MeshGradient:
```swift
MeshGradient(width: 3, height: 3,
    points: [ .init(0,0), .init(0.5,0), .init(1,0),
              .init(0,0.5), .init(0.5,0.5), .init(1,0.5),
              .init(0,1), .init(0.5,1), .init(1,1) ],
    colors: [.purple, .blue, .indigo,
             .pink, .white, .cyan,
             .red, .orange, .yellow])
```

ControlWidget for Control Center:
```swift
struct StartPartyControl: ControlWidget {
    var body: some ControlWidgetConfiguration {
        StaticControlConfiguration(kind: "com.apple.karaoke_start_party") {
            ControlWidgetButton(action: StartPartyIntent()) {
                Label("Start the Party!", systemImage: "music.mic")
            }
        }
    }
}
```

`@Previewable` for stateful previews:
```swift
#Preview(traits: .sampleData) {
    @Previewable @Query var trips: [Trip]
    BucketListItemView(trip: trips.first)
}
```

## Takeaways
- Adopt the new `Tab` builder syntax and `.tabViewStyle(.sidebarAdaptable)` on iPadOS to get sidebar navigation and user customization (reordering, hiding) with minimal code.
- Use `MeshGradient` for visually rich hero backgrounds and onboarding imagery — the control-point grid produces naturally organic color fields unavailable from linear or radial gradients.
- The `TextRenderer` protocol unlocks entirely custom text drawing (glow, blur, per-word highlighting) without resorting to Canvas or UIKit overlays.
- All SwiftUI views are `@MainActor`-isolated in Swift 6 mode — enabling Swift 6 migration with few or no concurrency warnings in view code.

---
_Source: WWDC24 Session 10144 page (abstract, chapter summaries, code samples, and resource links)._
