# Use SwiftUI with UIKit
**WWDC22 · Session 10072** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10072/)

_Platforms:_ iOS 16, iPadOS 16

## Overview
Presented by an engineer from the Health app team, this session covers four concrete patterns for integrating SwiftUI into an existing UIKit application: using `UIHostingController` with its new sizing options, bridging data with `ObservableObject`, the brand-new `UIHostingConfiguration` for building collection and table view cells entirely in SwiftUI, and managing two-way data flow when SwiftUI cells connect directly to model objects.

All examples use a Health-app heart rate widget as a running narrative, demonstrating each technique incrementally until a fully interactive, self-resizing cell with a Swift Charts line chart is complete.

## Key Topics

### UIHostingController
`UIHostingController` is a `UIViewController` that renders a SwiftUI view tree. It is composable with all UIKit view controller APIs: `present(_:animated:)`, `addChild(_:)`, embed in a container view controller, or use as a custom popover. New in iOS 16: the `sizingOptions` property controls how the hosting controller exposes its preferred/intrinsic size to UIKit Auto Layout.

### Data Flow: Manual vs. ObservableObject
Passing raw values to a SwiftUI struct requires manually reassigning `hostingController.rootView` whenever data changes. The preferred approach is to make the model class conform to `ObservableObject`, annotate mutable properties with `@Published`, and reference the object with `@ObservedObject` inside the SwiftUI view. SwiftUI then subscribes automatically and re-renders on every published change — no manual `rootView` update needed.

### UIHostingConfiguration (New in iOS 16)
`UIHostingConfiguration` is a `UIContentConfiguration` initialized with a SwiftUI `ViewBuilder`. Assigning it to a cell's `contentConfiguration` renders SwiftUI directly inside the cell, with no hosting view controller required. It supports the vast majority of SwiftUI features and works with both `UICollectionView` and `UITableView`.

Special capabilities of `UIHostingConfiguration`:
- `.margins(_:_:)` — overrides the default content insets derived from the cell's layout margins
- `.background { ... }` — hosts a SwiftUI view behind the cell content, edge-to-edge
- `.swipeActions(edge:)` — declares swipe actions on rows in list/table views
- `configurationUpdateHandler` — lets you rebuild the `UIHostingConfiguration` in response to UIKit cell state changes (selected, highlighted, etc.)

### Self-Sizing Cells
iOS 16 adds automatic self-resizing to `UICollectionView` and `UITableView` cells. When SwiftUI content inside a `UIHostingConfiguration` changes size, the cell resizes automatically without any manual invalidation. Controlled by `UICollectionView.selfSizingInvalidation` and `UITableView.selfSizingInvalidation`.

### Data Flow in Cells
When cells use `UIHostingConfiguration`, use `@ObservedObject` (not `@State`) inside the SwiftUI cell view. Published property changes flow directly from the model to the SwiftUI cell view without going through the diffable data source. The diffable data source snapshot should store only stable identifiers, not full model objects. Two-way binding is created by using `$condition.text` in a `TextField` — SwiftUI writes back to the `@Published` property on the `ObservableObject` automatically.

## APIs & Frameworks

**SwiftUI / UIKit Integration**
- `UIHostingController(rootView:)` — wraps a SwiftUI view as `UIViewController`
- `UIHostingController.rootView` — update to change content; triggers SwiftUI diff
- `UIHostingController.sizingOptions` **[NEW]** — controls automatic size updates
  - `.preferredContentSize` **[NEW]** — auto-updates `preferredContentSize` (useful for popovers)
  - `.intrinsicContentSize` **[NEW]** — auto-updates intrinsic content size constraints
- `UIHostingConfiguration` **[NEW]** — `UIContentConfiguration` built with a SwiftUI `ViewBuilder`
  - `.margins(_:_:)` **[NEW]** — sets content insets
  - `.background { ... }` **[NEW]** — sets a SwiftUI background view
  - `.swipeActions(edge:allowsFullSwipe:content:)` **[NEW]** — row swipe actions (collection/table lists)
- `UICollectionViewCell.configurationUpdateHandler` (existing) — rebuild config on state changes
- `UICollectionView.selfSizingInvalidation` **[NEW]** — controls automatic cell resizing
- `UITableView.selfSizingInvalidation` **[NEW]** — same for table views

**SwiftUI Data Flow**
- `ObservableObject` — model class protocol enabling automatic view updates
- `@Published` — property wrapper that publishes changes from a model class
- `@ObservedObject` — property wrapper in SwiftUI view referencing an external `ObservableObject`
- `Binding` / `$` prefix — creates a two-way binding to a `@Published` property

## Code Highlights

Presenting a UIHostingController as a popover with automatic sizing:
```swift
let hostingController = UIHostingController(rootView: HeartRateView(data: heartData))
hostingController.sizingOptions = .preferredContentSize
hostingController.modalPresentationStyle = .popover
self.present(hostingController, animated: true)
```

ObservableObject data bridging (model + view):
```swift
class HeartData: ObservableObject {
    @Published var beatsPerMinute: Int
    init(beatsPerMinute: Int) { self.beatsPerMinute = beatsPerMinute }
}

struct HeartRateView: View {
    @ObservedObject var data: HeartData
    var body: some View {
        Text("\(data.beatsPerMinute) BPM")
    }
}
```

Building a custom UICollectionView cell with UIHostingConfiguration and Swift Charts:
```swift
cell.contentConfiguration = UIHostingConfiguration {
    VStack(alignment: .leading) {
        HeartRateTitleView()
        Spacer()
        HStack(alignment: .bottom) {
            HeartRateBPMView()
            Spacer()
            Chart(heartRateSamples) { sample in
                LineMark(
                    x: .value("Time", sample.time),
                    y: .value("BPM", sample.beatsPerMinute)
                )
                .symbol(Circle().strokeBorder(lineWidth: 2))
                .foregroundStyle(.pink)
            }
        }
    }
}
```

Responding to UIKit cell selection state:
```swift
cell.configurationUpdateHandler = { cell, state in
    cell.contentConfiguration = UIHostingConfiguration {
        HStack {
            HealthCategoryView()
            Spacer()
            if state.isSelected {
                Image(systemName: "checkmark")
            }
        }
    }
}
```

Two-way binding for an editable cell:
```swift
class MedicalCondition: Identifiable, ObservableObject {
    let id: UUID
    @Published var text: String
}

struct MedicalConditionView: View {
    @ObservedObject var condition: MedicalCondition
    var body: some View {
        TextField("Condition", text: $condition.text)
    }
}
```

## Takeaways
- Use `UIHostingController` to embed any SwiftUI view inside UIKit; in iOS 16 set `sizingOptions` to `.preferredContentSize` so popovers and sheets size themselves to SwiftUI content automatically.
- Bridge model data with `ObservableObject` + `@ObservedObject` to eliminate manual `rootView` reassignments — SwiftUI handles subscriptions and re-renders automatically.
- `UIHostingConfiguration` is the simplest path to custom cells in iOS 16: assign it to `contentConfiguration`, write SwiftUI, and the cell auto-resizes via `selfSizingInvalidation`.
- Store only stable identifiers in diffable data source snapshots; let `@ObservedObject` inside cells handle in-place property updates directly, bypassing the data source entirely.

---
_Source: WWDC22 Session 10072 page (abstract, transcript, and code samples)._
