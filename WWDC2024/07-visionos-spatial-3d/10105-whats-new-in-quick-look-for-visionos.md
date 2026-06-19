# What's New in Quick Look for visionOS
**WWDC24 · Session 10105** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10105/)

_Platforms:_ visionOS 2, iOS 18, iPadOS 18, macOS Sequoia 15

## Overview
Quick Look on visionOS gains a brand-new `PreviewApplication` API that gives apps direct, programmatic control over the windowed Quick Look experience — opening individual files or collections, customizing display names and editing options, and managing session lifecycle. Previously, windowed Quick Look was accessible only through drag and drop; the new API requires just a few lines of Swift code and is built on SwiftUI and Swift Concurrency.

On the content side, Quick Look in visionOS 2 adds surface snapping for 3D models (automatic snap to horizontal surfaces like tables and floors) and configurations/variants — the ability to switch between different visual states of the same 3D asset from within Quick Look without requiring separate files.

## Key Topics

### PreviewApplication API
`PreviewApplication.open(urls:)` is a new static async method that opens one or more files in windowed Quick Look. Passing a single URL opens that file; passing an array opens a collection view with navigation arrows between all items. An optional `selected` parameter sets the initially focused item.

For more control, `PreviewItem` wraps a URL with a `displayName` (shown as the window title) and an `editingMode` that can be set to `.disabled` to hide Quick Look's default trim/edit menu items.

`PreviewApplication.open(items:)` returns a `Session` object. The session exposes an `events` async data stream that emits when a preview opens or closes, enabling apps to track which files are currently being previewed and update their UI accordingly (e.g., showing an indicator icon on thumbnails with open previews). Quick Look ensures only one preview exists per file — opening the same file again brings the existing preview to the front.

### In-App Quick Look (Existing)
The in-app style integrates Quick Look as a full-screen cover inside the host application. The windowed style opens 3D models in a detached volume alongside the app, enabling multitasking — viewing a spatial video while interacting with the rest of the app.

### Surface Snapping for 3D Models
All 3D models in windowed Quick Look now automatically snap to horizontal surfaces (tables, floors). Dragging the window bar near a surface triggers the snap; a sound signals the action. Pitch rotation is disabled while snapped to prevent the model from clipping through the surface. No code changes are needed — surface snapping is enabled automatically. Authors should ensure their 3D model's bottom face sits at the origin for best results.

### Configurations / Variants in USDZ and Reality Files
Quick Look now surfaces a configuration switcher when a USDZ or Reality file contains variant sets. Users tap a menu button in Quick Look to select a variant (e.g., different iPhone colors). Configurations are synced across all participants in a FaceTime SharePlay session and work on iOS and macOS as well. When authoring variants, all variant sets must be defined on the default prim so Quick Look can discover and display them correctly. Variant names use underscores in place of spaces (Quick Look converts automatically on display). Avoid heavy textures or complex geometry per-variant to maintain fast load times.

## APIs & Frameworks

**QuickLook**
- `PreviewApplication.open(urls:selected:)` **[NEW]** — open one or more files in windowed Quick Look
- `PreviewApplication.open(items:selected:)` **[NEW]** — open `PreviewItem` values with custom display names and editing options
- `PreviewItem` **[NEW]** — wraps a URL with `displayName` and `editingMode`
  - `displayName: String` — title shown in the windowed Quick Look menu bar
  - `editingMode: PreviewItem.EditingMode` — e.g., `.disabled` to hide edit options
- `PreviewApplication.Session` **[NEW]** — returned by `open`, manages the lifecycle of a windowed preview
  - `events: AsyncStream<PreviewApplication.Session.Event>` **[NEW]** — async stream of open/close events
- Surface snapping for 3D models **[NEW]** — automatic, no API required; requires model origin at bottom face
- USDZ variant/configuration display **[NEW]** — automatic when variant sets present on default prim
- Reality file configuration display **[NEW]**
- SharePlay configuration sync for visionOS **[NEW]** — variant selections synced between FaceTime participants

**USDZ / USD**
- `variantSet` — USD schema construct for defining configurations (authoring-time, no runtime API)
- `variants` — variant selection on the default prim's Xform

**SwiftUI / Swift Concurrency**
- `PreviewApplication` API is designed for use with `async`/`await` and `Task`

## Code Highlights

Open a single file in windowed Quick Look:
```swift
// onTapGesture on a thumbnail view
await PreviewApplication.open(urls: [selectedURL])
```

Open all files in a collection view, focused on the tapped file:
```swift
await PreviewApplication.open(urls: allURLs, selected: selectedURL)
```

Open with custom title and editing disabled:
```swift
let item = PreviewItem(url: selectedURL, displayName: entry.name, editingMode: .disabled)
await PreviewApplication.open(items: [item], selected: item)
```

Track session lifecycle to show/hide a "currently previewing" indicator:
```swift
@State private var isPreviewing = false

func observeSession(_ session: PreviewApplication.Session) async {
    for await event in session.events {
        switch event {
        case .opened: isPreviewing = true
        case .closed: isPreviewing = false
        }
    }
}
```

USDZ variant authoring for configurations (USD):
```
#usda 1.0
(defaultPrim = "iPhone")

def Xform "iPhone" (
    variants = { string Color = "Black_Titanium" }
    prepend variantSets = ["Color"]
) {
    variantSet "Color" = {
        "Black_Titanium" { }
        "Blue_Titanium" { }
        "Natural_Titanium" { }
        "White_Titanium" { }
    }
}
```

## Takeaways
- Replace drag-and-drop Quick Look launch with `PreviewApplication.open(urls:)` — a few lines of Swift that unlock collection views, editing control, and session lifecycle management.
- Use `PreviewItem.editingMode = .disabled` when previewing content users should not trim or modify (e.g., a read-only media library).
- Observe `PreviewApplication.Session.events` to reflect Quick Look state in your app UI — avoid assuming a preview is still open after the user dismisses it.
- Embed variant sets in USDZ/Reality files to let users explore product configurations (colors, materials) within a single Quick Look preview; no extra files needed and the feature works on iOS, iPadOS, macOS, and visionOS with FaceTime sync.

---
_Source: WWDC24 Session 10105 page (abstract, chapter summaries, code samples, and resource links)._
