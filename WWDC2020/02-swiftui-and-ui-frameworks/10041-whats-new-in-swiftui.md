# What's New in SwiftUI
**WWDC20 · Session 10041** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10041/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, tvOS 14, watchOS 7

## Overview
SwiftUI's second major release introduced the ability to build entire apps using only SwiftUI, replacing the requirement to embed views inside UIKit, AppKit, or WatchKit app delegates. The new `App` protocol, `Scene` hierarchy, `WindowGroup`, `DocumentGroup`, and `Settings` scene types give developers a purely declarative path from app entry point to window management. Widgets and watchOS complications are now built exclusively with SwiftUI.

Collections gained lazy-loading grid layouts (`LazyVGrid`, `LazyHGrid`) and hierarchical outline support in `List`, while a new unified `toolbar` modifier replaces platform-specific toolbar APIs. System integration expanded to include URL opening, drag and drop with Uniform Type Identifiers, and a native Sign In with Apple button. Visual effects like `matchedGeometryEffect` and `ContainerRelativeShape` provided new animation and styling primitives.

## Key Topics

### App and Scene APIs
SwiftUI can now serve as a complete app lifecycle. A struct conforming to `App` defines the app's `body` as one or more `Scene` values. Scene types:
- `WindowGroup` — manages one or more windows; on iPadOS/macOS supports multiple instances side-by-side; adds a New Window menu command on macOS
- `Settings` — macOS-only; adds a Preferences menu command and standard preferences window style
- `DocumentGroup` — document-based apps; manages open/edit/save; presents document browser on iOS; multi-window on macOS
- `commands` modifier — adds menus/menu items to the macOS main menu with `CommandMenu`
- `keyboardShortcut` modifier — assigns keyboard shortcuts to buttons and commands
- Launch Screen configuration via Info.plist key (alternative to storyboard launch screens)

### Widgets and Complications
- Widgets built exclusively with SwiftUI; conform to `Widget` protocol with `WidgetConfiguration`
- `StaticConfiguration` and intent-configured widgets via WidgetKit
- watchOS complications now support full-color SwiftUI views; complication modifiers: `complicationForeground()`, `complicationChartFont()`

### Lists and Collections
- `List(data, children:)` — hierarchical outlines with automatic expand/collapse; `children` key path points to optional child array
- `LazyVGrid(columns:)` — lazy vertical grid; `GridItem(.adaptive(minimum:))` for flexible columns; `GridItem()` for fixed-count
- `LazyHGrid(rows:)` — lazy horizontal grid
- `LazyVStack`, `LazyHStack` — lazy versions of existing stacks for custom scrollable layouts
- Supports `switch` statements in view builders for alternating layouts (new Swift 5.3 feature)

### Toolbars and Controls
- `toolbar` modifier — unified toolbar API for all platforms; items placed automatically or via `ToolbarItem(placement:)`
- `ToolbarItemPlacement` semantic placements **[NEW]**: `.primaryAction`, `.confirmationAction`, `.cancellationAction`, `.principal`, `.bottomBar`, `.navigationBarLeading`, `.navigationBarTrailing`
- `Label` **[NEW]** — combined title + icon view; adapts to context (toolbar shows icon only; list shows full label); dynamic type aware; SF Symbols integration
- `help(_:)` modifier **[NEW]** — tooltip text on macOS; accessibility hint on all platforms
- `keyboardShortcut(_:)` modifier **[NEW]** — available on controls shown on screen (not just commands)
- `defaultFocus` support **[NEW]** — control where focus starts on tvOS and watchOS
- `ProgressView` **[NEW]** — determinate and indeterminate progress; linear and circular styles
- `Gauge` **[NEW]** — level indicator; supports currentValueLabel, min/max value labels; watchOS circular gauge

### Visual Effects and Styling
- `matchedGeometryEffect(id:in:)` **[NEW]** — seamless view transitions when a view moves from one location to another; connects views with a shared ID within a `@Namespace`
- `@Namespace` **[NEW]** — property wrapper for namespacing matched geometry effect identifiers
- `ContainerRelativeShape` **[NEW]** — shape that adopts the concentric corner radius of the nearest containing shape (ideal for widget artwork)
- Custom fonts now scale with Dynamic Type automatically
- Images embedded in `Text` scale with Dynamic Type
- `@ScaledMetric` **[NEW]** — property wrapper that scales a numeric value with the current Dynamic Type size
- Accent color support on macOS **[NEW]**; defined in Xcode 12 asset catalog
- `listItemTint(_:)` **[NEW]** — tint color for sidebar icons per item or section
- Toggle, Button tinting via new style customizations

### System Integration
- `Link` **[NEW]** — URL-opening view; works in widgets (deep links into app)
- `openURL` environment action **[NEW]** — programmatic URL opening with optional completion
- Drag and drop with Uniform Type Identifiers framework **[NEW on iOS 14/macOS Big Sur]**
  - `UTType` — strongly-typed content type identifier; `exportedAs:`, `importedAs:` extensions
  - `FileDocument` protocol — `readableContentTypes`, `init(fileWrapper:contentType:)`, `write(to:contentType:)`
- Sign In with Apple button **[NEW]** — `SignInWithAppleButton` from `AuthenticationServices`; available on all platforms by importing `AuthenticationServices` + `SwiftUI`

## APIs & Frameworks

### SwiftUI (New Types and Modifiers)
- `App` protocol **[NEW]** — top-level app entry point
- `Scene` protocol **[NEW]** — unit of UI shown by the platform
- `WindowGroup` **[NEW]** — multi-platform window scene
- `Settings` **[NEW]** — macOS preferences scene
- `DocumentGroup` **[NEW]** — document-based scene
- `Widget` protocol **[NEW]** — WidgetKit entry point
- `WidgetConfiguration`, `StaticConfiguration` **[NEW]**
- `LazyVGrid`, `LazyHGrid` **[NEW]**
- `GridItem` **[NEW]** — column/row descriptor (`.adaptive`, `.flexible`, `.fixed`)
- `LazyVStack`, `LazyHStack` **[NEW]**
- `Label` **[NEW]**
- `ProgressView` **[NEW]**
- `Gauge` **[NEW]** (watchOS primarily)
- `Link` **[NEW]**
- `ToolbarItem` **[NEW]**, `ToolbarItemPlacement` **[NEW]**
- `CommandMenu` **[NEW]**, `CommandGroup` **[NEW]**
- `matchedGeometryEffect(id:in:)` modifier **[NEW]**
- `@Namespace` **[NEW]**
- `ContainerRelativeShape` **[NEW]**
- `@ScaledMetric` **[NEW]**
- `help(_:)` modifier **[NEW]**
- `keyboardShortcut(_:)` modifier **[NEW]**
- `listItemTint(_:)` modifier **[NEW]**
- `toolbar(_:)` modifier **[NEW]**
- `commands(_:)` modifier **[NEW]**
- `@StateObject` **[NEW]** — observed object owned by the view
- `@SceneStorage` **[NEW]** — scene-scoped persistent storage
- `@AppStorage` **[NEW]** — UserDefaults-backed storage

### Related Frameworks
- `UniformTypeIdentifiers` framework **[NEW]** — `UTType` for drag and drop and `DocumentGroup`
- `AuthenticationServices` — `SignInWithAppleButton` available via combined import with SwiftUI **[NEW]**
- `WidgetKit` **[NEW]** — widget extension framework
- AVKit, MapKit, and others now provide SwiftUI views/modifiers **[NEW]**

## Code Highlights

Minimal full SwiftUI app:
```swift
@main
struct HelloWorld: App {
    var body: some Scene {
        WindowGroup {
            Text("Hello, world!").padding()
        }
    }
}
```

Hierarchical List with children:
```swift
List(graphics, children: \.children) { graphic in
    GraphicRow(graphic)
}.listStyle(SidebarListStyle())
```

Adaptive lazy grid:
```swift
ScrollView {
    LazyVGrid(columns: [GridItem(.adaptive(minimum: 176))]) {
        ForEach(items) { item in ItemView(item: item) }
    }
}
```

Toolbar with semantic placement:
```swift
.toolbar {
    ToolbarItem(placement: .primaryAction) {
        Button(action: recordProgress) {
            Label("Record Progress", systemImage: "book.circle")
        }
    }
}
```

DocumentGroup with UTType:
```swift
DocumentGroup(newDocument: ShapeDocument()) { file in
    DocumentView(document: file.$document)
}
extension UTType {
    static let shapeEditDocument = UTType(exportedAs: "com.example.ShapeEdit.shapes")
}
```

## Takeaways

- SwiftUI can now build complete apps end-to-end using `App`, `Scene`, `WindowGroup`, `DocumentGroup`, and `Settings` — no UIKit/AppKit app delegate required.
- `LazyVGrid`/`LazyHGrid` and outline `List` fill major layout gaps that previously required custom UIKit/AppKit implementations.
- `matchedGeometryEffect` enables seamless cross-container view transitions with minimal code, while `ContainerRelativeShape` provides automatic concentric corner radius for widgets and other nested UIs.
- The new `toolbar` modifier replaces all platform-specific toolbar patterns with a single declarative API using semantic `ToolbarItemPlacement` values.

---
_Source: WWDC20 Session 10041 page (abstract, transcript, code samples, and resource links)._
