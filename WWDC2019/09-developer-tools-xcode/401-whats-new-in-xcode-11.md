# What's New in Xcode 11
**WWDC19 · Session 401** · [Watch](https://developer.apple.com/videos/play/wwdc2019/401/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15, watchOS 6, tvOS 13

## Overview
Xcode 11 is the most feature-rich Xcode release since Xcode was introduced, adding four major headline areas: deeply integrated Swift Package Manager support, a completely redesigned editor workspace with flexible splits and a new Minimap, full SwiftUI preview integration with live editing, and significant improvements to testing, simulation, and Instruments.

The session is a fast-paced overview touching every part of the tool; deeper coverage of each area is provided in companion sessions (Debugging in Xcode 11, Testing in Xcode, Getting Started with Instruments, etc.).

## Key Topics

### Editor Workspace Redesign **[NEW]**
- Editor modes (Standard, Assistant, Version) no longer apply to the entire window; each editor pane now has its own independent mode, set via a per-pane Options menu.
- Source Control history moved to the new **History Inspector** in the inspector pane — available any time for any file without occupying editor space.
- Editor splitting is now unrestricted: create any number of horizontal or vertical splits within a single window using the Add Editor button or Option+click on the button for the perpendicular direction.
- **Destination Chooser**: Option+Shift+click on a file to get an interactive overlay for choosing which editor, new split, tab, or window to open it in.
- **Focus Mode**: click the Focus button (top corner of any editor pane) to expand that editor to fill the full window; click again to return to the previous layout.
- **Code Review Mode**: similar to Focus mode but dedicated to reviewing changes.

### Source Editor Improvements
- **Minimap** **[NEW]**: bird's-eye view of the entire file with syntax coloring, marks, breakpoints, and changed-line indicators; hover to see symbol names; shows transient info like find matches.
- New syntax coloring options (e.g., declarations highlighted distinctly).
- New themes including high-contrast Light and Dark variants.
- **Improved documentation rendering**: restyled for readability; Xcode parses more documentation structure; the action popover's "Add Documentation" command fills missing parameter docs and works with multiple cursors.
- **Edit All In Scope** now fixes identifiers in documentation comments as well as code and signatures.
- **Inline Diff** **[NEW]**: click a change bar in the gutter to expand an inline diff view showing exactly what changed (live-updating as you make further edits).
- Spell checking support added.
- Nested code folding improvements.

### Code Completion Improvements
- Completions for `#available` / `#unavailable` and other compiler control statements.
- `never` completions work more reliably and in more contexts (e.g., appending to an enum array).
- Completions for function overloads.

### Swift Package Manager Integration **[NEW]**
- Swift Package Manager fully integrated into Xcode 11's core workflows.
- Project Editor gains a **Swift Packages** tab — add, remove, and manage package dependencies.
- Add a package by URL; Xcode resolves dependencies and fetches the source automatically.
- GitHub, GitLab, and Bitbucket account integration lets you browse and star personal/org repositories.
- Version rules: up-to-next-major (default), up-to-next-minor, exact, branch, or commit.
- Package source appears in the Project Navigator under a **Swift Package Dependencies** section and is fully navigable (Jump to Definition, search, etc.).
- Source Control integration, debugging, and testing all work with package code just like project code.
- See Sessions 408 and 410 for creation and adoption details.

### Source Control Additions
- **Git Stash**: stash working changes from the Source Control menu; pop from the Source Control navigator.
- **Cherry-Pick**: available from the Source Control history view or the commit context menu.
- History Inspector: per-file Source Control history in the inspector pane (works for non-text files too).

### SwiftUI Integration **[NEW]**
- **Canvas / Preview** built into each editor pane via the Options menu ("Editor in Canvas"); shows live SwiftUI previews alongside code.
- Previews are live: code changes update the preview, and direct manipulation in the canvas generates code.
- New inline library (redesigned) for browsing and dragging SF Symbols, views, and modifiers.
- Existing apps can use previews via `UIViewRepresentable` without migrating to SwiftUI.
- New SwiftUI tutorials and documentation experience.
- Works for iOS, macOS, watchOS, and tvOS.

### Storyboard / Asset Catalog Improvements
- Device bar gains a **Mac** option for customizing iPad apps on Mac (Catalyst).
- **Dark/Light toggle** in the device bar for quick visual verification without leaving the editor.
- SF Symbols fully integrated in the inspector: browse symbols, set size configuration (e.g., font-based, title size), drag into storyboards.
- Asset Catalogs now support **asset localization**: select an asset, click Localize, choose localizations.
- **Custom Symbol Assets** **[NEW]**: create your own SF-Symbol-compatible vector assets in Asset Catalogs; supports multiple weights and scales; runtime picks the right variant.
- `UIViewRepresentable` protocol enables previewing `UIView` subclasses in SwiftUI Canvas.

### Testing: Test Plans **[NEW]**
- **Test plans** define a reusable, shareable set of tests with multiple configurations (arguments, environment variables, sanitizers, localizations, etc.).
- Running a test plan executes all tests in all configurations in one action.
- Test plans are shared across schemes and work with Xcode Server for parallelized multi-device/simulator runs.
- Supports iPad apps on Mac and SwiftUI apps.
- See Session 413 for full details.

### Simulator Improvements
- **Metal-backed Simulator** **[NEW]**: the simulator is now built on Metal; Metal apps can run in the simulator; UIKit rendering gets Metal acceleration (60fps, CPU use reduced by up to 90%).
- **Standalone watchOS apps**: deploy and run watchOS apps directly on the watch simulator (no iPhone simulator required).
- Warm boot time up to 2x faster.
- **Device Conditions** (via the Devices window): Network Link Conditioner and Thermal State Conditioner can be applied to a connected device directly from Xcode without installing a separate profile.

### Instruments Improvements
- **Hierarchical tracks** for OS Signpost categories; each category gets its own track automatically; tracks can be pinned for correlation with CPU or other tracks.
- New **SwiftUI Instruments template**: measures time spent in `body` computations.
- Completely rewritten **Metal System Trace template**: hierarchical, up to 10x faster.
- See Session 411 for details.

### Environment Overrides (Debug bar) **[NEW]**
- Runtime control panel accessible from the Debug bar during simulation.
- Toggle: interface style (Light/Dark), bold text, increase contrast, reduced motion, reduced transparency.
- Dynamic Type size slider — test all text sizes without changing device settings.

## APIs & Frameworks

### Swift Package Manager (Xcode integration)
- Package.swift — package manifest
- `Package.Dependency` — URL-based dependency with version rules
- `.upToNextMajor(from:)`, `.upToNextMinor(from:)`, `.exact(_:)`, `.branch(_:)`, `.revision(_:)` — version specification

### SwiftUI (Previews)
- `PreviewProvider` protocol — conform to provide previews
- `#Preview` macro (Xcode 15+); in Xcode 11: `static var previews: some View { ... }`
- `UIViewRepresentable` — wrap UIKit views for SwiftUI preview
- `UIViewControllerRepresentable` — wrap UIKit view controllers for SwiftUI preview

### Testing
- `XCTestPlan` — test plan file format (`.xctestplan`)
- `XCTestCase` — base class for unit and UI tests (unchanged)

### Asset Catalog / Symbols
- `UIImage(systemName:)` — load SF Symbol
- `UIImage.SymbolConfiguration` — configure weight, scale, point size
- Custom symbol assets in `.xcassets` — new asset type **[NEW]**
- Localized asset slots in Asset Catalogs **[NEW]**

## Code Highlights

Using `UIViewRepresentable` to preview a UIKit view in SwiftUI Canvas:
```swift
import SwiftUI

struct MyViewPreview: UIViewRepresentable {
    func makeUIView(context: Context) -> MyCustomUIView {
        return MyCustomUIView()
    }
    func updateUIView(_ uiView: MyCustomUIView, context: Context) {}
}

struct MyViewPreview_Previews: PreviewProvider {
    static var previews: some View {
        MyViewPreview()
            .previewLayout(.sizeThatFits)
    }
}
```

Adding a Swift Package dependency (Package.swift style):
```swift
// In Package.swift of your own package:
.package(url: "https://github.com/example/Forecast.git", from: "1.0.0")
```

## Takeaways
- The flexible editor workspace (unlimited splits, Focus mode, per-pane modes, Minimap) fundamentally changes the editing experience and should be explored before adopting old habits.
- Swift Package Manager integration in Xcode 11 makes package consumption first-class — discovering, adding, and using packages is as simple as using a framework dependency.
- SwiftUI's Canvas/Preview integration transforms UI development into a tight code+visual loop; `UIViewRepresentable` lets existing apps adopt previews today.
- Test plans enable true multi-configuration testing in a single action, making localization and accessibility testing practical to automate.

---
_Source: WWDC19 Session 401 page (abstract, chapter summaries, code samples, and resource links)._
