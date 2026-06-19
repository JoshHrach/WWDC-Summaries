# Build Your First App in Swift Playgrounds
**WWDC22 · Session 110348** · [Watch](https://developer.apple.com/videos/play/wwdc2022/110348/)

_Platforms:_ iPadOS 15.2+, macOS 12.2+

## Overview
This session provides a walkthrough of the complete app-building workflow in Swift Playgrounds 4, from blank template to TestFlight submission — all without Xcode. Two engineers build a "Tea Time" app collaboratively: one on Mac and one on iPad, sharing the project via an iCloud Shared Folder. The session covers App Settings customization, building a SwiftUI interface with the code Library and inline code completion, adding a Swift Package dependency, using SwiftUI Previews and the console for debugging, and uploading directly to App Store Connect for TestFlight distribution.

The session is primarily a demonstration rather than an API reference, making it well-suited for developers new to Swift or to Swift Playgrounds. It shows that Swift Playgrounds is a fully viable development environment for production apps, not just a learning tool — the same `.swiftpm` project format shared via iCloud works seamlessly between Mac and iPad, and the submission flow handles app record creation automatically.

## Key Topics

### Swift Playgrounds App Projects
- Swift Playgrounds 4 creates `.swiftpm` Swift Package-format projects for buildable apps
- App Settings panel (top-left sidebar button): set app name, accent color, app icon (custom or placeholder), Bundle ID, capabilities, and purpose strings — all without editing Info.plist or a project file
- Templates: blank App template is the starting point for a new app from scratch
- Library (plus button in toolbar): searchable snippets for SwiftUI Views, Modifiers, SF Symbols, and colors — click to insert at cursor
- Inline code completion panel: use Return key to accept suggestions

### Adding Swift Package Dependencies
- File menu → "Add Package" — enter a package URL, fetch, and select products to add
- Added packages appear in the sidebar under "Packages"
- Example: `swift-collections` package (`https://github.com/apple/swift-collections`) for `OrderedSet<Element>` and other collection types
- After adding, `import Collections` in source files to use package types

### Previews and Debugging
- `PreviewProvider` (or `#Preview` macro) — write a preview provider at the bottom of any source file
- Page dots appear at the bottom of the preview area when a preview provider is recognized; tap the chevron to switch between app preview and view preview
- Inline console: `print()` output appears at the bottom-left of the source editor when a preview or the app runs — use to verify callback values during debugging
- Project-wide find: tap the search field at the top of the sidebar, type query, press Return to search all source files

### iCloud Collaboration
- Save project to an iCloud Shared Folder to collaborate; any collaborator can open from "Locations" in the project browser
- Changes are reflected automatically in the shared project across devices
- Mac and iPad can work on the same `.swiftpm` project interchangeably

### Capabilities & Purpose Strings
- Add capabilities in App Settings → Capabilities (e.g., microphone, camera, location)
- Purpose string added alongside each capability — Swift Playgrounds adds it to the app's `Info.plist` automatically
- No manual `Info.plist` editing required

### TestFlight Submission
- App Settings → scroll to bottom → "Upload to App Store Connect" button
- Swift Playgrounds creates an app record in App Store Connect automatically if one doesn't exist
- After upload, submit for Beta App Review in App Store Connect; install via TestFlight on any device
- No Xcode or Mac required for submission from iPad

## APIs & Frameworks

**Swift Playgrounds 4** (tool features, not runtime APIs)
- App Settings panel — name, accent color, icon, Bundle ID, capabilities, purpose strings
- Library panel — SwiftUI snippet browser
- Inline code completion panel
- View Previews (`PreviewProvider` / `#Preview`) with dedicated preview canvas
- Inline console for `print()` output
- Project-wide find via sidebar search
- "Add Package" menu item (File menu) for Swift Package Manager dependencies
- "Upload to App Store Connect" — direct TestFlight submission

**SwiftUI** (used in the demo)
- `List { ForEach(collection, id: \.self) { Text($0) } }` — data-driven list
- `TabView` — tab-bar navigation
- `Text`, `Button`, `Alert` — standard views
- `@State` — local view state
- `PreviewProvider` — view preview type

**Swift Collections Package** (third-party SPM dependency)
- `OrderedSet<Element>` — ordered set that prevents duplicate elements
- Imported as `import Collections`

## Code Highlights

ForEach-driven list from an OrderedSet:
```swift
import Collections

let teas: OrderedSet<String> = [
    "Byte's Oolong", "Golden Tippy Assam", "English Breakfast",
    "Matt P's Tea Party", "Darjeeling", "Genmaicha",
    "Jasmine Green", "Vanilla Rooibos"
]

var body: some View {
    List {
        ForEach(teas, id: \.self) { tea in
            Text(tea)
        }
    }
}
```

Preview provider for an isolated custom view:
```swift
struct TeaWheelView_Previews: PreviewProvider {
    static let items: [String] = ["Item 1", "Item 2", "Item 3", "Item 4", "Item 5"]
    static var previews: some View {
        TeaWheelView(items, id: \.self) { selectedItem in
            print(selectedItem) // Debug: verify callback value in console
        }
        .padding()
    }
}
```

Using a custom view's action closure:
```swift
TeaWheelView(dataSource.teas) { tea in
    lastPickedTea = tea
    showPickAlert = true
}
```

## Takeaways
- Swift Playgrounds 4 is a fully capable app-development environment — projects use the standard `.swiftpm` package format and can be shared, edited, and submitted to TestFlight entirely from iPad without Xcode.
- App Settings handles all project configuration (name, icon, Bundle ID, capabilities, purpose strings) through a UI, making `.swiftpm` projects accessible without a traditional Xcode project file.
- Use `PreviewProvider` for isolated component debugging: pair with `print()` statements and watch the inline console to verify callback values and logic without running the full app.
- "Upload to App Store Connect" in App Settings automates app record creation and binary upload, eliminating the manual Xcode Archive & Distribute workflow for TestFlight distribution.

---
_Source: WWDC22 Session 110348 page (abstract, transcript, and code samples)._
