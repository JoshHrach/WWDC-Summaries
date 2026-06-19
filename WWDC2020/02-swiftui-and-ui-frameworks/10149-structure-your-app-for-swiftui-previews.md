# Structure your app for SwiftUI previews
**WWDC20 · Session 10149** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10149/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, watchOS 7, tvOS 14

## Overview
SwiftUI Previews are most powerful when your app's architecture is designed to support them. This session presents four structural techniques — pinning previews, managing data initialization with `@StateObject`, isolating sample data using Xcode Development Assets, and designing views around simple/generic inputs — that together make previews faster, more useful, and a direct driver of better app architecture.

The running example is a photo collage app backed by CloudKit. The session shows how to preview views across multiple files simultaneously, how to avoid unnecessary network and CloudKit initialization during previews, how to keep sample assets out of the final app binary, and how to define view inputs using protocols and generics so that a Design Time mock can replace a full CloudKit-backed model in both previews and tests.

## Key Topics

### Pinning Previews
- Pin a preview using the pin button in Xcode's canvas bottom bar — pinned previews from one file remain visible while editing a different file
- Use cases: editing a sub-view while seeing it rendered in its parent context; editing a non-Swift file (e.g., Asset catalog color) while watching the live impact on a Swift view; pulling in an "extra previews" file with specialized layouts (e.g., accessibility size matrix) for a specific editing task
- Dividers separate pinned-file previews from current-file previews in the canvas
- Use the Dark Appearance inspector + "Duplicate Preview" to quickly add a dark mode variant

### `@StateObject` and the SwiftUI App Life Cycle (NEW)
- Properties on the `@main App` type with `@StateObject` are only initialized when the app is **actually run** — not when Xcode builds a preview of a leaf view
- Without `@StateObject`, every preview for any view triggers the app struct's `init`, which may include expensive CloudKit or network initialization
- Adding `@StateObject` to a model property moves initialization from app startup to first-use, eliminating unnecessary CPU and network work during preview rendering
- Checking with the Debug Preview feature (hold the play button → Debug Preview) and the CPU/network debug gauges confirms whether expensive work is being done

### Development Assets (NEW in Xcode 12)
- Development Assets are file/folder paths listed in the target's build settings — they are included only in development builds, not in archived/distributed apps
- Add large preview-only asset catalogs (many sample images, etc.) as Development Assets so they don't inflate the shipped binary
- Development Assets also apply to Swift code files — move preview-only `struct`s, mock models, and helper code into a folder listed under Development Assets
- Multi-target projects: add a separate Development Assets path for each framework target that needs preview content

### Designing Previewable Views
Four techniques for making view inputs easier to preview:

1. **Immutable simple inputs** — replace a rich observed model (e.g., a type backed by `CKUserIdentity`) with individual `let` properties of simple types (`String`, `Color`, `Date`). Use `PersonNameComponents` instead of a `String` when locale-sensitive formatting is needed; pass a `Formatter` to `Text` for automatic locale adaptation.

2. **Binding inputs** — when a view needs to mutate a value, accept a `Binding<SimpleType>` instead of an `ObservedObject` referencing a complex model. Use `Binding.constant(_:)` in read-only previews and an intermediate `@State`-holding wrapper view in interactive previews. Pass callbacks (closures) instead of model references when a view needs to trigger an action.

3. **Generic views with protocol abstractions** — define a protocol (e.g., `CollageProtocol`) that captures exactly the data a view hierarchy needs (name, layout, slot publishers, effects). Make the view generic over a type conforming to the protocol. Create a lightweight `DesignTimeCollage` that satisfies the protocol with static data from the preview asset catalog. This lets full, deeply interactive previews run without any CloudKit connection.

4. **Environment objects** — `EnvironmentObject` inputs are supplied per preview via `.environmentObject(_:)`. Simple enum-based status types (e.g., `CloudSyncStatus`) make it trivial to preview multiple states (online, offline, last-synced time).

### On-Device Previews (New in Xcode 12 / iOS 14)
- Click the device button in the canvas to mirror any preview to a connected device
- Multiple devices can be selected simultaneously; live edits reflect on all devices instantly
- An **Xcode Previews** icon appears on the device Home Screen — tap it to return to the last preview even after disconnecting from Xcode

## APIs & Frameworks

**SwiftUI**
- `@StateObject` **[NEW]** — property wrapper; initializes model objects lazily (only when the view is first created), not during preview construction
- `@ObservedObject` — use in child views; initialization happens at the parent level
- `@EnvironmentObject` / `.environmentObject(_:)` — pass shared objects through the view hierarchy; supply values per-preview using the modifier
- `Binding.constant(_:)` — read-only binding for previews
- `@State` — local mutable state; use in intermediate wrapper views for interactive previews
- `PreviewProvider` — `static var previews: some View` protocol
- `.previewDevice(_:)` / `.previewLayout(_:)` — configure preview device and size
- `Text(_:formatter:)` — locale-aware text formatting

**Xcode Previews**
- Pin button (canvas bottom bar) — pin previews from a file while editing another file
- Duplicate Preview button — add additional preview variants
- Debug Preview (hold canvas play button) — attach debugger and view CPU/network debug gauges
- Device button — mirror preview to a connected physical device
- Development Assets (Build Settings → `DEVELOPMENT_ASSET_PATHS`) **[NEW in Xcode 12]** — paths excluded from archived builds

**Foundation / CloudKit** (referenced as examples of expensive initialization to avoid)
- `CKUserIdentity`, `CKShare` — CloudKit types used as motivating examples
- `PersonNameComponents` + `PersonNameComponentsFormatter` — structured name formatting

## Code Highlights

Avoid expensive init by using `@StateObject` in the App struct:
```swift
@main
struct CollageApp: App {
    @StateObject private var networkModel = NetworkModel()
    var body: some Scene {
        WindowGroup { RootView(model: networkModel) }
    }
}
```

Interactive preview with a state-holding wrapper view:
```swift
struct InspectorPreviewWrapper: View {
    @State var effects = SlotEffects()
    var body: some View {
        ImageInspector(effects: $effects) {
            // replacePhotoHandler — change effects to confirm the button works
            effects.saturation = 0.5
        }
    }
}
struct ImageInspector_Previews: PreviewProvider {
    static var previews: some View {
        ImageInspector(effects: .constant(SlotEffects()), replacePhotoHandler: {})
        InspectorPreviewWrapper()
    }
}
```

Generic view with protocol abstraction:
```swift
// Protocol defines exactly what the UI needs
protocol CollageProtocol {
    var name: String { get set }
    var layout: CollageLayout { get set }
    var slots: [any SlotProtocol] { get }
}
// View is generic over the protocol
struct CollageEditor<C: CollageProtocol, Picker: ImagePickerProtocol>: View { ... }
// Preview uses lightweight Design Time mock — no CloudKit required
struct CollageEditor_Previews: PreviewProvider {
    static var previews: some View {
        CollageEditor(
            collage: DesignTimeCollage(name: "My Collage", layout: .twoByTwo, slots: []),
            makePhotoPicker: { DesignTimePhotoPicker() }
        )
    }
}
```

Environment object preview:
```swift
struct CloudSyncStatusView_Previews: PreviewProvider {
    static var previews: some View {
        CloudSyncStatusView().environmentObject(CloudSyncStatus(state: .online))
        CloudSyncStatusView().environmentObject(CloudSyncStatus(state: .offline))
        CloudSyncStatusView().environmentObject(CloudSyncStatus(lastSync: "a few hours ago"))
    }
}
```

## Takeaways

- Mark model properties on your `App` struct with `@StateObject` to prevent expensive CloudKit, network, or disk initialization from running during every preview render — check with Debug Preview + debug gauges to confirm.
- Add large sample asset catalogs and preview-only Swift files to `DEVELOPMENT_ASSET_PATHS` so they are excluded from shipped app binaries; Xcode template projects create this path automatically.
- Replace rich observed-object inputs with simple `let`/`Binding` properties and protocol-constrained generics — the smaller the surface area a view requires from your model, the easier it is to create comprehensive multi-configuration preview suites.
- Use the canvas pin button to keep a parent view's preview visible while editing a child view, enabling you to see the impact of every change in context without switching files.

---
_Source: WWDC20 Session 10149 page (transcript and resource links)._
