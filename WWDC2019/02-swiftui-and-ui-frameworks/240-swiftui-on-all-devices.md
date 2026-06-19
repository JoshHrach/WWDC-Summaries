# SwiftUI On All Devices
**WWDC19 · Session 240** · [Watch](https://developer.apple.com/videos/play/wwdc2019/240/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15, watchOS 6, tvOS 13

## Overview
SwiftUI is the first UI framework that runs on every Apple device — iOS, iPadOS, macOS, tvOS, and watchOS — using a single, shared skill set and toolset. This session explores what "learn once, apply anywhere" means in practice: the same composable views, layout system, and data binding concepts work on every platform, while platform-specific idioms (focus/remote on tvOS, multi-windowing on macOS, Digital Crown on watchOS) are expressed through consistent modifier and API patterns.

The session walks through the Landmarks app — Apple's SwiftUI tutorial project — being adapted for Apple TV, macOS, and Apple Watch, with concrete examples of code sharing strategies and platform-specific customization.

## Key Topics

### Single Framework, Multiple Platforms
- SwiftUI replaces UIKit (iOS/iPadOS/tvOS), AppKit (macOS), and WatchKit (watchOS) as the common UI layer.
- Common elements — `Toggle`, stacks, spacers, `List`, `Picker` — share a single API across platforms and automatically adapt appearance to each platform.
- Philosophy: "learn once, apply anywhere" rather than "write once, run anywhere" — shared skill set, strategic code sharing.

### tvOS: 10-Foot Experience Design
- Focus on rich, immersive content (beautiful imagery, video) over task-focused UI.
- Use `TabView` for top-level navigation; on tvOS, `NavigationView` should be the root with `TabView` as its content (opposite of iOS) so the tab bar disappears when navigating deeper.
- `ScrollView` with nested `HStack` ("horizontal shelves") provides natural browsing navigation for tvOS vs. vertical lists.
- `.focusable()` modifier — makes custom views participiate in tvOS focus engine. **[NEW]**
- `.onPlayPauseCommand {}` — responds to Siri remote play/pause. **[NEW]**
- `.onExitCommand {}` — responds to Siri remote menu button. **[NEW]**
- Eliminate: lengthy text content, dense sorting/filtering controls, geofenced notifications (poor fit for TV).

### macOS: High Information Density and Mac Idioms
- SwiftUI automatically adjusts control spacing/padding to macOS conventions.
- `.controlSize(.small)` / `.controlSize(.mini)` — smaller control sizes available on macOS. **[NEW]**
- **Multi-window support:** Use AppKit `NSWindowController` with `NSHostingController` to wrap SwiftUI views in windows; add double-click `onTapGesture(count: 2)` to open detail windows.
- **Keyboard shortcuts:** Use `.onCommand(_:perform:)` modifier bound to AppKit menu items/selectors to respond to keyboard shortcuts. **[NEW]**
- **Touch Bar:** Declare Touch Bar content with `TouchBar { ... }` and attach via `.touchBar(_:)` modifier. **[NEW]**
- Generic list views allow sharing list logic while swapping row types per platform without `#if os()` conditionals.

### watchOS: Digital Crown and At-a-Glance Info
- SwiftUI is the first truly native UI framework for watchOS.
- `ScrollView` wraps content that exceeds the watch screen bounds.
- **Digital Crown API** **[NEW]** — `.digitalCrownRotation($value)` modifier gives precise control over crown interaction and haptics; enables crown input in entirely new ways.
- Aim for critical actions/information within 2–3 taps.
- Beyond app UI: complications, Siri shortcuts, and notifications are equally important for watchOS experiences.

### Code Sharing Strategy
- Share views that represent pure data/content (row cells, detail cards) across platforms without modification.
- Wrap platform-specific composition (navigation structure, row type selection) in platform-specific container views that reference the shared views.
- Use generics to make shared list/container views accept different row types per platform without `#if` blocks.
- Avoid: mechanically shrinking iOS UI to other platforms; one-size-fits-all designs.

## APIs & Frameworks

### SwiftUI — tvOS Specific **[NEW]**
- `TabView` — top-level navigation for tvOS
- `.focusable(_:onFocusChange:)` — custom view focus participation **[NEW]**
- `.onPlayPauseCommand(perform:)` — Siri remote play/pause **[NEW]**
- `.onExitCommand(perform:)` — Siri remote menu/back button **[NEW]**

### SwiftUI — macOS Specific **[NEW]**
- `.controlSize(_:)` — `.regular`, `.small`, `.mini` control sizes **[NEW]**
- `.onCommand(_:perform:)` — keyboard shortcut/menu command binding **[NEW]**
- `TouchBar(content:)` — defines Touch Bar content **[NEW]**
- `.touchBar(_:)` — attaches a `TouchBar` to the focused view **[NEW]**
- `NSHostingController` — wraps SwiftUI view for use in AppKit windows
- `NSWindowController` — AppKit window management for multi-window

### SwiftUI — watchOS Specific **[NEW]**
- `ScrollView` — scrollable content on watch **[NEW on watchOS]**
- `.digitalCrownRotation(_:)` — binds a value to Digital Crown rotation **[NEW]**
- `.digitalCrownRotation(_:from:through:by:sensitivity:isContinuous:isHapticFeedbackEnabled:)` — full crown control with haptics **[NEW]**

### SwiftUI — Cross-Platform **[NEW]**
- `TabView` — tab navigation (adapts appearance per platform)
- `NavigationView` — navigation stack
- `List` — data-driven scrollable list
- `ForEach` — view generation from collections
- `ScrollView` — scrollable container
- `HStack`, `VStack`, `ZStack` — layout stacks
- `Toggle`, `Picker`, `Button`, `Stepper` — controls (auto-adapt per platform)
- `.onTapGesture(count:perform:)` — tap gesture with count (e.g., double-click on macOS)

## Code Highlights

tvOS navigation structure (NavigationView wraps TabView):
```swift
// tvOS: NavigationView is root, TabView is content
NavigationView {
    TabView {
        ExploreView().tabItem { Text("Explore") }
        HikesView().tabItem { Text("Hikes") }
    }
}
// iOS: TabView is root, NavigationView is inside each tab
```

tvOS focusable custom view with remote button handling:
```swift
MyCardView()
    .focusable(true) { isFocused in
        self.isFocused = isFocused
    }
    .onPlayPauseCommand { togglePlayback() }
```

macOS keyboard shortcut via onCommand:
```swift
TabView(selection: $selectedTab) {
    ExploreView().tag(0)
    HikesView().tag(1)
}
.onCommand(#selector(AppDelegate.showExplore)) {
    selectedTab = 0
}
```

macOS Touch Bar:
```swift
ContentView()
    .touchBar(TouchBar {
        Button("Favorite") { toggleFavorite() }
        Divider()
        Button("Share") { share() }
    })
```

watchOS Digital Crown binding:
```swift
@State private var scrollAmount = 0.0

ScrollView {
    LandmarkDetailView()
}
.digitalCrownRotation($scrollAmount)
```

Multi-window support (macOS):
```swift
func showDetail(for landmark: Landmark) {
    let controller = NSWindowController(view: LandmarkDetail(landmark: landmark))
    controller.showWindow(nil)
}

LandmarkRow(landmark: landmark)
    .onTapGesture(count: 2) { showDetail(for: landmark) }
```

Generic list for platform-specific row types:
```swift
struct LandmarkList<Row: View>: View {
    var landmarks: [Landmark]
    var rowProvider: (Landmark) -> Row

    var body: some View {
        List(landmarks, id: \.id) { landmark in
            rowProvider(landmark)
        }
    }
}
```

## Takeaways
- SwiftUI's consistent composable API means skills learned building one platform's UI transfer directly to all others — the framework handles platform-appropriate rendering automatically.
- On tvOS, flip the `NavigationView`/`TabView` nesting order vs. iOS, and prefer horizontal shelf layouts over vertical lists.
- On macOS, use `.controlSize()`, multi-window with `NSHostingController`, `.onCommand()` for keyboard shortcuts, and `.touchBar()` for Touch Bar support.
- On watchOS, `.digitalCrownRotation()` unlocks crown-driven interactions that were never previously possible with WatchKit.

---
_Source: WWDC19 Session 240 page (abstract, chapter summaries, code samples, and resource links)._
