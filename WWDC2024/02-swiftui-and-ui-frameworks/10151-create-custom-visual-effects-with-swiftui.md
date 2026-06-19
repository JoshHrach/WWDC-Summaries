# Create Custom Visual Effects with SwiftUI
**WWDC24 · Session 10151** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10151/)

_Platforms:_ iOS 18, iPadOS 18, macOS 15, tvOS 18, watchOS 11, visionOS 2

## Overview
This session is a practical tour of SwiftUI's visual effects toolkit as expanded in iOS 18. Two Apple engineers build out five categories of effect — custom scroll transitions, mesh gradients, custom view transitions, text renderer animations, and Metal shader effects — demonstrating how layering these tools produces polished, differentiated app experiences.

The standout new APIs are `MeshGradient` (a grid-based gradient interpolation view), `TextRenderer` (a protocol giving full control over how `Text` draws itself into a `GraphicsContext`), and expanded `ShaderLibrary` / `layerEffect` support for writing GPU shaders in Metal that plug directly into SwiftUI's render tree.

## Key Topics
- **Scroll effects** — `scrollTransition` and `visualEffect` modifiers let views change rotation, offset, hue, scale, or blur based on scroll position; the proxy provides geometry values performantly.
- **Mesh Gradients** — `MeshGradient` composes a grid of SIMD2 control points with per-point colors; moving control points produces organic, animated color fields.
- **Custom view transitions** — the `Transition` protocol's `body(content:phase:)` method now accepts richer `TransitionPhase` checks (`.willAppear`, `.didDisappear`, `.isIdentity`) enabling directional animations.
- **Text renderer** — `TextRenderer` protocol with `draw(layout:in:)` gives access to `Text.Layout`, its lines, runs, and individual run slices (glyphs) for per-glyph animation; combined with `TextAttribute` for marking specific words.
- **Metal shaders** — `ShaderLibrary` instantiates `.metal` functions as SwiftUI `Shader` values; `layerEffect` applies them per-pixel on the GPU; `keyframeAnimator` drives time-based shader parameters.

## APIs & Frameworks

**SwiftUI**
- `scrollTransition(axis:transition:)` — apply transforms based on a view's position in a scroll view; `phase.value` and `phase.isIdentity` drive the effect
- `visualEffect(_:)` — geometry-aware effect closure on any view; `proxy.frame(in:)` provides position
- **[NEW]** `MeshGradient(width:height:points:colors:)` — grid-based gradient; points are `[SIMD2<Float>]`, colors are `[Color]`
- `Transition` protocol — `body(content:phase:)` with `TransitionPhase` (`.willAppear`, `.isIdentity`, `.didDisappear`)
- `AnyTransition.combined(with:)` — compose multiple transitions
- **[NEW]** `TextRenderer` protocol — `draw(layout:in:)` receives `Text.Layout` and `GraphicsContext`
  - `Text.Layout` — `Collection` of `Line`; `Line` is `Collection` of `Run`; `Run` is `Collection` of `RunSlice`
  - `Text.Layout.Line.typographicBounds` — bounds including ascender/descender for positioning
  - `RunSlice.typographicBounds.descent` — descender length; useful for spring start position
- **[NEW]** `TextAttribute` protocol — custom attribute attachable to `Text` via `.customAttribute(_:)`; readable in `draw(layout:in:)` via `run[MyAttribute.self]`
- `.customAttribute(_:)` — new `Text` modifier for attaching `TextAttribute` instances
- **[NEW]** `.textRenderer(_:)` — view modifier applying a `TextRenderer` to all `Text` in the subtree
- `GraphicsContext.draw(_:options:)` — draw a `Text.Layout.RunSlice`; `.disablesSubpixelQuantization` option prevents jitter during spring animations
- `ShaderLibrary` — access compiled `.metal` functions as `Shader` values via dynamic member lookup (`ShaderLibrary.myFunction(...)`)
- `Shader` — wraps a Metal function plus parameter values (`.float2`, `.float`, `.color`, `.image`)
- `.layerEffect(_:maxSampleOffset:isEnabled:)` — apply a `Shader` as a GPU layer effect per pixel
- `keyframeAnimator(initialValue:trigger:content:keyframes:)` — drive shader `elapsedTime` via `LinearKeyframe` / `SpringKeyframe` / `MoveKeyframe`

## Code Highlights
Mesh gradient with movable control points:

```swift
MeshGradient(
    width: 3, height: 3,
    points: [[0,0],[0.5,0],[1,0], [0,0.5],[0.9,0.3],[1,0.5], [0,1],[0.5,1],[1,1]],
    colors: [.black,.black,.black, .blue,.blue,.blue, .green,.green,.green]
)
```

Minimal `TextRenderer`:

```swift
struct AppearanceEffectRenderer: TextRenderer, Animatable {
    var elapsedTime: TimeInterval
    var animatableData: Double { get { elapsedTime } set { elapsedTime = newValue } }
    func draw(layout: Text.Layout, in context: inout GraphicsContext) {
        for line in layout { context.draw(line) }
    }
}
```

Ripple Metal shader applied via `layerEffect`:

```swift
let shader = ShaderLibrary.Ripple(.float2(origin), .float(elapsedTime),
                                   .float(amplitude), .float(frequency),
                                   .float(decay), .float(speed))
content.layerEffect(shader, maxSampleOffset: CGSize(width: amplitude, height: amplitude))
```

## Takeaways
- `TextRenderer` + `TextAttribute` is the path to per-glyph, per-run, or per-line custom text animation without reaching for `Canvas` or `UIKit`.
- Use `GraphicsContext.draw(_:options: .disablesSubpixelQuantization)` when animating text with spring physics to eliminate pixel-grid jitter as springs settle.
- Build a debug UI (sliders for shader parameters) before finalizing shader constants — the `RippleModifier` pattern in this session shows the technique.
- `MeshGradient` control points range 0–1 on each axis; animate them with `withAnimation` or `keyframeAnimator` to produce smooth, organic background effects.

---
_Source: WWDC24 Session 10151 page (abstract, chapter summaries, code samples, and resource links)._
