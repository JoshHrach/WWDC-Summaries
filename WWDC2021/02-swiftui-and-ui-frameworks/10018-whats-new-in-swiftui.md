# What's New in SwiftUI
**WWDC21 · Session 10018** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10018/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, watchOS 8, tvOS 15

## Overview
SwiftUI's third major release focuses on enabling deeper adoption across all platforms and real-world apps — exemplified by the new Weather app, the macOS Shortcuts app, Apple Pay purchase flow, and Help Viewer all built in SwiftUI. The session tours the biggest new features across six areas: lists and grids, beyond-list data features, graphics and visual effects, text and formatting, keyboard/focus enhancements, and buttons.

Swift Concurrency integration is a major theme: `AsyncImage`, `.refreshable`, and `.task` provide first-class async support for view lifecycle and data loading. `Table`, multi-column Core Data fetch requests, `.searchable`, materials, `Canvas`, `TimelineView`, Markdown text, format styles, `FocusState`, and a completely revamped button API round out the release.

## Key Topics

### Lists and Grids
`AsyncImage` loads remote images asynchronously with a URL, optional placeholder, custom animations, and error handling phases — available on all platforms. `.refreshable` adds pull-to-refresh to `List`; the closure is an `async` context. `.task` attaches an `async` task to a view's lifetime, automatically cancelling on disappear. `for await in` with `AsyncSequence` enables incremental list updates. `List($collection)` now passes `Binding<Element>` into the closure — no more index subscripting needed and back-deployable. New list customization: `.listRowSeparator(.hidden)`, `.listRowSeparatorTint`, `.listSectionSeparatorTint`. `.swipeActions` enables fully custom leading/trailing swipe actions on list rows with tint support. macOS: `.insetListStyle` with `.alternatesRowBackgrounds`.

### Beyond Lists: Tables, CoreData, Search
`Table` brings multi-column sortable tables to macOS (and later iPad). `TableColumn` defines columns; row selection and sort-order binding are built in. `@SectionedFetchRequest` drives multi-section lists from a single Core Data request. `.searchable(text:)` adds a platform-appropriate search field to `NavigationView`. `.onDrag` gains custom preview support. New `.importsItemProviders(_:action:)` and `.exportsItemProviders(_:_:)` for Continuity Camera and Shortcuts/Services integration.

### Graphics and Visual Effects
`Canvas` — immediate-mode drawing view similar to `drawRect`, supports gestures and environment, pairs with `.accessibilityChildren` for accessible content. `TimelineView(schedule:)` — updates a view on a schedule; supports watchOS Always On display with preloaded future states. `TimelineSchedule` — `.animation`, `.everyMinute`, `.explicit(dates:)`. SF Symbols: new `.hierarchical` and `.palette` rendering modes; automatic symbol variant selection based on context (tab bar → filled, toolbar → outlined). `.symbolRenderingMode(_:)`, `.foregroundStyle(_:_:_:)`. Materials: `.background(.ultraThinMaterial)` with vibrant foreground styles (`.primary`, `.secondary`, `.tertiary`, `.quaternary`). `.safeAreaInset(edge:content:)` — places content over a scrollable view without obscuring scroll content. `.privacySensitive()` — redacts view in widgets and watchOS Always On.

### Text, Formatting, and Localization
`Text` now renders inline Markdown. `AttributedString` (new Foundation type) backs rich text with type-safe attributes. `.textSelection(.enabled)` — enables text selection on all platforms. `dynamicTypeSize` environment value + `.dynamicTypeSize(_:)` modifier to restrict range. New Foundation format styles: `.formatted(.dateTime.hour().minute())`, list format, `PersonNameComponents` format. `TextField` now accepts format styles for typed bindings. `TextField("Prompt", value:, format:)` separates label from placeholder. Xcode 13 generates localization catalogs from `LocalizedStringKey` and new `String(localized:)`.

### Keyboard and Focus
`FocusState` property wrapper — reflects and controls focus for a Boolean or any `Hashable` type. `.focused(_:equals:)` binds a view to a specific focus state value. `.onSubmit` — action when the user submits a text field (works on groups). `.submitLabel(_:)` — sets the Return key label on software keyboards. `.toolbar` with `.keyboard` placement — adds accessory views above the software keyboard / Touch Bar.

### Buttons
`.buttonStyle(.bordered)` — new standard bordered button style on iOS. `.controlSize(_:)` — `.small`, `.regular`, `.large`. `.controlProminence(_:)` — `.standard`, `.increased`. Confirmation dialogs: `.confirmationDialog(_:isPresented:actions:)` — cross-platform replacement for action sheets. `.tint(_:)` on buttons. `Menu` with primary action + `menuIndicator(.hidden)`. Toggle with `.buttonStyle(.plain)`. `ControlGroup` — groups related controls with platform-appropriate chrome. Buttons can now be marked `.role(.destructive)` for automatic red tinting.

## APIs & Frameworks

- `AsyncImage` **[NEW]** — async remote image loading view with placeholder and phase support
- `.refreshable(action:)` **[NEW]** — pull-to-refresh on `List`; async closure
- `.task(action:)` **[NEW]** — attach async task to view lifetime; auto-cancels on disappear
- `AsyncSequence` — `for await in` loop for incremental async data
- `List($collection)` **[NEW]** — binding-based list iteration; passes `Binding<Element>` to closure
- `.listRowSeparator(_:)` **[NEW]** — `.hidden` / `.visible`
- `.listRowSeparatorTint(_:)` **[NEW]** — custom separator color per row
- `.listSectionSeparatorTint(_:)` **[NEW]** — custom section separator color
- `.swipeActions(edge:allowsFullSwipe:content:)` **[NEW]** — custom swipe actions on list rows
- `Table` **[NEW]** (macOS/iPadOS) — multi-column sortable data table
- `TableColumn` **[NEW]** — defines a table column with key path or `ViewBuilder`
- `@SectionedFetchRequest` **[NEW]** — sectioned Core Data fetch with `SectionIdentifier`
- `.searchable(text:placement:prompt:)` **[NEW]** — adds search field to `NavigationView`
- `.importsItemProviders(_:action:)` **[NEW]** — accept drops/Continuity Camera imports
- `.exportsItemProviders(_:_:)` **[NEW]** — expose data to Services/Shortcuts
- `ImportFromDevicesCommands()` **[NEW]** — Continuity Camera menu items
- `Canvas` **[NEW]** — immediate-mode drawing view; `GraphicsContext`, `Symbol`, `Path`
- `TimelineView(_:content:)` **[NEW]** — schedule-driven view updates
- `TimelineSchedule` **[NEW]** — `.animation`, `.everyMinute`, `.explicit(dates:)`
- `.symbolRenderingMode(_:)` **[NEW]** — `.monochrome`, `.hierarchical`, `.palette`, `.multicolor`
- `.foregroundStyle(_:_:_:)` **[NEW]** — up to three layered foreground styles
- `.background(.ultraThinMaterial)` **[NEW]** — system material backgrounds with vibrancy
- `.safeAreaInset(edge:content:)` **[NEW]** — inset content over a scroll view
- `.privacySensitive()` **[NEW]** — redacts in widgets (locked screen) and watchOS Always On
- `Text` Markdown support **[NEW]** — inline `**bold**`, `_italic_`, `[links](url)`, `` `code` ``
- `AttributedString` **[NEW]** (Foundation) — type-safe attributed string
- `.textSelection(_:)` **[NEW]** — enables text copy on iOS/macOS
- `.dynamicTypeSize(_:)` **[NEW]** — restricts Dynamic Type range
- `TextField(_:value:format:)` **[NEW]** — format-style typed text field
- `.onSubmit(of:_:)` **[NEW]** — action on text field submission
- `.submitLabel(_:)` **[NEW]** — Return key label (`.done`, `.go`, `.search`, etc.)
- `.toolbar { } with .keyboard placement` **[NEW]** — above-keyboard / Touch Bar toolbar
- `FocusState` **[NEW]** — property wrapper to read/drive accessibility and keyboard focus
- `.focused(_:equals:)` **[NEW]** — binds view to a `FocusState` value
- `.buttonStyle(.bordered)` **[NEW]** — standard bordered button on iOS
- `.controlSize(_:)` **[NEW]** — `.small`, `.regular`, `.large`, `.extraLarge`
- `.controlProminence(_:)` **[NEW]** — `.standard`, `.increased`
- `.confirmationDialog(_:isPresented:actions:)` **[NEW]** — cross-platform action sheet/alert
- `Button(role:action:label:)` **[NEW]** — `.destructive` role for red tint
- `Menu(primaryAction:content:label:)` **[NEW]** — menu with tap-primary-action, long-press menu
- `.menuIndicator(_:)` **[NEW]** — hide/show menu indicator chevron
- `ControlGroup` **[NEW]** — groups controls with visual affordance
- `Toggle` with `.buttonStyle` **[NEW]** — toggle displayed as a button

## Code Highlights

AsyncImage with phases:
```swift
AsyncImage(url: photo.url, transaction: .init(animation: .spring())) { phase in
    switch phase {
    case .success(let image): image.resizable().aspectRatio(contentMode: .fill)
    case .failure(let error): ErrorView(error)
    case .empty: Color.gray.opacity(0.2)
    @unknown default: EmptyView()
    }
}
```

Task modifier with AsyncSequence:
```swift
.task {
    for await candidate in photoStore.newestCandidates {
        photoStore.photos.insert(candidate, at: 0)
    }
}
```

FocusState driving focus programmatically:
```swift
enum Field { case name, email }
@FocusState private var focusedField: Field?

TextField("Name", text: $name).focused($focusedField, equals: .name)
Button("Next") { focusedField = .email }
```

TimelineView for watchOS Always On:
```swift
TimelineView(.everyMinute) { context in
    ClockFace(date: context.date)
        .privacySensitive()
}
```

## Takeaways

- Swift Concurrency is fully integrated: `AsyncImage`, `.task`, `.refreshable`, and `for await in AsyncSequence` cover the most common async UI patterns with minimal boilerplate.
- `Table`, `.searchable`, and `@SectionedFetchRequest` dramatically reduce code for data-rich macOS and iPadOS apps.
- `Canvas` + `TimelineView` + materials give SwiftUI parity with UIKit/AppKit for complex, animated, and schedule-driven graphics.
- `FocusState` and the new keyboard/submit APIs close the remaining gap for sophisticated form and text-editing experiences on all platforms.

---
_Source: WWDC21 Session 10018 page (abstract, chapter summaries, code samples, and resource links)._
