# What's new in Xcode
**WWDC22 · Session 110427** · [Watch](https://developer.apple.com/videos/play/wwdc2022/110427/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, tvOS 16, watchOS 9

## Overview
Xcode 14 is the fastest and most capable version of Xcode yet. The installer is 30% smaller. Builds are up to 25% faster thanks to eager Swift module emission and a 2x faster linker. Testing parallelism is improved by up to 30%, and notarization is 4x faster. Interface Builder document loading is up to 50% faster and device switching up to 30% faster.

Editor and preview productivity got significant upgrades: SwiftUI previews are interactive by default, a new Variants control enables side-by-side preview of multiple color schemes, text sizes, and orientations without code, and code completion now offers whole-initializer and Codable method completions. The Organizer window gained two new reports — Feedback (TestFlight feedback from beta users) and Hangs (App Store hang traces with weighted backtraces) — allowing developers to triage real-world issues without leaving Xcode.

Multiplatform app targets land as a first-class concept: a single target can target multiple platforms, eliminating the need to keep separate configurations in sync.

## Key Topics

### Size and Download Improvements
Xcode 14 is 30% smaller, installs faster, and additional platform simulators can be downloaded on demand (or at first use), rather than bundled upfront.

### SwiftUI Previews — Interactive by Default
The preview canvas is now interactive (live mode) by default. A new Variants button generates multiple preview configurations (color scheme, text size, orientation) side by side without any code changes.

### Code Completion Improvements
- Whole-initializer completion: Xcode offers to synthesize the complete init body from a struct's stored properties.
- Codable method completion: synthesizes `encode(to:)` and `init(from:)` implementations.
- Initializer calls appear directly in the completion list; optional parameters shown in italic (have default values) and can be opted into.
- More intelligent argument completion for modifiers with many optional parameters (e.g., `.frame`).

### Editor Improvements
- Redesigned Jump to Definition list: highlights distinguishing type/protocol info for each result.
- Callers list: Command-click to see all call sites for a method, with inline previews.
- Context breadcrumbs at the top of the editor show the enclosing definition even when scrolled off screen.
- Error dimming: stale diagnostics are shown in gray while the file is being re-evaluated; clears once the new build confirms them.

### Build Performance
- Eager Swift module emission: modules are produced sooner, unblocking downstream compilation and link tasks.
- Linker is up to 2x faster (parallelism improvements).
- Builds up to 25% faster overall (machines with most cores see the largest gains).
- New Build Timeline visualization in build logs and result bundles: shows per-task durations, critical path bottlenecks, and identifies long serial script phases.

### Testing Performance
- Test scheduling no longer creates dependencies between targets and test classes, increasing parallelism.
- Up to 30% improvement for projects with long-running tests across targets.

### Multiplatform Targets
A single app target can now list multiple supported platforms. Eliminates the need to keep settings and files synchronized across per-platform targets. Unique platform behaviors are described minimally.

### Organizer — Feedback Report
Shows all TestFlight feedback (comments, screenshots) from beta testers directly in Xcode. Inspector shows tester info, device configuration. One-click button to email the tester.

### Organizer — Hangs Report
New report showing highest-impact hangs from App Store users, with weighted backtraces identifying problematic code. Filterable by OS version, device. "Open in Project" button jumps directly to the offending code.

### Swift Package Plugins
Packages can now include plugins that run in-place (linters, formatters, invocable from the Project navigator) or as build tools (code generators, resource processors). Package resources support localization (default localization setting, export/import localization catalog).

### Icons — Single Size
App icon assets can specify a single image and let Xcode automatically generate all required sizes, replacing the need to pixel-hint separate images for every resolution.

### Memory Debugger
Expanded to show all reference paths in and out of an object (not just shortest paths to leaked objects), enabling total "weight" analysis of object graphs.

## APIs & Frameworks

**Xcode 14 — IDE Features**
- SwiftUI canvas interactive by default **[NEW]**
- Preview Variants button (color scheme, text size, orientation) **[NEW]**
- Whole-initializer code completion **[NEW]**
- Codable method code completion **[NEW]**
- Initializer completions in completion list with italic optional params **[NEW]**
- Redesigned Jump to Definition list **[NEW]**
- Callers list (Command-click) **[NEW]**
- Context breadcrumb bar (scrolled-out definitions) **[NEW]**
- Error dimming for stale diagnostics **[NEW]**
- Build Timeline visualization **[NEW]**
- Organizer Feedback Report **[NEW]**
- Organizer Hangs Report **[NEW]**
- Single Size app icon asset **[NEW]**
- On-demand simulator/platform download **[NEW]**
- Multiplatform app target (single target, multiple platforms) **[NEW]**
- Swift Package plugins support (run/build tool plugins) **[NEW]**
- Swift Package resource localization **[NEW]**
- Updated Run Destination chooser (recent devices, filter) **[NEW]**

**Swift 5.7 (shown in session)**
- Regular expression literals **[NEW]** — compiler-checked regex with syntax highlighting
- `\d` character class in regex literals

## Code Highlights

No code samples were provided in the session page. Key demonstration was in the live editor showing:
- Accepting whole-init completion for a struct initializer
- Using the `.frame` modifier with selective argument completion
- Command-clicking to open the Jump to Definition list and Callers list
- Regular expression literal with compiler checking: `/[0-9]+/` or `/\d+/`
- Previews Variants button — no code required

## Takeaways
- Xcode 14 is 30% smaller, builds up to 25% faster (eager Swift module emission + faster linker), and notarizes 4x faster.
- SwiftUI previews are interactive by default with a new Variants control that generates multi-configuration previews (color scheme, text size, orientation) without writing code.
- The new Organizer Feedback and Hangs reports bring TestFlight feedback and App Store hang traces directly into Xcode for fast triage with one-click navigation to source.
- A single multiplatform app target now replaces multiple per-platform targets, and Swift Package plugins extend Xcode with linters, formatters, and code generators.

---
_Source: WWDC22 Session 110427 page (abstract, chapter summaries, code samples, and resource links)._
