# Customize Feature Discovery with TipKit
**WWDC24 · Session 10070** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10070/)

_Platforms:_ iOS 18, iPadOS 18, macOS 15, tvOS 18, watchOS 11, visionOS 2

## Overview
TipKit, introduced in iOS 17, receives a set of targeted enhancements in iOS 18 that make tips more flexible and contextually accurate. This session focuses on three areas: reusable tip-display logic through `TipView` customization, new `Tip` configuration options for controlling cadence and eligibility, and expanded testing and debugging support through `Tips.resetDatastore()` and the TipKit preview API.

The session uses a note-taking app as a running example, progressively enriching its tip appearances with the new APIs. Each chapter is a self-contained recipe developers can apply independently.

## Key Topics
- **Custom `TipView` styling** — `TipView` now accepts a `style` parameter with a `DefaultTipViewStyle` or a custom `TipViewStyle` conformance, letting developers redesign the tip chrome without rebuilding the tip content.
- **`Tip.MaxDisplayCount`** — configure how many times a tip may appear before it is permanently dismissed; combine with `Tip.IgnoresDisplayFrequency` for one-shot onboarding tips.
- **Grouped tips** — `TipGroup` aggregates tips with shared display rules; the group's `currentTip` property returns the first tip whose conditions are satisfied, enabling a sequential tip flow.
- **`Tips.configure` options** — `Tips.ConfigurationOption.displayFrequency(_:)` and `Tips.ConfigurationOption.datastoreLocation(_:)` for per-app cadence tuning and custom persistence paths.
- **Testing** — `Tips.resetDatastore()` wipes all tip state for clean test runs; combined with Xcode previews and `#Preview` macros for live tip iteration.

## APIs & Frameworks

**TipKit**
- `Tip` protocol — unchanged core conformance; define `title`, `message`, `image`, `rules`, `options`
- `TipView` — primary SwiftUI display component
  - **[NEW]** `TipView(tip:arrowEdge:action:)` — updated initializer signature; `action` closure handles action button taps
  - **[NEW]** `TipViewStyle` protocol — conform to customize the visual rendering of a `TipView`
  - **[NEW]** `DefaultTipViewStyle` — system-provided style; use as baseline for custom styles
  - `.tipViewStyle(_:)` — apply a `TipViewStyle` to a `TipView` or a view hierarchy
- `popoverTip(_:arrowEdge:action:)` — unchanged modifier for popover-anchored tips
- `Tip.Options` — array of option values set on a `Tip` conformance's `options` property
  - `Tips.MaxDisplayCount(_:)` — **[NEW]** maximum times the tip shows; after that it is auto-dismissed
  - `Tips.IgnoresDisplayFrequency(_:)` — override global display frequency for this tip
- **[NEW]** `TipGroup` — aggregate multiple `Tip` types; declare as `@State var group = TipGroup([TipA.self, TipB.self])`
  - `TipGroup.currentTip` — the first tip in the group whose eligibility rules are met
- `Tips.configure(_:)` — configure global TipKit settings
  - `Tips.ConfigurationOption.displayFrequency(_:)` — `.immediate`, `.daily`, `.weekly`, `.monthly`, `.hourly`
  - `Tips.ConfigurationOption.datastoreLocation(_:)` — custom `URL` for tip persistence
- **[NEW]** `Tips.resetDatastore()` — wipe all stored tip state; ideal for testing
- **[NEW]** `Tips.showAllTipsForTesting()` — force all tips eligible regardless of rules; useful in Xcode previews
- `Tips.hideAllTipsForTesting()` — suppress all tips; existing API now documented for testing flows
- `Tip.Rule` / `#Rule` macro — unchanged eligibility rule DSL
- `Event` / `Parameter` — unchanged tracking primitives

## Code Highlights
Limit a tip to a single appearance:

```swift
struct NewFeatureTip: Tip {
    var title: Text { Text("New Feature") }
    var options: [TipOption] {
        Tips.MaxDisplayCount(1)
    }
}
```

Sequential tip group in a view:

```swift
@State var onboardingGroup = TipGroup([WelcomeTip.self, ExportTip.self, ShareTip.self])

var body: some View {
    ContentView()
        .task {
            if let tip = onboardingGroup.currentTip {
                TipView(tip)
            }
        }
}
```

Reset tip state before each UI test:

```swift
override func setUp() async throws {
    try await Tips.resetDatastore()
    try await Tips.configure()
}
```

## Takeaways
- Use `TipGroup` to build linear onboarding flows — the group automatically advances to the next tip once the current one is dismissed, without any coordinator logic.
- Set `Tips.MaxDisplayCount(1)` on critical onboarding tips so they appear exactly once and never return, preventing repetition fatigue.
- Call `Tips.showAllTipsForTesting()` in Xcode previews to see every tip in your UI without satisfying eligibility rules.
- Store the TipKit datastore in an app group container (`datastoreLocation`) when you need tips shared across app extensions (e.g., widgets and main app).

---
_Source: WWDC24 Session 10070 page (abstract, chapter summaries, code samples, and resource links)._
