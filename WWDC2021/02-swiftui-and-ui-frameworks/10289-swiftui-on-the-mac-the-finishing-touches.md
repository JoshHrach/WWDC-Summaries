# SwiftUI on the Mac: The Finishing Touches
**WWDC21 · Session 10289** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10289/)

_Platforms:_ macOS Monterey 12

## Overview
Part two of the "SwiftUI on the Mac" code-along series continues building the gardening app from Session 10062. This session focuses on polish, customization, and flexible workflows: automatically reacting to system-wide settings (Dark Mode, accent color, sidebar icon size), building a Settings window with per-tab preferences, adding drag and drop to a `Table`, exporting data with `fileExporter`, and integrating Continuity Camera for image import.

The key theme is that a great Mac app adapts to individual user preferences rather than imposing a fixed experience. SwiftUI makes many of these adaptations automatic, while providing APIs for the cases that require explicit support.

## Key Topics

### Automatic System Adaptations
SwiftUI apps automatically respond to Dark Mode, accent color changes, and sidebar icon size set in System Preferences without any developer code. Setting an `AccentColor` asset in the asset catalog customizes buttons, selection highlighting, and sidebar glyphs app-wide.

### Settings Scene and AppStorage
The `Settings` scene adds a menu item and keyboard shortcut (Cmd-,) and opens a settings window. A `TabView` with `tabItem` provides the standard macOS tabbed settings UI. `@AppStorage` persists picker selections (and other lightweight values) to `UserDefaults` automatically, surviving app restarts and OS updates.

### Drag and Drop in Table
A `Table` row builder with `ForEach` and `TableRow` is required to support per-row customization. The `.itemProvider` modifier on `TableRow` makes rows draggable. The `.onInsert(of:perform:)` modifier on the `Table` makes it a drop destination, accepting `NSItemProvider`-based content types.

### File Export with fileExporter
The `.fileExporter(isPresented:document:contentType:onCompletion:)` modifier presents a save panel and exports a document conforming to `FileDocument` or `ReferenceFileDocument`. A `CommandGroup` placed at the `.importExport` position adds the export menu item in the standard File menu location.

### Continuity Camera with ImportFromDevicesCommands
`ImportFromDevicesCommands()` adds the system-provided "Import from iPhone/iPad" menu item. The `.importsItemProviders(_:action:)` modifier on a view makes it the destination for imported images, receiving `NSItemProvider` objects when the user captures or scans with a nearby device.

## APIs & Frameworks

- `Settings` scene **[NEW]** — adds a Settings window with Cmd-comma keyboard shortcut
- `@AppStorage` — persists values to `UserDefaults`; takes a string key
- `TabView` with `tabItem` — standard macOS settings window tab pattern
- `Table` row builder / `TableRow` — required for per-row drag-and-drop customization
- `.itemProvider` modifier on `TableRow` **[NEW]** — makes a table row a drag source
- `.onInsert(of:perform:)` **[NEW]** — makes a `Table` accept drops; receives index and `[NSItemProvider]`
- `.fileExporter(isPresented:document:contentType:onCompletion:)` **[NEW]** — presents a save panel for `FileDocument` export
- `FileDocument` / `ReferenceFileDocument` — protocols for exportable document types
- `CommandGroup` — inserts menu items at system-defined positions (e.g., `.importExport`)
- `ImportExportCommands` — custom `Commands` type for import/export menu items
- `ImportFromDevicesCommands()` **[NEW]** — system-provided Continuity Camera menu items
- `.importsItemProviders(_:action:)` **[NEW]** — registers a view as a Continuity Camera import destination
- `NSItemProvider` — carries dragged/dropped/imported data
- `AccentColor` asset — sets the app-wide accent color
- `@SceneStorage` — (from part one) persists window-level UI state
- `WindowGroup` — multi-window app entry point
- `ForEach` in `Table` row builder — required to add per-row modifiers

## Code Highlights

Adding a Settings window scene:
```swift
@main
struct GardenApp: App {
    var body: some Scene {
        WindowGroup { ContentView() }
        Settings { SettingsView(store: store) }
    }
}
```

Persisting a settings picker selection with AppStorage:
```swift
@AppStorage("defaultGarden") private var selection: Garden.ID?

TabView {
    GeneralSettings()
        .tabItem { Label("General", systemImage: "gear") }
    ViewingSettings()
        .tabItem { Label("Viewing", systemImage: "eyeglasses") }
}
```

Table with drag and drop:
```swift
Table(sortOrder: $sortOrder) {
    // columns...
} rows: {
    ForEach(garden.plants) { plant in
        TableRow(plant)
            .itemProvider { plant.itemProvider }
    }
}
.onInsert(of: [Plant.draggableType]) { index, providers in
    Plant.fromItemProviders(providers) { plants in
        garden.plants.insert(contentsOf: plants, at: index)
    }
}
```

Enabling Continuity Camera import:
```swift
GardenGalleryView(...)
    .importsItemProviders(selection.isEmpty ? [] : Plant.importImageTypes) { providers in
        Plant.importImageFromProviders(providers) { url in
            for id in selection {
                store.garden(for: id)?.imageURL = url
            }
        }
        return true
    }
```

## Takeaways

- SwiftUI apps inherit Dark Mode, accent color, and density preferences automatically — set an `AccentColor` asset to complete the picture.
- `Settings` scene + `@AppStorage` is the canonical pattern for macOS user preferences; it wires up the menu item, keyboard shortcut, and persistence in a few lines.
- Per-row drag and drop in `Table` requires the row-builder form with `TableRow`; `.itemProvider` and `.onInsert` handle the rest.
- `ImportFromDevicesCommands` + `.importsItemProviders` adds full Continuity Camera support with minimal code.

---
_Source: WWDC21 Session 10289 page (abstract, chapter summaries, code samples, and resource links)._
