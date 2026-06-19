# Inspectors in SwiftUI: Discover the Details
**WWDC23 · Session 10161** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10161/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14

## Overview
This session introduces `inspector` — a new structural SwiftUI API for displaying supplemental detail views alongside primary content. The inspector presents as a resizable trailing column on macOS and larger iPads, and automatically adapts to a resizable sheet on compact horizontal size class devices (iPhone and split-screen on smaller iPads).

Inspector bridges navigation structure APIs (like `NavigationSplitView`) and presentation APIs (like `sheet`): it builds scene scaffolding similar to navigation components, but is presented and dismissed like a sheet. This dual character determines where toolbar content appears and whether the inspector occupies the full window height or sits below the main toolbar.

The session also covers presentation customization modifiers introduced in iOS 16.4 — `presentationBackground`, `presentationBackgroundInteraction`, and `presentationDetents` — and demonstrates that these modifiers compose directly with `inspector` when it presents as a sheet on compact devices.

## Key Topics

### Inspector Fundamentals
- `inspector(isPresented:content:)` modifier — the new structural API; takes a `Bool` binding and a trailing view builder.
- Presents as a trailing sidebar column on macOS and iPadOS regular horizontal size class.
- Automatically adapts to a resizable sheet on compact horizontal size class.
- Automatically overlays in split-screen mode on larger iPads.
- Form content inside an inspector uses `.grouped` style by default — no need to set explicitly.

### Column Width Control
- `inspectorColumnWidth(min:ideal:max:)` modifier — controls the trailing column width.
- `ideal` width is used at first launch; the system persists user-resized width across launches.

### Placement Rules (Inside vs. Outside Navigation)
- **Inspector inside a navigation structure**: inspector sits under the navigation toolbar ("under toolbar" appearance). Toolbar items declared outside the inspector appear in the navigation toolbar. In compact size class, the toolbar item stays in the main toolbar.
- **Inspector outside a navigation structure**: inspector gets full window height ("full height" appearance). Toolbar items declared inside the inspector's view builder appear in an inspector-specific toolbar section. In compact size class, toolbar items travel into the sheet.
- For `NavigationSplitView`: place inspector in the detail column's view builder, or entirely outside the split view.

### Scene Phase and Dismissal
- Inspector is presented and dismissed programmatically via the `isPresented` binding.
- A toolbar button toggling the binding is the conventional control.

### Presentation Customizations (iOS 16.4+)
- `presentationBackground(_:)` — sets sheet/popover background material (e.g., `.thinMaterial`); allows underlying content to show through.
- `presentationBackgroundInteraction(_:)` — enables or disables interaction with content behind the sheet.
  - `.enabled` — removes dimming; allows full interaction behind.
  - `.enabled(upThrough: someDetent)` — dimming only at detents above the specified one.
- `presentationDetents(_:)` — defines the set of detents the sheet can snap to (e.g., `.height(200)`, `.medium`, `.large`).
- These modifiers compose with `inspector` when inspector presents as a sheet on compact devices.

## APIs & Frameworks
- `inspector(isPresented:content:)` **[NEW]** — structural modifier for trailing detail views
- `inspectorColumnWidth(min:ideal:max:)` **[NEW]** — controls inspector column width range
- `presentationBackground(_:)` **[NEW, iOS 16.4]** — sets presentation background material
- `presentationBackgroundInteraction(_:)` **[NEW, iOS 16.4]** — controls interaction with content behind a presentation
- `PresentationBackgroundInteraction.enabled` **[NEW]** — enables background interaction
- `PresentationBackgroundInteraction.enabled(upThrough:)` **[NEW]** — conditionally enables background interaction up to a detent
- `presentationDetents(_:)` **[iOS 16, updated]** — defines sheet snap detents
- `PresentationDetent.height(_:)` — custom height detent
- `PresentationDetent.medium` — medium height detent
- `PresentationDetent.large` — large/full height detent
- `NavigationStack` — navigation container; inspector inside it uses "under toolbar" style
- `NavigationSplitView` — split view; inspector goes in detail column or outside
- `Form` — container for inspector content; defaults to `.grouped` style inside inspectors
- `.formStyle(.grouped)` — explicitly sets grouped form style
- `ToolbarItem(placement:)` — positions toolbar items inside or outside inspector view builders
- `ContentUnavailableView` — shown when no item is selected in the inspector

## Code Highlights

Basic inspector adoption:
```swift
AnimalTable(state: $state)
    .inspector(isPresented: $presented) {
        AnimalInspectorForm(animal: $state.binding())
            .inspectorColumnWidth(min: 200, ideal: 300, max: 400)
            .toolbar {
                Spacer()
                Button { presented.toggle() } label: {
                    Label("Toggle Inspector", systemImage: "info.circle")
                }
            }
    }
```

Inspector inside a NavigationStack (toolbar outside inspector):
```swift
NavigationStack {
    AnimalTable(state: $state)
        .inspector(isPresented: $state.inspectorPresented) {
            AnimalInspectorForm(animal: $state.binding())
        }
        .toolbar {
            Button { state.inspectorPresented.toggle() } label: {
                Label("Toggle Inspector", systemImage: "info.circle")
            }
        }
}
```

Sheet with presentation customizations:
```swift
.sheet(item: $nibbledFruit) { fruit in
    FruitNibbleBulletin(fruit: fruit)
        .presentationBackground(.thinMaterial)
        .presentationDetents([.height(200), .medium, .large])
        .presentationBackgroundInteraction(.enabled(upThrough: .height(200)))
}
```

Same modifiers applied to inspector (compact size class):
```swift
.inspector(presented: $state.inspectorPresented) {
    AnimalInspectorForm(animal: $state.binding())
        .presentationDetents([.height(200), .medium, .large])
        .presentationBackgroundInteraction(.enabled(upThrough: .height(200)))
}
```

## Takeaways
- `inspector` is the canonical SwiftUI API for trailing detail panels (like Keynote's formatting sidebar or Shortcuts' action library) and automatically adapts to a sheet on compact devices.
- Placement of `inspector` relative to `NavigationStack`/`NavigationSplitView` determines whether toolbar content appears in the main toolbar or an inspector-specific toolbar section.
- `presentationBackground`, `presentationBackgroundInteraction`, and `presentationDetents` compose with `inspector` when it presents as a sheet, enabling fine-grained control over the compact layout.
- `inspectorColumnWidth(min:ideal:max:)` is required to make the inspector resizable; the ideal width is persisted by the system across launches.

---
_Source: WWDC23 Session 10161 page (abstract, chapter summaries, code samples, and resource links)._
