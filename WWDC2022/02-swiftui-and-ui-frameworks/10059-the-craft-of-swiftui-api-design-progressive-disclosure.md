# The Craft of SwiftUI API Design: Progressive Disclosure
**WWDC22 · Session 10059** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10059/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, watchOS 9, tvOS 16

## Overview
This session pulls back the curtain on one of SwiftUI's core design principles — progressive disclosure — and shows how it applies both to Apple's own API design and to reusable components that any developer builds. The talk is not about new APIs per se; it's about design philosophy that shapes every SwiftUI API and that developers can apply to their own abstractions.

Progressive disclosure means designing APIs so that the complexity of the call site grows with the complexity of the use case. A simple use case should produce minimal call-site code; advanced use cases unlock more powerful — but more verbose — options. This minimizes time-to-first-build-and-run, lowers learning curves, and creates tight feedback loops for rapid iteration.

Four concrete strategies are examined using existing SwiftUI APIs as case studies: considering common use cases, providing intelligent defaults, optimizing the call site, and composing primitives instead of enumerating possibilities.

## Key Topics

### Consider Common Use Cases
Identify the most common call patterns and provide convenience overloads or initializers for them. Example: `Button("Next Page") { }` versus the full `Button { } label: { }` form. The short form handles 99% of cases; the long form handles the rest. This label-plus-action pattern appears throughout SwiftUI (`Menu`, `Toggle`, `NavigationLink`, etc.).

### Provide Intelligent Defaults
Every piece of information not specified at the call site should have a sensible default. `Text("Hello")` demonstrates this: localization, dark mode support, dynamic type scaling, and inter-element spacing are all automatic. `toolbar { }` places items per platform convention by default; explicit `ToolbarItemGroup(placement:)` is available when needed.

### Optimize the Call Site
Each character at a call site should have a clear purpose. The session traces `Table` through multiple simplification steps:
1. Replace explicit `ForEach { TableRow }` rows with a direct collection parameter (handled internally).
2. Omit the view builder on `TableColumn` when the value key path points to a `String` (text is implied).
3. Remove `sortOrder` entirely when sorting isn't needed — a no-sort overload exists.
The result is `Table(collection) { TableColumn(...) }` for the simplest case.

### Compose, Don't Enumerate
Avoid adding enum cases for every conceivable behavior variation. Instead, expose composable primitives. Example: `HStack` doesn't have a `.leading` / `.centered` / `.trailing` arrangement enum; it exposes `Spacer()` as a composable primitive that can express all of those layouts — and many more — without any enum cases.

## APIs & Frameworks

This session uses existing SwiftUI APIs as illustrations; no new APIs are introduced. Key APIs referenced:

**SwiftUI (existing)**
- `Button("Label") { action }` — common-case initializer (string label)
- `Button { action } label: { View }` — full-form initializer (arbitrary label view)
- `Text(_:)` — intelligent defaults: localization, dark mode, dynamic type, spacing
- `.toolbar { }` — platform-default placement
- `ToolbarItemGroup(placement:)` — explicit placement override
- `Table(_:)` / `Table(_:sortOrder:)` — collection convenience and sort-optional overloads
- `TableColumn(_:value:)` — string key path convenience (no view builder required)
- `TableColumn(_:value:content:)` — full-form with custom cell view
- `TableRow(_:)` — explicit row wrapper
- `HStack` / `VStack` — layout composites
- `Spacer()` — composable layout primitive for spacing

## Code Highlights

Table API progressive disclosure — from complex to simple:
```swift
// Most verbose (advanced case)
Table(sortOrder: $sortOrder) {
    TableColumn("Title", value: \Book.title) { book in Text(book.title).bold() }
    TableColumn("Author", value: \Book.author) { book in Text(book.author).italic() }
} rows: {
    Section("Favorites") { ForEach(favorites) { TableRow($0) } }
    Section("Currently Reading") { ForEach(currentlyReading) { TableRow($0) } }
}

// Simple case — no sorting, no custom cells, collection passed directly
Table(currentlyReading) {
    TableColumn("Title", value: \.title)
    TableColumn("Author", value: \.author)
}
```

`Spacer`-based composition replaces a hypothetical arrangement enum:
```swift
// Centered — no enum needed
HStack { Spacer(); Box(); Box(); Box(); Spacer() }

// Evenly spaced — no enum needed
HStack { Spacer(); Box(); Spacer(); Box(); Spacer(); Box(); Spacer() }
```

## Takeaways
- Progressive disclosure = call-site complexity scales with use-case complexity; start simple, reveal power gradually.
- Every API should have a shortest valid form that covers the most common case, with additional parameters or overloads unlocking more control.
- Prefer composable primitives (like `Spacer`) over exhaustive enumerations of possibilities — composition scales to use cases you haven't anticipated.
- As a developer building reusable components, apply the same four strategies: common-case first, intelligent defaults, minimal call site, composability.

---
_Source: WWDC22 Session 10059 page (abstract, chapter summaries, code samples, and resource links)._
