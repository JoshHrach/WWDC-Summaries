# What's new in SwiftUI
**WWDC23 · Session 10148** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10148/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, tvOS 17, watchOS 10, visionOS 1

## Overview
SwiftUI in 2023 expands to an entirely new platform (visionOS), gains major data flow improvements through `@Observable` and SwiftData integration, delivers extraordinary new animation APIs (keyframe animators, phase animators, spring animations, symbol effects, Metal shaders), and adds a wealth of interaction enhancements including powerful new scroll view APIs, haptic feedback, focus and keyboard improvements, and refined controls.

The session is structured around four themes: SwiftUI in more places, simplified data flow, extraordinary animations, and enhanced interactions — demonstrating each with a running "dog watching app" example.

## Key Topics

**SwiftUI in More Places**
- visionOS: `WindowGroup` renders as 2D window with depth; `.volumetric` scene style for bounded 3D; `ImmersiveSpace` scene for full or mixed immersion; `Model3D`, `RealityView`
- watchOS 10: Redesigned full-screen UI; new `containerBackground(_:for:)` modifier; `topBarLeading`/`topBarTrailing` toolbar placements; vertical paging `TabView`; `DatePicker` on watchOS (new); selection in `List` (new on watchOS)
- Widgets: Interactive controls (`Toggle`, `Button` via App Intents); animated widgets via transitions/animations; new placements on Lock Screen (iPadOS 17), Standby Mode, macOS desktop
- Xcode Previews: New `#Preview` macro syntax; widget timeline previews; interactive Mac app previews in Xcode

**Simplified Data Flow**
- `@Observable` macro replaces `ObservableObject`/`@Published`/`ObservedObject` — automatic property tracking, cleaner view code, precise invalidation
- SwiftData integration: `@Model` macro for persistent models; `@Query` property wrapper for fetching; `modelContainer` modifier on `App`
- `DocumentGroup` with SwiftData initializer for document-based apps; automatic sharing and document renaming (iOS/iPadOS 17)
- `Inspector` modifier: contextual detail panel (trailing sidebar on Mac/iPadOS regular, sheet on compact)
- Dialog enhancements: `confirmationDialog` customizations — custom button label, severity, suppression toggle, `HelpLink`
- Table improvements: column ordering/visibility with `TableColumnCustomization`; `DisclosureTableRow` for hierarchical data; `SceneStorage` persistence for column preferences; section programmatic expansion with binding
- `backgroundProminence` environment value for custom controls in prominent backgrounds

**Extraordinary Animations**
- `KeyframeAnimator` — parallel animation tracks with `KeyframeTrack`, using `LinearKeyframe`, `CubicKeyframe`, `SpringKeyframe`
- `PhaseAnimator` — sequential phase-based animations, each phase animated separately
- Spring animations: new `spring(duration:bounce:)` initializer; `.snappy`, `.bouncy` preset springs; springs are now the default animation for iOS 17+
- `sensoryFeedback(_:trigger:)` modifier — haptic/audio feedback on all supporting platforms **[NEW]**
- `visualEffect(_:)` modifier — apply effects based on geometry without GeometryReader
- `scrollTransition(_:)` modifier — apply effects as items enter/leave scroll view
- `ShaderLibrary` and `Shader` types — integrate Metal shader functions as SwiftUI `ShapeStyle` or effects **[NEW]**
- `.symbolEffect(_:)` modifier — apply SF Symbol animations (`bounce`, `pulse`, `scale`, `appear`, `disappear`, `replace`, `variableColor`)
- `textScale(_:)` modifier — semantic text scaling (alternative to small caps)
- `typesettingLanguage(_:)` modifier — reserve space for taller scripts in mixed-language text

**Enhanced Interactions**
- `ScrollView` improvements: `scrollTransition`, `scrollTargetLayout`, `scrollTargetBehavior`, `scrollPosition(id:)`, `containerRelativeFrame(count:span:)`; `.paging` behavior; custom `ScrollTargetBehavior` protocol
- `allowedDynamicRange(_:)` on `Image` for HDR rendering **[NEW]**
- `accessibilityZoomAction(_:)` modifier **[NEW]**
- `Color` static member lookup from asset catalog (compile-time safety) **[NEW]**
- `ControlGroup` with `.compactMenu` style **[NEW]**
- `Picker` with `.palette` style and `paletteSelectionEffect(_:)` **[NEW]**
- Button `borderShape` — `.circle`, `.roundedRectangle` built-in shapes **[NEW]**
- `springLoadingBehavior(_:)` modifier — drag-to-trigger on buttons **[NEW]**
- `.highlightHover` button highlight effect (tvOS) **[NEW]**
- `onKeyPress(_:action:)` modifier — hardware keyboard input handling **[NEW]**
- `ContentUnavailableView` — standard empty state view **[NEW]**

## APIs & Frameworks

**SwiftUI**
- `@Observable` macro **[NEW]** (Swift/Observation framework)
- `Observable` protocol **[NEW]**
- `@Model` macro (SwiftData) **[NEW]**
- `@Query` property wrapper **[NEW]** (SwiftData)
- `modelContainer(_:)` modifier **[NEW]** (SwiftData)
- `Inspector` modifier / `inspector(isPresented:content:)` **[NEW]**
- `ImmersiveSpace` scene type **[NEW]** (visionOS)
- `.immersionStyle(.mixed)`, `.immersionStyle(.full)` **[NEW]** (visionOS)
- `Model3D` **[NEW]** (visionOS)
- `RealityView` **[NEW]** (visionOS/RealityKit)
- `WindowGroup` `.volumetric` style **[NEW]** (visionOS)
- `containerBackground(_:for:)` **[NEW]**
- `KeyframeAnimator` **[NEW]**
- `KeyframeTrack` **[NEW]**
- `LinearKeyframe`, `CubicKeyframe`, `SpringKeyframe` **[NEW]**
- `PhaseAnimator` **[NEW]**
- `.spring(duration:bounce:)` animation **[NEW]**
- `.snappy`, `.bouncy` spring presets **[NEW]**
- `sensoryFeedback(_:trigger:)` **[NEW]**
- `SensoryFeedback` type **[NEW]**
- `visualEffect(_:)` **[NEW]**
- `scrollTransition(_:)` **[NEW]**
- `scrollTargetLayout()` **[NEW]**
- `scrollTargetBehavior(_:)` **[NEW]**
- `ScrollTargetBehavior` protocol **[NEW]**
- `scrollPosition(id:)` **[NEW]**
- `containerRelativeFrame(_:count:span:spacing:)` **[NEW]**
- `ShaderLibrary` **[NEW]**
- `Shader` **[NEW]**
- `.symbolEffect(_:)` **[NEW]**
- `textScale(_:)` **[NEW]**
- `typesettingLanguage(_:)` **[NEW]**
- `allowedDynamicRange(_:)` **[NEW]**
- `accessibilityZoomAction(_:)` **[NEW]**
- `paletteSelectionEffect(_:)` **[NEW]**
- `springLoadingBehavior(_:)` **[NEW]**
- `onKeyPress(_:action:)` **[NEW]**
- `ContentUnavailableView` **[NEW]**
- `DisclosureTableRow` **[NEW]**
- `TableColumnCustomization` **[NEW]**
- `backgroundProminence` environment value **[NEW]**
- `#Preview` macro **[NEW]**
- `DatePicker` on watchOS **[NEW]**
- `DocumentGroup` SwiftData initializer **[NEW]**
- `HelpLink` **[NEW]**

## Code Highlights

`@Observable` simplifies model observation:
```swift
@Observable class Dog {
    var name: String
    var isFavorite: Bool
}
// In view — no property wrapper needed:
var body: some View {
    Text(dog.name) // auto-tracks isFavorite reads
}
```

Keyframe animation:
```swift
KeyframeAnimator(initialValue: AnimationValues(), trigger: phase) { values in
    LogoView().offset(y: values.verticalTranslation)
} keyframes: { _ in
    KeyframeTrack(\.verticalTranslation) {
        SpringKeyframe(-30, duration: 0.25)
        CubicKeyframe(100, duration: 0.3)
        SpringKeyframe(0)
    }
}
```

Scroll snap with containerRelativeFrame:
```swift
ScrollView(.horizontal) {
    LazyHStack {
        ForEach(parks) { park in
            ParkCard(park: park)
                .containerRelativeFrame(.horizontal, count: 3, span: 1, spacing: 8)
        }
    }
    .scrollTargetLayout()
}
.scrollTargetBehavior(.viewAligned)
```

## Takeaways
- Migrate to `@Observable` today — it replaces `ObservableObject`, eliminates `@Published` annotations, and improves SwiftUI view performance with precise invalidation.
- Use `KeyframeAnimator` for complex multi-property parallel animations and `PhaseAnimator` for sequential staged effects.
- The new scroll APIs (`scrollTargetLayout`, `containerRelativeFrame`, `scrollPosition`) make snapping and position-aware layouts straightforward without custom solutions.
- `sensoryFeedback` provides a single cross-platform API for haptic and audio feedback.

---
_Source: WWDC23 Session 10148 page (abstract, chapter summaries, code samples, and resource links)._
