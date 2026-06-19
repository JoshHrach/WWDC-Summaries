# 18 things from WWDC24
**WWDC24 · Session 111976** · [Watch](https://developer.apple.com/videos/play/wwdc2024/111976/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia, watchOS 11, tvOS 18, visionOS 2

## Overview
WWDC24 brought a sweeping set of announcements across every Apple platform. This curated highlights session distills 18 of the most developer-relevant features and changes from the week, spanning Apple Intelligence, Swift 6, Xcode 16, new system frameworks, and major updates to existing ones.

Apple Intelligence is the headline: on-device generative AI for text, images, and Siri, integrated through new APIs like Writing Tools, Image Playground, and Genmoji. Developers can surface their app's functionality into Apple Intelligence's action layer through App Intents and the new Assistant Schemas.

Beyond Apple Intelligence, the session touches on Swift 6's complete data-race safety, SwiftUI improvements, the new Translate API, RealityKit additions, and Game Porting Toolkit 2—making it an essential orientation for developers evaluating where to invest in the new SDK.

## Key Topics
- **Apple Intelligence** — On-device generative AI across writing, image creation, and Siri; requires A17 Pro / M-series
- **App Intents & Assistant Schemas** — Expose app actions to Siri and Apple Intelligence with structured schemas
- **Genmoji** — `NSAdaptiveImageGlyph` embeds custom AI-generated emoji in text; requires `NSAttributedString` and `UITextView`/`NSTextView` adoption
- **Writing Tools** — System-level text rewriting/summarization; standard text views get it automatically
- **Image Playground** — New framework to let users create AI-generated images within your app
- **Swift 6** — Complete concurrency checking by default; data-race safety verified at compile time
- **Xcode 16** — Predictive code completion, new debug features (RealityKit Debugger, MPSGraph Viewer), Swift Testing integration
- **SwiftUI** — Custom containers, new animation APIs, enhanced scroll view, `.searchable` improvements
- **RealityKit** — `LowLevelMesh`, `LowLevelTexture`, `SpatialTrackingSession`, hover effects
- **Translate API** — New `Translation` framework for in-app language translation
- **Game Porting Toolkit 2** — Improved Metal shader conversion, AVX2 support, better performance for Windows game ports
- **TabletopKit** — New framework for building multiplayer tabletop games on visionOS
- **HealthKit** — New health data types, mental health domains
- **Passkeys** — Expanded to group accounts, enterprise SSO improvements
- **Safari & Web** — New WebXR capabilities on visionOS, Distraction Control, Highlights

## APIs & Frameworks
### Apple Intelligence
- **[NEW] Writing Tools** — automatic in `UITextView`/`NSTextView`; opt-out via `writingToolsBehavior`
- **[NEW] Image Playground** — `ImagePlaygroundViewController` to launch image creation UI
- **[NEW] NSAdaptiveImageGlyph** — embed Genmoji in attributed strings; `contentDescription` property required for accessibility

### App Intents
- **[NEW] Assistant Schemas** — `@AssistantIntent(schema: .photos.search)` etc.; maps intents to Siri/Apple Intelligence
- **[NEW] IndexedEntity** — index App Entities for on-device semantic search

### Swift & Xcode
- **Swift 6 language mode** — opt-in via `swiftLanguageVersion: .v6` in Package.swift or build settings
- **[NEW] Swift Testing framework** — `@Test`, `@Suite`, `#expect()` macros; available in Xcode 16
- **Xcode 16 Predictive Completion** — model-powered code completion suggestions

### RealityKit / visionOS
- **[NEW] LowLevelMesh** — direct GPU buffer access for custom mesh layouts
- **[NEW] LowLevelTexture** — GPU-backed texture updates
- **[NEW] SpatialTrackingSession** — replaces raw ARKit session for hand/world tracking in RealityKit apps
- **[NEW] HoverEffectComponent** — `.highlight` and `.shader` hover styles (visionOS 2)

### System Frameworks
- **[NEW] Translation framework** — `TranslationSession` for in-app translation
- **[NEW] TabletopKit** — multiplayer tabletop game framework (visionOS)
- **HealthKit** — new `HKQuantityType` for mental health state, anxiety/depression assessments

## Code Highlights
```swift
// Swift 6 package manifest
// swift-tools-version: 6.0
let package = Package(name: "MyApp", ...)

// App Intents with Assistant Schema
@AssistantIntent(schema: .system.search)
struct SearchIntent: AppIntent {
    @Parameter(title: "Query") var query: String
    func perform() async throws -> some ReturnsValue<[Entity]> { ... }
}

// Genmoji support in text view
textView.supportsAdaptiveImageGlyph = true
```

## Takeaways
- Apple Intelligence is the platform story for 2024; App Intents + Assistant Schemas are the primary integration path for third-party developers
- Swift 6's data-race safety is opt-in per module for 2024 but is the future-forward default; migration tooling is available in Xcode 16
- RealityKit's `LowLevelMesh`/`LowLevelTexture` unlock custom GPU pipelines for high-performance spatial content
- Game Porting Toolkit 2 dramatically reduces the effort to bring Windows/DirectX games to macOS and Apple Silicon

---
_Source: WWDC24 Session 111976 page (abstract, chapter summaries, code samples, and resource links)._
