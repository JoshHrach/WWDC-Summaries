# What's new in Xcode 26
**WWDC25 · Session 247** · [Watch](https://developer.apple.com/videos/play/wwdc2025/247/)

_Platforms:_ macOS Tahoe 26

## Overview
Xcode 26 brings improvements across six areas: download size/performance optimizations (24% smaller, 40% faster workspace load), workspace and editing quality-of-life (tabs, multi-word search, Voice Control Swift mode, the new `#Playground` macro, Icon Composer, String Catalog enhancements), integrated LLM coding assistance (ChatGPT, Claude, local models), debugging and performance (Swift concurrency debugger improvements, Processor Trace, CPU Counters presets, next-gen SwiftUI instrument, Power Profiler, Organizer Trending Insights and Recommendations), build system changes (Explicit Modules for Swift by default, Swift Build open source integration, Enhanced Security capability), and significantly improved UI testing (new code generation, Automation Explorer element inspection, `XCTHitchMetric`, Thread Performance Checker).

## Key Topics

### Optimizations
Xcode 26 is **24% smaller** than Xcode 15. Simulator runtimes drop Intel support by default; the Metal toolchain downloads only when your project needs it. Text input latency improved up to 50% in complex expressions. Workspace loads **40% faster**.

### Workspace and editing

**Editor tabs**: Safari-style new tab start page; pin a tab to lock it to a specific file.

**Multiple Words search**: New search mode finds clusters of terms in proximity across files, sorted by relevance, terms can appear in any order across lines.

**Voice Control Swift Mode**: Dictate Swift code naturally — Voice Control understands Swift syntax (camelCase, operators, spaces).

**`#Playground` macro**: Inline code execution canvas. Add `import Playgrounds`, write `#Playground { ... }`, and results appear in a canvas tab. Multiple playgrounds open separate canvas tabs. Results update live as code changes. Regex match results, structures, and custom types have Quick Look visualizations. The macro is being open-sourced for Swift developers on other platforms.

**Icon Composer**: New app bundled with Xcode 26. Creates multi-layered, multi-platform icons (iOS dark/tinted/light, macOS, watchOS) in a single file. Supports blur, shadow, specular highlights, translucency. Also generates flat icons for older OS/web.

**String Catalogs**: Type-safe Swift symbols for localized strings — symbols defined in the String Catalog become auto-complete suggestions. Automatic context comment generation using on-device model.

### Intelligence (LLM integration)
Xcode integrates LLMs for coding assistance:
- **Code assistant panel**: Ask general Swift questions or project-specific questions. Xcode sends relevant source context. Responses include links to mentioned files.
- **`@` symbol references**: Type `@` in a prompt to reference specific symbols, files, or issues; attach image files for UI-from-sketch generation.
- **Coding tools menu**: Lightweight inline action menu (Cmd-click or right-click in editor) with quick actions: generate playground, explain code, fix issues, fix deprecations.
- **Project context toggle**: Control whether Xcode includes project context in queries.
- **Auto-apply toggle**: Code changes can be applied automatically or reviewed manually.
- **Modification history**: Scrub through staged changes per conversation turn; revert individual change sets.
- **Providers**: ChatGPT built-in (limited free daily requests, or bring your own account). Anthropic (Claude), local models via Ollama/LM Studio, any OpenAI-compatible API. Multiple providers configurable in preferences; switch per conversation.

### Debugging and performance
**Swift concurrency debugger**: Xcode follows execution into async functions across thread hops. Task IDs shown in backtrace view and Program Counter annotation. Concurrency types (`Task`, `TaskGroup`, actors) show readable representations in Variables view.

**Usage description assistant**: When an app stops due to missing privacy usage description, Xcode explains the missing key and provides an Add button that navigates directly to Signing & Capabilities.

**Signing & Capabilities**: Many capabilities requiring usage descriptions now editable directly in Xcode (instead of editing Info.plist manually).

**Processor Trace** (Xcode 16.3+, M4/iPhone 16+): Captures every CPU branch decision on all threads with little overhead — non-sampling, full execution flow visualization.

**CPU Counters**: Redesigned with preset modes: CPU Bottlenecks (useful vs. bottlenecked work breakdown), Instruction Characteristics, Metrics. Includes documentation for each counter.

**SwiftUI instrument** (next-gen): Cause-and-effect graph showing which Observable changes triggered which view updates; per-view update counts.

**Power Profiler**: New instrument for battery usage profiling. Tethered + passive (initiated from Developer Settings without connection) recording modes. System-level power correlated with thermal/charging state. Per-component process impact (CPU, GPU, Display, Networking).

**Organizer Trending Insights**: Expanded from Disk Write to **Hang** and **Launch** diagnostics. Flame icon flags worsening trends across the last 5 app versions. Chart shows trend over time for context.

**Metric Recommendations**: Compare app metrics (starting with Launch Time) against similar apps and historical data. Shows target value to aim for.

**URL sharing**: Share specific diagnostic reports with colleagues via URL.

### Builds
**Explicit Modules for Swift**: Enabled by default (was C/ObjC only in Xcode 16). Three-phase build: scan → build modules → build source. More precise module sharing, better build reliability, faster Swift debugger startup.

**Swift Build open source**: Integrated into Swift Package Manager. Cross-platform support (Linux, Windows, Android). Preview with `swift build --build-system <impl>`. Community can contribute on GitHub.

**Enhanced Security capability**: New capability in Signing & Capabilities — provides pointer authentication and other Apple-internal app protections. Recommended for social media, messaging, image viewing, browsing apps.

### Testing
**UI automation recording** (redesigned): New code generation system — interact with the app in Simulator, Xcode generates concise test code with multiple identifier options per element. Recording triggered by clicking the gutter button with cursor in a test method body.

**Automation Explorer**: Test failure video playback + element inspection. Generated code snippet for any inspected element. Correct identifiers even after app UI changes.

**Hardware interactions**: Expanded support for hardware keyboard and hardware button presses in UI tests.

**`XCTHitchMetric`** **[NEW]**: Measure scrolling animation hitch performance in UI tests. Reports Hitch Time Ratio.

**Runtime API Checks in Test Plans**: Framework runtime issues and Thread Performance Checker (priority inversions, non-UI work on main thread) surfaced as test issues. Option to fail tests on any runtime API check issue.

## APIs & Frameworks

### Xcode features (no public API)
- `#Playground { }` macro **[NEW]** — inline canvas code execution (requires `import Playgrounds`)
- `import Playgrounds` **[NEW]** module
- Icon Composer app **[NEW]** — bundled with Xcode 26
- String Catalog typed Swift symbols **[NEW]**
- Automatic context comment generation for String Catalogs **[NEW]** (on-device model)
- Code assistant panel with `@` symbol/file/issue references **[NEW]**
- Coding tools inline action menu **[NEW]**
- Modification history in code assistant **[NEW]**
- Multi-provider LLM support: ChatGPT, Anthropic, Ollama, LM Studio **[NEW]**
- Multiple Words search mode **[NEW]**
- Swift Mode for Voice Control **[NEW]**
- Editor tab pin + start page **[NEW]**

### Instruments
- **Processor Trace** instrument (Xcode 16.3+, M4/iPhone 16+) — full execution flow, no sampling
- **CPU Counters** — redesigned with preset modes (CPU Bottlenecks, Instruction Characteristics, Metrics) **[NEW modes]**
- **SwiftUI instrument** (next-gen) — cause-and-effect graph **[NEW]**
- **Power Profiler** instrument **[NEW]** — tethered + passive recording, per-component impact
- SwiftUI `LevelOfDetail` environment value instrument support

### Xcode Organizer
- Trending Insights for Hang and Launch **[NEW]** (was Disk Write only)
- Metric Recommendations for Launch Time (+ more coming) **[NEW]**
- URL sharing for diagnostic reports **[NEW]**

### Build system
- Explicit Modules for Swift — enabled by default **[NEW default]**
- Swift Build integration into Swift Package Manager **[NEW]**
- Enhanced Security capability **[NEW]**
- `swift build --build-system <impl>` **[NEW]** — preview Swift Build in SPM

### XCTest
- **`XCTHitchMetric(application:)`** **[NEW]** — measures hitch time ratio
- Thread Performance Checker in Test Plans **[NEW]**
- Runtime API Checks in Test Plans — `.failOnRuntimeAPIChecks` **[NEW]**

## Code Highlights

```swift
// Inline playground
import Playgrounds
#Playground {
    let landmark = Landmark.exampleData.first
    let region = landmark?.coordinateRegion
}

// XCTHitchMetric for scrolling
func testScrollingAnimationPerformance() throws {
    let options = XCTMeasureOptions()
    options.invocationOptions = .manuallyStop
    let app = XCUIApplication()
    app.launch()
    let scrollView = app.scrollViews.firstMatch
    measure(metrics: [XCTHitchMetric(application: app)], options: options) {
        scrollView.swipeUp(velocity: .fast)
        stopMeasuring()
        scrollView.swipeDown(velocity: .fast)
    }
}
```

## Takeaways
- Enable the code assistant with ChatGPT (free daily quota) or your own API key immediately — use `@` references for targeted multi-file changes and Coding Tools for inline playground generation and fix-it actions.
- Upgrade to Xcode 26 and rebuild to get Explicit Modules for Swift by default — this speeds up Swift debugging significantly and improves incremental build reliability.
- Add Power Profiler traces to your development workflow alongside Memory Graph and Instruments CPU profiling — passive recording mode lets you capture battery issues in real-world usage without a tether.
- Use `XCTHitchMetric` in existing scroll performance tests to get quantitative hitch budgets; enable Thread Performance Checker in test plans to catch priority inversions early.

---
_Source: WWDC25 Session 247 page (abstract, chapter summaries, code samples, and resource links)._
