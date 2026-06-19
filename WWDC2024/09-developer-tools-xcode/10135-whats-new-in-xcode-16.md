# What's New in Xcode 16
**WWDC24 · Session 10135** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10135/)

_Platforms:_ macOS Sequoia (required for predictive code completion on Apple silicon)

## Overview
Xcode 16 delivers improvements at every stage of the development cycle — editing, building, debugging, testing, and profiling. Predictive code completion is powered by a new on-device ML model trained specifically on Swift and Apple SDKs, available exclusively on Apple silicon Macs running macOS Sequoia. Previews are faster than ever thanks to a new execution engine that reuses build products. Explicit modules dramatically improve build parallelism and diagnostic clarity. The debugger gains a new Unified Backtrace View, an extended Thread Performance Checker, and a new RealityKit debugger for spatial apps. Swift Testing integrates natively into the Test Navigator, and Instruments gains a Flame Graph for rapid performance analysis.

## Key Topics

### Code Completion
Xcode 16's code completion uses an on-device model trained on Swift and Apple SDKs. The model uses surrounding context — function names, comments, and nearby symbols — to produce accurate multi-token suggestions. No network connection is required. Requires Apple silicon running macOS Sequoia.

### Adopting Swift 6 Data-Race Safety
Swift 6 turns data races from runtime problems into compile-time errors. To adopt incrementally, navigate to "Swift Compiler – Upcoming Features" in Build Settings and enable features one at a time (e.g., "Isolated Global Variables"). Each enabled feature surfaces warnings that must be resolved before they become Swift 6 errors. When ready, set the Swift Language Version to 6. See "Migrate your app to Swift 6" for guidance.

### Improvements to Previews
Two new APIs make previews simpler and more powerful:
- `@Previewable` macro: attach to property wrappers like `@State` inside a `#Preview` block — no wrapper view needed.
- `PreviewModifier`: a protocol for sharing expensive setup (e.g., model containers, async-loaded data) across multiple previews, with system-level caching so `makeSharedContext()` is only called once per modifier type.

Beyond the API, previews in Xcode 16 use a new execution engine that reuses the same build products as the project itself, reassembling the program on the fly rather than producing a separate copy — making them faster across the board.

### Explicit Modules
Explicit modules (enabled by default for C/Objective-C, opt-in for Swift via "Explicitly Built Modules" in Build Settings) split compilation into three visible phases: scanning, building modules, and building source files. This improves parallelism, surfaces clearer error messages when module builds fail, and makes the build timeline more informative. LLDB also benefits: it reuses module build outputs when evaluating expressions, speeding up debugging. See "Demystify explicitly built modules" for depth. Additionally, builds can now begin before Swift package resolution completes.

### Debugging
- **DWARF5** is now the default debug symbol format for macOS Sequoia / iOS 18 deployment targets — dSYM bundles are smaller and symbol lookups are faster.
- **Thread Performance Checker** now detects excessive disk writes and slow app launch paths in addition to main-thread hangs and priority inversions.
- **Organizer** gains a new "App Launch" diagnostic category showing slowest code-path signatures across customer devices. Disk writes signatures now show trend arrows across app versions.
- **Unified Backtrace View**: enabled from the Debug Bar, it shows the full call stack as an interactive scrollable view with inline source at each frame; hover over variables to inspect values inline.
- **RealityKit Debugger**: capture a snapshot of a running app's entity hierarchy and explore it in 3D inside Xcode, inspecting built-in and custom components. See "Break into the RealityKit debugger."

### Swift Testing
Swift Testing is a new framework that uses Swift language features for concise, expressive tests — running alongside existing XCTests. Key features demonstrated:
- `@Test` macro marks any function as a test; it appears in the Test Navigator immediately.
- `#expect(_:)` validates boolean expressions with rich failure diagnostics (including "Show Details" to expand both sides of a comparison).
- Parameterized tests: supply arguments to `@Test` and a single function handles all cases, each running in parallel and displayed individually in the navigator.
- Custom tags (`extension Tag { @Tag static var planting: Self }`) group tests across suites for filtering and test plan inclusion/exclusion.
- Test plans support including and excluding tags, letting teams gate unstable feature tests from CI without deleting them.

### Instruments: Flame Graph
Instruments 16 adds a Flame Graph view accessible from the jump bar in any call-tree instrument (e.g., Time Profiler). Execution intervals are weighted by time percentage and sorted left-to-right (heaviest on the left). Right-clicking a frame offers "Reveal in Xcode" for direct navigation to the offending code. Flame Graph works for every Instrument that uses call trees.

## APIs & Frameworks

**Xcode Previews**
- `@Previewable` macro **[NEW]** — inline `@State` / `@Query` in `#Preview` block, no wrapper view needed
- `PreviewModifier` protocol **[NEW]** — shared, cached async setup for preview environments
  - `makeSharedContext() async throws -> Context` **[NEW]** — called once; result cached across modifiers of same type
  - `body(content:context:) -> some View` **[NEW]**
- `PreviewTrait` extension for `.modifier(_:)` **[NEW]** — shorthand trait for reuse at call sites
- `#Preview(traits:)` **[NEW]** — attach `PreviewTrait` values including custom modifiers

**Build System**
- Explicit modules for Swift **[NEW]** — "Explicitly Built Modules" Build Setting (`SWIFT_ENABLE_EXPLICIT_MODULES`)
- "Scan dependencies" / "Compile Clang module" / "Compile Swift module" build log phases **[NEW]**
- Build timeline updated to show module phases **[NEW]**
- Swift package build begins before resolution completes **[NEW]**
- DWARF5 default debug format (macOS Sequoia / iOS 18 targets) **[NEW]**

**Debugger / Organizer**
- Unified Backtrace View **[NEW]** — scrollable inline call stack in the Debug Bar
- Thread Performance Checker: disk write diagnostics **[NEW]**, app launch diagnostics **[NEW]**
- Organizer "App Launches" diagnostic category **[NEW]**
- Organizer disk-write signature trend arrows **[NEW]**
- RealityKit Debugger **[NEW]** — 3D entity hierarchy snapshot and inspection

**Swift Testing (new framework)**
- `import Testing` **[NEW]**
- `@Test` macro **[NEW]** — mark functions as tests; accepts display name, traits
- `#expect(_:)` **[NEW]** — boolean assertion with expandable diagnostic detail
- Parameterized tests via `@Test(arguments:)` **[NEW]** — each argument runs as a separate parallel case
- `extension Tag { @Tag static var name: Self }` **[NEW]** — custom test tags
- `.tags(_:)` trait on `@Test` **[NEW]** — group tests for navigator tag view and test plans
- Test plan tag inclusion/exclusion **[NEW]**
- Quick Actions shortcut (Command-Shift-A) for "test again" **[NEW]**

**Instruments**
- Flame Graph view **[NEW]** — available in all call-tree instruments; "Reveal in Xcode" context menu

## Code Highlights

`@Previewable` — inline state in preview:
```swift
#Preview {
    @Previewable @State var currentFace = RobotFace.heart
    RobotFaceSelectorView(currentFace: $currentFace)
}
```

`PreviewModifier` — shared, cached async environment:
```swift
struct SampleRobotNamer: PreviewModifier {
    typealias Context = RobotNamer

    static func makeSharedContext() async throws -> Context {
        let url = URL(fileURLWithPath: "/tmp/local_names.txt")
        return try await RobotNamer(url: url)
    }

    func body(content: Content, context: Context) -> some View {
        content.environment(context)
    }
}

extension PreviewTrait where T == Preview.ViewTraits {
    @MainActor static var sampleNamer: Self = .modifier(SampleRobotNamer())
}

#Preview(traits: .sampleNamer) {
    RobotNameSelectorView()
}
```

Swift Testing — basic test with `#expect`:
```swift
import Testing
@testable import BOTanist

@Test func plantingRoses() {
    let plant = Plant(type: .rose)
    let expected = Plant(type: .rose, style: .graft)
    #expect(plant == expected)
}
```

Swift Testing — parameterized test with custom tag:
```swift
extension Tag {
    @Tag static var planting: Self
}

@Test(.tags(.planting), arguments: [PlantAnimationState.grow, .water, .idle])
func canTransitionToCelebrate(state: PlantAnimationState) {
    #expect(state.canTransition(to: .celebrate))
}
```

Fixing main-thread I/O flagged by Thread Performance Checker:
```swift
private nonisolated func robotVideoAVPlayer() async throws -> AVPlayer? {
    guard let url = Bundle.main.url(forResource: RobotVideo.resource,
                                    withExtension: RobotVideo.ext) else {
        throw BOTanistAppError.videoNotFound(forResource: RobotVideo.resource,
                                              withExtension: RobotVideo.ext)
    }
    let avPlayer = BOTanistAVPlayer()
    return try avPlayer.player(url: url)
}
```

Parallelizing asset loading (Instruments Flame Graph finding):
```swift
// Before (serial, blocks main thread):
for asset in allAssets { asset.load() }

// After (parallel, background):
await withDiscardingTaskGroup { group in
    for asset in allAssets {
        group.addTask { asset.load() }
    }
}
```

## Takeaways
- Use `@Previewable` to eliminate wrapper views in previews and `PreviewModifier` to share expensive setup (SwiftData `ModelContainer`, async server data) across previews with system-level caching.
- Enable "Explicitly Built Modules" in Build Settings for better build parallelism, clearer error messages, and faster LLDB expression evaluation — no source changes required.
- Adopt Swift Testing's `@Test` and `#expect` for more expressive tests: rich failure detail, parameterized cases that run in parallel, and tag-based organization across suites and test plans.
- Profile with Instruments' Flame Graph first — its left-weighted, percentage-proportional display pinpoints the heaviest call paths in seconds, and "Reveal in Xcode" takes you directly to the problem.

---
_Source: WWDC24 Session 10135 page (abstract, chapter summaries, code samples, and resource links)._
