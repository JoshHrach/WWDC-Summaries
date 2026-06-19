# Prototype with Xcode Playgrounds
**WWDC23 · Session 10250** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10250/)

_Platforms:_ iOS 17, macOS Sonoma 14, Xcode 15

## Overview
This session demonstrates how Xcode Playgrounds accelerate feature development by eliminating the repetitive rebuild-relaunch cycle. Using a practical wildlife photography app as a running example, the presenter shows how to add a playground to an Xcode project, import app types automatically, inspect complex data models and UI snapshots, prototype a feature using an unfamiliar Swift package, and validate UI with the playground's Live View — all before writing a single line in the actual project.

Key Xcode 15 improvements to Playgrounds include a new split-view result inspector (showing object structure alongside a visual preview), type information labels with detailed tooltips in inline results, improved MapKit and CoreLocation result previews, and source-code highlighting that shows which expression produced which result.

## Key Topics

### Adding a Playground to a Project or Package
Playgrounds added to an Xcode project automatically have "Build Active Scheme" and "Import App Types" enabled. This means the app module is imported automatically so project types are usable without extra import statements.

### Execution Modes
- **Automatic Run** — re-executes the full playground on every edit pause; ideal for tight iteration loops
- **Manual Run** — developer controls exactly which lines run using gutter controls; essential when code involves expensive network requests or side effects

The execution mode toggle lives in the long-press menu of the run button on the bottom bar. In manual mode, gutter indicators show which lines would re-execute without needing to run the entire file.

### Inline Result Inspector (Xcode 15 Improvements)
- New **split-view layout** shows object structure tree alongside a visual preview (image, map, UI snapshot, etc.)
- **Type information labels** on each row show a short type summary; tooltips reveal full module and definition site
- **Source-code highlighting** — interacting with an inline result highlights the expression that produced it
- **Value History mode** — shows the result of a single expression across multiple executions (useful for seeing how a UI evolves in a loop)

### CustomStringConvertible for Better Array Inspection
By default, custom types show only their index in array rows. Conforming to `CustomStringConvertible` and adding a meaningful `description` makes both inline results and the debugger more useful — so the conformance should live in the main source files, not the playground.

### CustomPlaygroundDisplayConvertible for Rich Previews
Conforming to `CustomPlaygroundDisplayConvertible` and returning a visual object from `playgroundDescription` (e.g., a `UIImage`) enables the split-view preview in the inline result. Since this only affects playground display, it can be added in the playground's `Sources` directory.

### Live View for Interactive UI Prototyping
Import `PlaygroundSupport` and assign a view or view controller to `PlaygroundPage.current.liveView` to display a fully interactive, simulator-like preview alongside the editor. Essential for complex UI like map views that can't be inspected from a static snapshot.

### Prototyping with Unfamiliar Packages
Packages that include playground-based documentation let clients try the API immediately. In the session, the `BirdSightings` package ships a playground showing `SightingsProvider.fetchSightings(of:around:)` usage, which the developer runs before adopting the API.

## APIs & Frameworks

- **Xcode Playgrounds** — primary tool
  - "Build Active Scheme" setting — builds target before playground execution
  - "Import App Types" setting — auto-imports app module
  - Automatic Run / Manual Run execution modes
  - Inline result toggle / Eye icon (quick-look)
  - Split-view result inspector **[NEW in Xcode 15]**
  - Type information label + tooltip **[NEW in Xcode 15]**
  - Value History mode
  - Source-code highlighting on result interaction **[NEW in Xcode 15]**
  - Gutter execution indicators (manual mode)
  - Playground `Sources` directory — shared source visible to the whole playground
- `PlaygroundSupport` framework
  - `PlaygroundPage.current.liveView` — sets the interactive Live View
- **`CustomStringConvertible`** protocol — `description: String` — improves inline result labels and debugger output
- **`CustomPlaygroundDisplayConvertible`** protocol — `playgroundDescription: Any` — provides a rich visual representation in the Playground result inspector
- Improved result previews for **MapKit** types **[NEW in Xcode 15]**
- Improved result previews for **CoreLocation** types (e.g., `CLLocationCoordinate2D` map preview) **[NEW in Xcode 15]**
- `CLLocationCoordinate2D` — CoreLocation coordinate type used in the demo
- `UIView` / `UIViewController` — assignable as live view

## Code Highlights

Setting up a playground live view:
```swift
import PlaygroundSupport
PlaygroundPage.current.liveView = sightingMapView
```

Conforming to `CustomPlaygroundDisplayConvertible` in the playground's Sources directory:
```swift
extension Bird: CustomPlaygroundDisplayConvertible {
    public var playgroundDescription: Any {
        return photo as Any  // explicit cast needed to preserve optional information
    }
}
```

Conforming to `CustomStringConvertible` in the main project source:
```swift
extension Bird: CustomStringConvertible {
    public var description: String {
        return "\(commonName) (\(scientificName))"
    }
}
```

Value History mode iteration (inspecting UI across loop iterations):
```swift
let checklist = ChecklistView()
for bird in owlsToFind {
    checklist.add(bird)  // select expression → Value History shows view at each step
}
```

## Takeaways

- Add a playground to your project to prototype features without rebuilding and relaunching; "Build Active Scheme" and "Import App Types" are enabled by default, giving immediate access to all project types.
- Conform custom types to `CustomStringConvertible` in the main source (benefits the debugger too) and to `CustomPlaygroundDisplayConvertible` in the playground's `Sources` directory for rich visual previews.
- Switch to Manual Run mode when making network calls or performing expensive operations — gutter indicators show exactly which lines will re-execute.
- Use `PlaygroundPage.current.liveView` for fully interactive previews of complex UI; use Value History mode to see how a view evolves across multiple loop iterations.

---
_Source: WWDC23 Session 10250 page (abstract, chapter summaries, code samples, and resource links)._
